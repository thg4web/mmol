# Messier Marathon Observer's Log

A browser-based observation log for Charles Messier's catalog of 110 deep-sky objects — designed for the field, built around the Astronomical League Messier Club certificate, and distributed as open-source software.

**Live site:** [mmol.thgnetworks.com](https://mmol.thgnetworks.com)

---

## What it is

The log presents all 110 Messier objects in **marathon search order** (the Don Machholz sequence — west to east, objects that set first after dusk through objects that rise last before dawn). It serves two kinds of observers:

- **Casual checklist** — tick objects as you observe them, add brief eyepiece notes, watch the progress bar. Export as CSV at any time.
- **AL Messier Club certificate** — full per-object forms capturing every field required for the Astronomical League Messier Observing Program submission. Generate a formatted multi-page PDF report directly in the browser.

Both use cases share the same interface.

---

## Features

- **Night vision mode** — deep black background, red-spectrum text. All emoji icons automatically swap to short ASCII labels in night mode so no bright or coloured symbols appear at the eyepiece. Object photos in the lightbox are automatically red-filtered.
- **110 objects in marathon order** — with a clickable toggle to numeric (M-number) order for cross-reference
- **Angular size column** — apparent dimensions in arcminutes for every object
- **Monospace type symbols** — each object type has a distinct ASCII symbol that reads clearly in red-only night mode (see [features.md](docs/features.md) for the full symbol map)
- **Image lightbox** — click any M-number for the object photo, full catalog data, finder chart link, and Stellarium Web link; images are red-filtered in night mode
- **Session Setup** — set location (with optional GPS), sky conditions, and equipment once per night; values auto-fill every record opened during the session
- **Full AL observation records** — date, time, site, seeing (Antoniadi scale), transparency, limiting magnitude, scope, eyepiece, description, and sketch notes
- **AL Report PDF** — multi-page submission document (cover, index, one page per record) generated entirely in the browser — no internet required
- **Themed confirmation dialogs** — all destructive actions use a custom modal that follows the current night/day theme, so no browser-native white dialog ever appears at the telescope
- **CSV and JSON export / import** — back up and restore all data; move between devices; merge records without overwriting
- **Works offline** — after first load, every core feature works without internet
- **Best on laptop or tablet** — the table has 10+ columns and is designed for larger screens

---

## Telescope Requirements

We recommend a minimum of **6–8 inch (152–203 mm)** aperture for a comfortable marathon experience. 

---

## Quick Start

1. Clone or download the repository and open `index.html` in a browser — no build step, no server, no installation required.
2. The `images/`, `charts/` and `fonts/` folders ship with the repo (see below); the site falls back to system fonts if fonts are missing.
4. At the telescope, switch to **Night Mode** using the toggle in the top-right corner.
5. Use **Settings → Session Setup…** to set your location and equipment once for the night.
6. Check off objects as you observe them. Click the **✎** button for a full AL record.
7. Back up your data after every session with **Data → Save…** (full backup) or **Data → Export…** (records JSON).

---

## Repository Structure

```
mmol/
├── index.html              Main observation log
├── CNAME                   mmol.thgnetworks.com
├── css/
│   └── mm-shared.css       Shared design tokens and layout
├── fonts/                  Self-hosted TTF files
├── images/                 Object photos, thumbnails and attribution files
├── charts/                 Finder charts
├── data/                   Source tables (Messier list, marathon order)
└── docs/
    ├── faq.html
    ├── resources.html
    ├── about.html
    ├── user-guide.html
    ├── features.md
    ├── project-design-summary.md
    └── javascript-design.md
```

---

## Setting Up Media Files

### Object photos (optional)

Images live in `images/`, named in lowercase: `m1.jpg` through `m110.jpg`, with thumbnails `m1_thn.jpg` … and an attribution file `m1-attrib.txt` for each. File names are case-sensitive on the live site. Photos are the author's own or used with attribution; the SEDS Messier Catalog at [messier.seds.org](http://www.messier.seds.org/) is a good source for your own copy.

In night mode, all lightbox photos are automatically red-filtered via CSS — no separate night-mode image set is needed.

### Finder charts (optional)

PNG charts live in `charts/`, named `M1.png` through `M110.png` (capital M). They are red-on-black night-vision charts.

### Fonts

The seven self-hosted TTF files in `fonts/` are listed below. All are available from Google Fonts.

| File | Font | Weight |
|------|------|--------|
| `Cinzel-Regular.ttf` | Cinzel | 400 |
| `Cinzel-SemiBold.ttf` | Cinzel | 600 |
| `Cinzel-Bold.ttf` | Cinzel | 700 |
| `ShareTechMono-Regular.ttf` | Share Tech Mono | 400 |
| `CrimsonPro-Light.ttf` | Crimson Pro | 300 |
| `CrimsonPro-Regular.ttf` | Crimson Pro | 400 |
| `CrimsonPro-LightItalic.ttf` | Crimson Pro | 300 italic |

---

## Data and Privacy

All observation data is stored in your browser's `localStorage` — a private, local database on your own device. Nothing is ever sent to any server.

| localStorage key | Contents |
|-----------------|----------|
| `mm_checks` | Observed status per object |
| `mm_notes` | Inline field notes per object |
| `mm_times` | Observation timestamps (ISO string) |
| `mm_fullnotes` | Full AL record data (JSON) |
| `mm_session` | Session defaults and GPS coordinates |
| `mm_mode` | Night or day mode |
| `mm_theme` | Light or dark theme used in Day mode |
| `mm_equipment` | Imported equipment presets |
| `mm_locations` | Imported observing-location presets |
| `mm_visited` | Quick Start "don't show again" flag |

**Important:** localStorage is tied to one browser on one device and is cleared by browser data resets or private/incognito mode. Use **Data → Save…** (or **Export…**) after every observing session. Use **Data → Restore…** or **Import…** to move your data to a new device or browser.

---

## Deployment (GitHub Pages)

Summary:

1. Push the repo to `github.com/thg4web/mmol` (must be public)
2. Settings → Pages → Branch: `main` / `(root)` → Custom domain: `mmol.thgnetworks.com`
3. DNS: add a CNAME record `mmol` → `thg4web.github.io` at your registrar
4. Enforce HTTPS once DNS propagates (~15 minutes to a few hours)

The `CNAME` file must be present on every push, or the custom domain will need to be re-entered.

---

## Documentation

| File | Purpose |
|------|---------|
| `README.md` | This file — setup and overview |
| `docs/features.md` | Complete feature reference |
| `docs/project-design-summary.md` | Architecture and technical design notes |
| `docs/javascript-design.md` | Section-by-section JavaScript walkthrough |
| `docs/user-guide.html` | End-user documentation |
| `docs/faq.html` | Common questions |
| `docs/resources.html` | Curated external links |
| `docs/about.html` | Project information and credits |

---

## References

- **Marathon sequence:** Don Machholz, *Messier Marathon Observer's Guide*
- **Object data:** [SEDS Messier Database](http://www.messier.seds.org/) (Hartmut Frommert & Christine Kronberg)
- **Sky atlas references:** Roger Sinnott, *Pocket Sky Atlas* (Sky & Telescope)
- **AL program:** [Astronomical League Messier Observing Program](https://www.astroleague.org/messier-observing-program/)

---

## License

GPLv3 — free to use, share, and modify.

---

## Contact

Questions, feedback, or bug reports: mmol [at] thgnetworks [dot] com
