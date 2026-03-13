# Messier Marathon Observer's Log — Project Design Summary

## Overview

The Messier Marathon Observer's Log (MMOL) is a single-file browser application for logging observations of Charles Messier's 110 deep-sky objects. It requires no server, no framework, and no build step. The entire tool is delivered as one `index.html` with embedded CSS, JavaScript, and a static DATA array.

**Version:** Beta v2.5.4  
**Live site:** [mmol.thgnetworks.com](https://mmol.thgnetworks.com)  
**Repository:** [github.com/thg4web/mmol](https://github.com/thg4web/mmol)

---

## Architecture

### Single-file design

`index.html` contains everything needed to run the application:

- Two `<style>` blocks — one screen stylesheet, one `@media print` stylesheet
- The full `DATA` array (110 objects × 9 fields each)
- All application logic in a single `<script>` block
- All modal HTML as static markup toggled by CSS class

No external dependencies are loaded at runtime. jsPDF and jspdf-autotable are loaded from Cloudflare CDN only when the user initiates PDF generation.

### Data model

Each Messier object is a flat array in the `DATA` constant:

```
[marathonOrder, messierID, type, constellation, magnitude, size, ra, dec, psaPages]
```

Example: `[1, "M77", "Spiral Gal", "Cet", 8.9, "7×6", "02 42.7", "-00 01", "4, 6"]`

- `marathonOrder` (index 0) — Don Machholz marathon sequence position (1–110)
- `messierID` (index 1) — canonical identifier used as the key for all localStorage operations
- `size` (index 5) — angular dimensions string from the SEDS marathon data (e.g. `"178×63"` or `"110'"`)

The array is never mutated at runtime. Sort operations in Numeric mode work on a shallow copy.

### State management

All mutable state lives in four top-level objects that mirror localStorage keys:

| Variable | localStorage key | Contents |
|----------|-----------------|----------|
| `SAVED_CHECKS` | `mm_checks` | `{ "M77": true, … }` |
| `SAVED_NOTES` | `mm_notes` | `{ "M77": "text…", … }` |
| `SAVED_TIMES` | `mm_times` | `{ "M77": "2026-03-28T22:14:00.000Z", … }` |
| `SAVED_FULLNOTES` | `mm_fullnotes` | `{ "M77": { date, time, site, … }, … }` |

`SESSION` mirrors `mm_session`. A single `saveState()` function serialises all four objects to localStorage on every mutation.

### Rendering

`renderTable()` builds the full table body as a template-literal HTML string and sets `tbody.innerHTML` in one operation. It is called on initial load, on sort toggle, on filter/search change, and after any data import.

`applyFilter()` works on the rendered DOM rows via `tr.dataset` attributes, toggling `display` without re-rendering. This keeps filter and search operations fast without touching the DATA array.

`openLightbox(idx)` uses `idx` as a direct index into `DATA`. In Numeric sort mode, each rendered `<a class="m-link">` stores its real DATA index via `data-idx`, computed at render time as `DATA.findIndex(d => d[1] === m)` — not the loop counter. This ensures the correct object is always loaded regardless of current sort order.

---

## CSS Architecture

### Design tokens (CSS custom properties)

All colours, backgrounds, and borders are CSS custom properties defined in `css/mm-shared.css`. Three themes are supported, each set via `data-theme` attribute on `<html>`:

| `data-theme` value | Description |
|--------------------|-------------|
| `night` | Deep black background, red-spectrum text — primary field use |
| `light` | Warm parchment atlas — day planning |
| `dark` | Pure black, white/grey text, no red effects — space terminal |

Switching theme is a single `setAttribute("data-theme", …)` call on the root element — no JavaScript colour manipulation.

Key token groups:
- `--red-bright`, `--red-mid`, `--red-dim` — red spectrum used for active elements, night mode text, accents
- `--bg`, `--bg2`, `--bg3` — background layers
- `--text-data`, `--text-sub`, `--text-meta`, `--text-dim` — text hierarchy
- `--border`, `--border2`, `--border3` — border hierarchy
- `--modal-bg`, `--modal-border`, `--modal-overlay` — modal surfaces

### Three-state theme system

`index.html` maintains two localStorage keys to express three visual states:

| `mm_mode` | `mm_theme` | Result |
|-----------|------------|--------|
| `night` | (any) | `data-theme="night"` — red on black |
| `day` | `light` | `data-theme="light"` — parchment |
| `day` | `dark` | `data-theme="dark"` — space terminal |

The Night/Day toggle switches `mm_mode`. The Light/Dark theme option in the Settings menu switches `mm_theme` and is only active in Day mode.

The `docs/` pages (about, faq, resources, user-guide) read both keys on load and apply the correct theme. Their toggle buttons write `mm_mode` only (night/day), preserving whatever `mm_theme` was last set in index.html.

### Night mode UI icon system

Emoji icons in the UI are wrapped in `<span class="ui-icon" data-night="[LABEL]">emoji</span>`. The CSS:

```css
[data-theme="night"] .ui-icon { font-size: 0; }
[data-theme="night"] .ui-icon::after {
  content: attr(data-night);
  font-size: 11px;
  font-family: 'Share Tech Mono', monospace;
  color: var(--red-bright);
}
```

This hides the emoji (`font-size: 0` removes it from layout) and renders the ASCII fallback label in red — entirely in CSS with no JavaScript required.

All arrow icons (⬇ ⬆ ⤓ ⎙) in the Data Sync and Reports menus are wrapped with this system so they render as red ASCII labels in night mode rather than coloured Unicode emoji.

### Night mode menu styling

All menu text is explicitly brightened in night mode via `[data-theme="night"]` overrides:

- `.nav-menu-link`, `.top-bar-item`, `.top-bar-dd-item` — `var(--text-sub)` at rest, `var(--text-data)` on hover
- `.nav-menu-section`, `.log-menu-section` (DATA SYNC, REPORTS, etc.) — `var(--red-bright)`, `opacity: 1`
- `.danger` items (Reset All Data) — `var(--red-bright)`
- `.note-input` underline — `var(--red-dim)` (approximately half the brightness of the text above it)

### Type badge system

Object type symbols are rendered as plain text characters inside `<span class="type-badge badge-[type]">`. In day mode, all badge classes set `color: #000; background: transparent`. In night mode, a single override rule sets all badges to `var(--red-bright)`.

The symbol characters (`@`, `●`, `◑`, `*`, `+`, `◎`, `~`, `○`, `✕`, `::`, `·`) are drawn from Unicode Basic Latin and Geometric Shapes to guarantee font coverage on all systems. `font-family: 'Courier New', monospace` is forced on the badge span to prevent OS-level emoji substitution.

### Lightbox image filter

In night mode, object photographs are filtered via CSS:

```css
[data-theme="night"] .lb-img-wrap img {
  filter: sepia(100%) brightness(50%) hue-rotate(-50deg);
}
```

This converts photographs to dim red-tinted images without JavaScript, preserving dark adaptation when the lightbox is opened at the telescope.

### Two stylesheets

The HTML contains two `<style>` blocks. The first is the interactive screen stylesheet. The second is a `@media print` stylesheet. Both maintain `colgroup` width definitions and type badge rules to stay in sync.

---

## Navigation

### Desktop (viewport > 768px)

A persistent top menu bar spans the full width of the page below the header area. It contains four dropdown menus:

| Menu | Items |
|------|-------|
| **Data** | Save…, Restore…, Import…, Export… |
| **Reports** | Generate PDF…, Generate CSV…, Print… |
| **Docs** | About, FAQ, Resources, User Guide |
| **Settings** | Session Setup…, Light/Dark Theme toggle, Reset All Data… |

The Night/Day toggle remains in the top-right corner at all times. The hamburger button and gear button are hidden on desktop.

Only one dropdown is open at a time. Clicking outside or opening another dropdown closes the current one. All dropdown items wire to the same underlying action functions as the mobile hamburger.

### Mobile (viewport ≤ 768px)

The top menu bar is hidden. All navigation and actions are consolidated into a single hamburger menu (top-left) containing all four sections with full content:

- **Data Sync** — Save, Restore, Import, Export
- **Reports** — Generate PDF, Generate CSV, Print
- **Docs** — About, FAQ, Resources, User Guide
- **Settings** — Session Setup, Theme toggle, Reset All Data

The Night/Day toggle is moved from the top-right corner to a centred position directly below the Observed progress bar, for easier thumb reach on phones.

### Mobile detection

Mobile layout is applied via an `is-mobile` class on `<html>`, set by JavaScript:

```js
function applyMobile() {
  if (window.innerWidth <= 768) {
    document.documentElement.classList.add("is-mobile");
  } else {
    document.documentElement.classList.remove("is-mobile");
  }
}
applyMobile();
window.addEventListener("resize", applyMobile);
```

Detection is based solely on viewport width — no user-agent sniffing, no touch detection. A `resize` listener ensures correct behaviour on orientation change or browser window resize.

The top bar and hamburger visibility are both driven by this same `is-mobile` class (not by CSS media queries), eliminating any mismatch between JS and CSS breakpoints.

---

## Mobile Table Layout

On mobile the full desktop table (10+ columns) is replaced with four columns:

| Column | Width | Content |
|--------|-------|---------|
| Thumbnail (`col-thn`) | 52px fixed | Object thumbnail image — tap to open lightbox |
| Description (`col-mobile-obj`) | ~40% of table | M# (Cinzel, 13px) · Informal name (italic, 12px) · Type · Constellation (mono, 10px) — stacked |
| Checkbox (`col-chk`) | Shrink-to-fit | Observed toggle |
| Notes (`col-note`) | ~40% of table | Inline notes input (`flex: 1`) + AL record button (✎) |

Description and Notes are the two widest columns, sharing the available space equally. Thumbnail and checkbox shrink to their minimum widths.

All other columns (Marathon #, Type, Constellation, Magnitude, Size, RA, Dec, PSA page) are hidden on mobile via `html.is-mobile` CSS selectors. The `col-mobile-obj` cell exists in every row in the DOM at all times and is shown only when `is-mobile` is present.

---

## Modals

All modals are static HTML present in the DOM at all times, toggled via `.open` class. There are seven modals:

| ID | Purpose |
|----|---------|
| `lbOverlay` | Image lightbox |
| `recOverlay` | Full AL observation record form |
| `sessOverlay` | Session setup |
| `rptOverlay` | AL report generation |
| `importOverlay` | CSV/JSON import conflict resolution |
| `confOverlay` | Custom themed confirm dialog |
| `qsOverlay` | Quick Start guide (shown on first visit) |

### Custom confirm dialog

The `confOverlay` modal replaces all three `window.confirm()` calls with a reusable themed confirmation modal driven by:

```js
showConfirm({ title, body, warning, okLabel, cancelLabel }, onOkCallback)
```

Because it uses CSS variables, it renders correctly in all three themes. The browser's native dialog (always light-coloured) is never shown.

---

## PDF Generation

The AL submission report is built entirely in the browser using jsPDF and jspdf-autotable, loaded from CDN at the moment of invocation. Report structure:

1. **Cover page** — title, observer name, club/subtitle, summary statistics (total records, objects with descriptions, date range)
2. **Observation index** — compact table, one row per completed record
3. **Individual record pages** — one full page per object with all four sections of the observation record

A standalone Python equivalent (`al_report.py`) is also in the repository for command-line PDF generation.

---

## File Structure

```
mmol/
├── index.html              Single-file application
├── CNAME                   mmol.thgnetworks.com
├── css/
│   └── mm-shared.css       Shared design tokens, layout, all three theme definitions
├── fonts/                  Self-hosted typefaces (7 TTF files)
│   ├── Cinzel-Regular.ttf
│   ├── Cinzel-SemiBold.ttf
│   ├── Cinzel-Bold.ttf
│   ├── ShareTechMono-Regular.ttf
│   ├── CrimsonPro-Light.ttf
│   ├── CrimsonPro-Regular.ttf
│   └── CrimsonPro-LightItalic.ttf
├── images/                 Object photos (not in repo)
│   └── M1.jpg … M110.jpg
├── charts/                 Finder charts (not in repo)
│   └── M1.png … M110.png
└── docs/
    ├── faq.html
    ├── resources.html
    ├── about.html
    └── user-guide.html
```

### Path conventions

All `docs/` pages use `../` relative paths for fonts, the shared stylesheet (`../css/mm-shared.css`), and links back to `../index.html`. Sibling links between doc pages use relative names only (e.g. `faq.html`, `resources.html`).

---

## Deployment

Hosted on GitHub Pages (free tier, public repository required) with a custom subdomain.

| Setting | Value |
|---------|-------|
| Repository | github.com/thg4web/mmol (public) |
| Branch | main, root folder |
| Custom domain | mmol.thgnetworks.com |
| DNS record | CNAME `mmol` → `thg4web.github.io` |
| HTTPS | Enforced via GitHub Pages / Let's Encrypt |

Deployments are push-based — any commit to `main` triggers a rebuild within ~60 seconds. The `CNAME` file must be present in the repository root on every push.

---

## Design Decisions

**No framework** — Intentionally vanilla JS/CSS/HTML so the tool can be opened directly from a filesystem without a dev server, forked without a build pipeline, and read by anyone with basic web knowledge.

**No server** — All data lives in localStorage. Nothing is ever transmitted. This is an explicit design choice for privacy and for reliability at remote dark-sky sites without cell coverage.

**Marathon order as default** — The table opens in marathon sequence, not M-number order, because the primary use case is live observation during a marathon night. Numeric sort is available for reference and cross-checking.

**Red-first night mode** — Every visual element has been reviewed for night-vision impact. Photographs are filtered to red in night mode. Emoji icons are replaced with ASCII text labels. Arrow and print icons use the `ui-icon` system so they render as red text rather than coloured Unicode glyphs. Confirmation dialogs use CSS variables so they never display a browser-native white dialog. Menu section headers and action labels are explicitly brightened in night mode. The AL record button fills solid red when populated so its status is readable with minimal eye movement.

**Three-state theme** — Night mode (red on black) is distinct from the Day/Dark space terminal theme (white on black). Night mode is for field use at the telescope; Day/Dark is for indoor use in low-light environments. Day/Light (parchment) is for planning in normal light.

**ASCII type symbols** — Colored dots were replaced with monospace ASCII symbols because color alone fails in a red-only display environment. The symbols convey type information through glyph shape rather than hue.

**Viewport-width mobile detection** — Mobile layout is triggered purely by `window.innerWidth <= 768` with a resize listener. No user-agent parsing or touch detection is used, ensuring consistent behaviour across all devices and browsers regardless of UA string.

**Mobile-first column design** — The mobile table shows four columns: thumbnail, combined description (M#, name, type, constellation), checkbox, and notes with AL button. Description and notes share the available width equally as the two widest columns, keeping all critical information and actions accessible without horizontal scrolling.

**Unified hamburger on mobile** — All four action categories (Data Sync, Reports, Docs, Settings) are consolidated into a single hamburger menu on mobile, eliminating the separate gear button and providing one consistent entry point for all functions.
