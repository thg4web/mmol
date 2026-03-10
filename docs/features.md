# Messier Marathon Observer's Log — Feature Reference

A complete listing of all features implemented in the log. Intended as a quick reference for contributors and observers wanting to understand what the tool can do.

---

## Observation Table

**110 objects in marathon search order** — The Don Machholz sequence (west to east), beginning with objects that set first after dusk and ending with those rising last before dawn. This is the optimal order for attempting a one-night complete observation of the entire catalog.

**Object sort toggle** — The Object column header is clickable. It toggles between Marathon order (the default, numbered 1–110 by marathon position) and Numeric order (M1–M110 by Messier number). The header label updates to reflect the active mode (`Marathon ▲` / `Numeric ▲`). Filters, search, and the lightbox all work correctly in both modes.

**Column layout** — Each row shows: marathon position (#), Messier designation (Object), type symbol (Type), constellation (Con), visual magnitude (Mag), angular size (Size), right ascension (RA), declination (Dec), Pocket Sky Atlas page reference (PSA Pg), observation checkbox (✓), and a combined field notes / AL record cell.

**Angular Size column** — Apparent dimensions in arcminutes sourced from the SEDS marathon data (Hartmut Frommert). Two-axis objects show width × height (e.g. `178×63`); circular objects show a single value (e.g. `110'`).

**Object type symbols** — Each type is represented by a monospace ASCII symbol that reads clearly in red night mode where colour differentiation is not available:

| Symbol | Type |
|--------|------|
| `@` | Spiral Galaxy |
| `●` | Elliptical Galaxy |
| `◑` | Lenticular Galaxy |
| `*` | Irregular Galaxy |
| `+` | Open Cluster |
| `◎` | Globular Cluster |
| `~` | Diffuse Nebula |
| `○` | Planetary Nebula |
| `✕` | Supernova Remnant |
| `::` | Star Cloud |
| `·` | Double Star / Asterism |

In day mode the symbols are rendered in black on a transparent background. In night mode all symbols switch to `var(--red-bright)` with a transparent background — the type column never bleaches adapted eyes. The **Object Type Key** legend below the toolbar uses the same symbols.

**Filter buttons** — Filter the table by object type (Galaxy, Open Cluster, Globular Cluster, Nebula) or by observation status (Unobserved). Click All to reset. Filters combine with the search field.

**Search** — Filters by M-number, type name, or constellation abbreviation. Examples: `Sgr` shows Sagittarius objects; `glob` shows globulars; `M10` shows all entries containing "M10". The × button clears instantly. Search and filter are fully combinable.

**Progress bar** — Shows observed / 110 as a percentage bar. Updates in real time as checkboxes are ticked or unticked.

**Observation checkboxes** — Ticking records a hidden ISO timestamp. Unticking removes it. The timestamp feeds into the full AL record and the CSV export as Date Observed.

**Field Notes ↔ AL Description sync** — The inline field notes cell and the Description field in the full AL record modal are kept in two-way sync. Any text typed in the table seeds the Description when the form is opened; any text saved in the form writes back to the table.

---

## Image Lightbox

Click any Messier number in the Object column to open the lightbox for that object.

**Left panel** — Loads `images/M[n].jpg` from the local `images/` folder. Shows a styled placeholder if the file is absent.

**Right sidebar** — Full catalog data: object name, type, constellation, magnitude, angular size (arcmin), right ascension, declination, PSA page reference.

**Finder Chart button** — Opens `charts/M[n].png` from the local `charts/` folder in a new tab. A red-bordered warning notice is shown before the button, reminding the observer to enable a device red filter before viewing — charts are white-background images that will harm dark adaptation.

**Stellarium Web button** — Opens the free online 3D sky simulator centred on the selected object (internet required).

**Night mode image treatment** — In night mode, lightbox photos are automatically filtered (`sepia(100%) brightness(50%) hue-rotate(-50deg)`) to render as dim red-tinted images, preserving dark adaptation when viewing object photos at the telescope.

**Night mode attribution** — Image attribution text overlaying the photo renders in `var(--red-bright)` in night mode.

**Correct object in any sort mode** — The lightbox always loads the object whose M-number was clicked, regardless of whether the table is in Marathon or Numeric sort order.

**Navigation** — ← / → footer buttons step through all 110 objects in order. Keyboard: arrow keys to navigate, Escape or click-outside to close.

---

## Night / Day Mode

The toggle in the top-right corner switches between two display themes. The preference is saved in `mm_theme` localStorage and restored on every page load.

**Night mode** — Near-black background, red-spectrum text and controls throughout.

**Day mode** — Warm parchment background, dark ink, maroon accents.

**UI icon system** — All icon buttons are wrapped in `<span class="ui-icon" data-night="[LABEL]">`. In night mode the emoji is hidden (`font-size: 0`) and a short ASCII label is shown via `::after { content: attr(data-night) }` in `var(--red-bright)` — so no bright or coloured emoji are ever displayed at the eyepiece:

| Button | Day emoji | Night label |
|--------|-----------|-------------|
| Theme toggle (to night) | 🌙 | `[N]` |
| Theme toggle (to day) | ☀ | `[D]` |
| Generate AL Report | 📄 | `[RPT]` |
| Session Setup | ⚙ | `[CFG]` |
| GPS Locate | 📍 | `[GPS]` |
| Finder Chart | 🗺 | `[MAP]` |
| Stellarium Web | 🔭 | `[SKY]` |
| Chart warning | ⚠ | `[!]` |
| Image placeholder | ✦ | `[+]` |
| User Guide link | 📖 | `[DOC]` |

**AL record button states** — The ✎ icon on each row shows in bright red when no record is saved. When a record exists, the button fills with a solid bright red background and dark foreground — a filled indicator visible at a glance across the whole table without needing to look closely.

**Type symbols in night mode** — All type badge symbols switch to `var(--red-bright)` with a transparent background, matching all other night-mode elements.

---

## Themed Confirmation Dialogs

All three destructive-action confirmations use a custom `showConfirm()` modal rather than the browser's native `window.confirm()`. The modal inherits all current CSS variables so it renders correctly in both night and day mode — no white browser dialog is ever shown at the telescope.

| Action | Title |
|--------|-------|
| ☰ Log → Reset All Data | Reset Observation Data |
| Session Setup → Clear Session | Clear Session Defaults |
| Import AL Records (conflict) | Import AL Records |

Each dialog shows a title, a summary of what data will be affected, an optional warning message, and labelled OK / Cancel buttons.

---

## Session Setup

Open via **☰ Log → Session Setup**. Sets defaults that auto-fill blank fields in every full observation record opened during the session. Existing saved data is never overwritten.

**Fields:** observing site, seeing (Antoniadi scale I–V), transparency (1–5), limiting magnitude, telescope/instrument, aperture (mm), eyepiece, magnification.

**GPS locate** — The 📍 Locate button requests GPS coordinates from the device. On success, coordinates are reverse-geocoded via OpenStreetMap Nominatim to produce a place name for the Site field. Raw coordinates are also stored in the session.

**Session indicator** — A pulsing "Session active" badge in the toolbar shows while any session default is set.

**Clear Session** — Removes all defaults without affecting any saved observation records.

---

## Full AL Observation Records

Click the **✎** button on any row to open the full record form. The button is outlined when no record is saved; it fills solid red when data exists.

**Auto-fill priority:** existing saved data → session defaults → checkbox timestamp → current date/time. The Description pre-populates from any inline Field Notes text.

**Fields:** Date, Time, Observing Site, Seeing (Antoniadi I–V), Transparency (1–5), Limiting Magnitude, Telescope/Instrument, Aperture (mm), Eyepiece, Magnification, Observation Description, Sketch Notes.

---

## Log Menu (☰ Log ▼)

| Item | Function |
|------|----------|
| ⎙ Print | Clean black-on-white print stylesheet + browser print dialog |
| ⬇ Export CSV | Timestamped CSV of all 110 rows — marathon order, catalog data, observed status, date, notes |
| ⬆ Import CSV | Reads a previously exported CSV; offers Merge (fill blanks) or Overwrite (replace all) |
| ⬇ Export AL Records | Timestamped JSON of all full records + observation timestamps |
| ⬆ Import AL Records | Merges exported JSON back in; shows new-vs-conflict counts in a themed confirmation dialog |
| 📄 Generate AL Report | Enter name/club, generate a multi-page PDF in the browser — cover, index, one page per record |
| ⚙ Session Setup… | Opens the session defaults modal |
| ↺ Reset All Data… | Shows counts of all saved data, then a themed confirmation before permanently clearing everything |

---

## Data Storage

All data is stored in `localStorage` — never transmitted to any server.

| Key | Contents |
|-----|----------|
| `mm_checks` | Observed status per object |
| `mm_notes` | Inline field notes per object |
| `mm_times` | Observation timestamps per object (ISO string) |
| `mm_fullnotes` | Full AL record data per object (structured JSON) |
| `mm_session` | Session defaults including GPS coordinates |
| `mm_theme` | Night or day mode preference |
| `mm_visited` | Set to `"1"` when "Don't show again" is ticked in the Quick Start |

---

## Site Pages

| Page | Location | Purpose |
|------|----------|---------|
| Log | `index.html` | Main observation tool |
| FAQ | `docs/faq.html` | Common questions about the marathon, AL program, and the log |
| User Guide | `docs/user-guide.html` | Full usage documentation |
| Resources | `docs/resources.html` | Curated external links |
| About | `docs/about.html` | Project information, credits, acknowledgements |

All pages share the same font stack (Cinzel, Share Tech Mono, Crimson Pro), CSS design system (`css/mm-shared.css`), theme toggle, and hamburger navigation menu. The nav dropdown on every page includes a link back to the Log as the first item.

---

## File Structure

```
mmol/
├── index.html              Main observation log
├── CNAME                   mmol.thgnetworks.com
├── css/
│   └── mm-shared.css       Shared design tokens and layout
├── fonts/                  Self-hosted TTF files (7 faces)
├── images/                 Object photos — M1.jpg … M110.jpg
├── charts/                 Finder charts — M1.png … M110.png
└── docs/
    ├── faq.html
    ├── resources.html
    ├── about.html
    └── user-guide.html
```
