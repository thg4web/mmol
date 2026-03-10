# Messier Marathon Observer's Log — Project Design Summary

## Overview

The Messier Marathon Observer's Log (MMOL) is a single-file browser application for logging observations of Charles Messier's 110 deep-sky objects. It requires no server, no framework, and no build step. The entire tool is delivered as one `index.html` with embedded CSS, JavaScript, and a static DATA array.

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

All colours, backgrounds, and borders are CSS custom properties defined on `[data-theme="dark"]` and `[data-theme="light"]` selectors in `css/mm-shared.css`. Switching theme is a single `setAttribute("data-theme", …)` call on the root element — no JavaScript colour manipulation.

Key token groups:
- `--red-bright`, `--red-mid`, `--red-dim` — red spectrum used for active elements, night mode text, accents
- `--bg`, `--bg2`, `--bg3` — background layers
- `--text-data`, `--text-sub`, `--text-meta`, `--text-dim` — text hierarchy
- `--border`, `--border2`, `--border3` — border hierarchy
- `--modal-bg`, `--modal-border`, `--modal-overlay` — modal surfaces

### Night mode UI icon system

Emoji icons in the UI are wrapped in `<span class="ui-icon" data-night="[LABEL]">emoji</span>`. The CSS:

```css
[data-theme="dark"] .ui-icon { font-size: 0; }
[data-theme="dark"] .ui-icon::after {
  content: attr(data-night);
  font-size: 11px;
  font-family: 'Share Tech Mono', monospace;
  color: var(--red-bright);
}
```

This hides the emoji (`font-size: 0` removes it from layout) and renders the ASCII fallback label in red — entirely in CSS with no JavaScript required.

### Type badge system

Object type symbols are rendered as plain text characters inside `<span class="type-badge badge-[type]">`. In day mode, all badge classes set `color: #000; background: transparent`. In night mode, a single override rule sets all badges to `var(--red-bright)`.

The symbol characters (`@`, `●`, `◑`, `*`, `+`, `◎`, `~`, `○`, `✕`, `::`, `·`) are drawn from Unicode Basic Latin and Geometric Shapes to guarantee font coverage on all systems. `font-family: 'Courier New', monospace` is forced on the badge span to prevent OS-level emoji substitution.

### Lightbox image filter

In night mode, object photographs are filtered via CSS:

```css
[data-theme="dark"] .lb-img-wrap img {
  filter: sepia(100%) brightness(50%) hue-rotate(-50deg);
}
```

This converts photographs to dim red-tinted images without JavaScript, preserving dark adaptation when the lightbox is opened at the telescope.

### Two stylesheets

The HTML contains two `<style>` blocks. The first is the interactive screen stylesheet. The second is a `@media print` stylesheet. Both maintain `colgroup` width definitions and type badge rules to stay in sync.

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

Because it uses CSS variables, it renders correctly in both night and day mode. The browser's native dialog (always light-coloured) is never shown.

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
│   └── mm-shared.css       Shared design tokens, layout, theme variables
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

**Red-first night mode** — Every visual element has been reviewed for night-vision impact. Photographs are filtered to red in night mode. Emoji icons are replaced with ASCII text labels. Confirmation dialogs use CSS variables so they never display a browser-native white dialog. The AL record button fills solid red when populated so its status is readable with minimal eye movement.

**ASCII type symbols** — Colored dots were replaced with monospace ASCII symbols because color alone fails in a red-only display environment. The symbols convey type information through glyph shape rather than hue.

**Best viewed on laptop or tablet** — The table has 10+ columns and is not designed for small phone screens. A notice in the Quick Start modal advises users of this.
