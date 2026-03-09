# Messier Marathon Observer's Log — Project Design Summary

---

## Mission Statement

The Messier Marathon Observer's Log is a self-contained, offline-capable web tool built for amateur astronomers pursuing the Astronomical League Messier Observing Program. Its purpose is to replace paper log sheets with a clean, distraction-free digital record that works in the field — including at dark sites with no internet connection — without requiring any app install, account, or subscription.

The guiding principle is **utility without compromise on atmosphere**. The interface is designed to feel like a well-made star atlas: precise, purposeful, and visually at home under a red flashlight. Every design decision, from the night-vision colour palette to the monospaced coordinate columns, serves the observer first.

---

## Development History

### Foundation
The project began as a single HTML file containing all 110 Messier objects in marathon observation order, sourced from the standard Astronomical League sequence used by observers at the spring new moon window. The data array stores each object's Messier number, type, constellation, magnitude, J2000.0 coordinates, and Pocket Sky Atlas page reference.

### Design System
A shared stylesheet (`mm-shared.css`) gives the project a consistent visual identity across all pages, built entirely on CSS custom properties that enable a full dark/light theme swap with a single attribute toggle on `<html>`.

Two complete themes were developed in parallel:

- **Dark / Night mode** — deep black background (`#030305`), red-spectrum text and accents for night-vision preservation.
- **Light / Day mode** — warm parchment atlas feel (`#f4ece0`), dark ink, maroon accents. Suited for planning sessions indoors.

Theme preference is saved to `localStorage` and restored on every page load without a flash of unstyled content.

Typography uses three typefaces, all self-hosted from local `fonts/` files:

- **Cinzel** — classical Roman capitals for headers and labels
- **Share Tech Mono** — monospaced data columns and coordinates
- **Crimson Pro** — italic body copy and prose pages

### Core Observation Log (`index.html`)
The main page provides the full 110-object table with:

- Marathon observation order (not Messier number order)
- Header subtitle: *Observation Log & AL Certificate Record*
- Per-object checkbox and free-text field notes, both persisted in `localStorage`
- Automatic silent observation timestamp (ISO string) recorded when a checkbox is ticked; stored in `mm_times`
- Animated progress bar tracking objects observed out of 110
- Filter buttons: All / Galaxy / Open Cluster / Globular / Nebula / Unobserved
- Live search filtering by object number, type, or constellation
- Type colour-coded dot badges (11 types) with a legend panel
- Print stylesheet producing a clean black-on-white log sheet

### Image Lightbox
Clicking any object row opens a modal overlay with:

- Object image from local `images/M[n].jpg`
- Attribution overlay fetched from `images/M[n]-attrib.txt`
- Full object data in a right-hand sidebar
- Local finder chart button linking to `charts/M[n].png`
- Stellarium Web button (deep-linked by Messier name)
- Keyboard navigation (arrow keys, Escape), click-outside-to-dismiss, prev/next footer controls

### Log Menu
A single **☰ Log ▼** dropdown keeps the toolbar clean:

- ⎙ Print
- ⬇ Export CSV / ⬆ Import CSV
- ⬇ Export AL Records / ⬆ Import AL Records
- 📄 Generate AL Report…
- ⚙ Session Setup…
- ↺ Reset All Data…

### CSV Export / Import
Export produces a timestamped `YYYY-MM-DD-HHmmss-observing-log.csv` with all catalog fields, observed status, date observed (from the hidden timestamp), and field notes. Import parses by column header name (position-independent), detects conflicts, and presents an Overwrite / Merge choice before applying.

### AL Observation Record Modal
A small **✎** button at the right of every notes cell opens a full-screen modal with all fields required by the Astronomical League Messier Club program:

- When & Where: date, time, observing site / location
- Sky Conditions: seeing (Antoniadi 1–5), transparency (1–5), limiting magnitude
- Equipment: telescope / instrument, aperture (mm), eyepiece, magnification
- Observation: written description (required for AL certificate), sketch notes

The ✎ icon is outlined (dim) when no record exists and renders as a solid filled red button when a record has been saved.

**Field Notes ↔ Description sync:** when opening the modal, Field Notes text always seeds the Description field (overriding any previously saved description), so the inline note is always the inbound source of truth. On save, the Description is written back to Field Notes and to the visible table cell, keeping both fields in sync through every open/save cycle.

### AL Records JSON Export / Import
Full observation records are stored in `mm_fullnotes` (localStorage) and can be backed up independently of the CSV log. Export produces a timestamped `YYYY-MM-DD-HHmmss-al-records.json` with a version field and ISO export timestamp. Import merges incoming records with existing ones, showing new-vs-conflict counts before confirming.

### In-Browser AL Report PDF Generator
Accessed via **☰ Log → Generate AL Report…**, this feature uses jsPDF and jspdf-autotable (loaded from cdnjs) to generate a complete submission-ready PDF entirely in the browser — no server, no Python required.

A small setup dialog collects observer name and optional club/subtitle. The generated PDF contains:

1. **Cover page** — dark header band, title, observer name, subtitle, summary statistics (record count, date range, generation date)
2. **Observation Index** — compact one-row-per-object table
3. **One page per record** — When & Where, Sky Conditions, Equipment, Observation Description + Sketch Notes, formatted for AL submission

Filename format: `YYYY-MM-DD-HHmmss-al-report.pdf`. A standalone Python version (`al_report.py`) is also available for command-line generation.

### Session Setup
A modal accessed via the Log menu allows the observer to set common defaults once at the start of each night — location, sky conditions, and equipment — that auto-fill blank fields in every observation record opened during the session. Existing saved data is never overwritten.

The **📍 Locate** button calls `navigator.geolocation`, then reverse-geocodes via OpenStreetMap Nominatim (free, no API key). A pulsing **Session active** indicator pill appears in the toolbar while any session default is set. Session data is persisted in `mm_session` localStorage.

### Quick Start Guide
On the first visit to the site (detected by the absence of all `mm_*` data keys and the `mm_visited` flag), a modal overlay opens automatically presenting:

- The two use cases: **Casual Messier Checklist** and **AL Messier Certificate**
- A six-item feature summary grid
- A "Don't show again" checkbox (sets `mm_visited` on dismiss)
- A link to the full User Guide

The guide is always presented in Day mode regardless of the user's current theme preference; the original theme is restored on dismiss.

### Companion Pages

| Page | Purpose |
|------|---------|
| `faq.html` | Common questions about the marathon, AL program, and how to use the log |
| `docs/user-guide.html` | Full HTML user guide — use cases, every feature, tables, notes, AL program reference. Matches site aesthetic with the same fonts, theme toggle, and shared CSS. |
| `resources.html` | Curated external links — SEDS, NASA Hubble, AL program, Deep Sky Watch, AstroPixels, marathon planners, printable logbooks, and a featured video. Styled as a resource directory matching the site design. |
| `docs/css-cheatsheet.md` | Internal developer reference — CSS classes, colour variables, layout patterns. Not deployed to the public site. |

All prose pages share `mm-shared.css`, the same `@font-face` declarations, the theme toggle, and `mm_theme` localStorage persistence.

### Footer Navigation
All pages carry a consistent footer nav linking between: **← Back to Log · FAQ · User Guide · Resources**. Paths are adjusted for each page's location in the directory tree.

---

## localStorage Keys

| Key | Contents |
|-----|----------|
| `mm_checks` | Observed state per object (keyed by M-id) |
| `mm_notes` | Brief field notes per object (synced with AL description on save) |
| `mm_times` | ISO observation timestamp per object |
| `mm_fullnotes` | Full AL record data per object (structured JSON) |
| `mm_session` | Session defaults: location, sky conditions, equipment, lat/lon |
| `mm_theme` | `"dark"` or `"light"` |
| `mm_visited` | Set to `"1"` when user ticks "Don't show again" in Quick Start |

---

## Infrastructure

- No frameworks, no build tools, no server-side code
- Fonts served locally from `fonts/` — no external CDN calls at runtime
- Images and charts stored locally — works completely offline after first load
- jsPDF + jspdf-autotable loaded from cdnjs for in-browser PDF generation (requires network on first load)
- GitHub Pages deployment via `.github/workflows/deploy.yml`; only public-facing files are published
- Nominatim geolocation call is the only other runtime network request (optional; gracefully skipped offline)

---

## Repository Structure (Current)

```
/
├── index.html                    ← Main observation log
├── faq.html                      ← Frequently asked questions
├── resources.html                ← Curated external resource links
├── mm-shared.css                 ← Shared design system stylesheet
├── README.md                     ← GitHub repo overview and setup guide
├── al_report.py                  ← Standalone PDF report generator (Python / ReportLab)
├── create_attributions.sh        ← Generates SEDS attribution .txt files
├── generate_charts.py            ← Offline finder chart generator (matplotlib)
├── .gitignore
├── .github/
│   └── workflows/deploy.yml      ← GitHub Actions Pages deploy
├── fonts/                        ← Self-hosted typefaces (not tracked in git)
├── images/                       ← Object photos + attribution files (not tracked)
├── charts/                       ← Finder chart PNGs M1–M110 (not tracked)
└── docs/                         ← Documentation
    ├── user-guide.html           ← Full user guide (public-facing)
    ├── features.md               ← Feature reference (internal)
    ├── project-design-summary.md ← This file (internal)
    └── css-cheatsheet.md         ← CSS developer reference (internal)
```

---

## Remaining Open Items

### Data Verification
All 110 object coordinates in the DATA array should be cross-checked against an authoritative source (e.g. SEDS Messier catalogue or SIMBAD) to confirm RA/Dec accuracy and PSA page numbers.

### Planned Future Features

**Session Planner** — a lightweight view showing which objects are observable on a given night based on date and observer latitude, highlighting the critical western and eastern horizon objects that define the marathon window.

**Equipment & Site Profiles** — named presets for Session Setup stored in `mm_profiles` localStorage, so regular observers can switch between home and dark-site configurations without re-entering equipment details.

**Observation Statistics Dashboard** — mines `mm_times` and `mm_fullnotes` for charts and summaries: objects per session, type breakdown, seeing distribution, session timeline.

**Observation Notes History** — keep the last 2–3 versions of the description with timestamps, so edits can be reviewed or reverted.

---

*Last updated: March 2026*
