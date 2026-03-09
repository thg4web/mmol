# Messier Marathon Observer's Log — Feature Guide

*A complete reference to everything the site can do.*

---

## What Is This?

The Messier Marathon Observer's Log is a free, browser-based tool for amateur astronomers observing the 110 objects in the Messier catalogue. It is designed around two use cases:

1. **Casual Messier Checklist** — a fast, red-light-friendly checklist for tracking which objects you have seen, with optional brief notes. No forms to fill in. Check, note, done.
2. **AL Messier Certificate** — a structured observation record for earning the Astronomical League Messier Club certificate. Per-object forms capture every required field; a one-click PDF generates your submission document.

Nothing is installed. Nothing is sent to a server. All data lives in your browser.

---

## Quick Start Guide

On your very first visit — before any observations are recorded — the site opens a **Quick Start Guide** modal automatically. It explains the two use cases above, summarises the key features at a glance, and links to the full User Guide.

- **"Don't show again"** checkbox — tick it before dismissing to suppress the guide on future visits. Leave it unticked and the guide will reappear on any browser where no data exists yet.
- The guide is always displayed in **Day mode** (warm parchment) for easy reading, then restores whichever theme was active before you opened it.
- The full User Guide is always accessible at `docs/user-guide.html`, linked in the footer of every page.

---

## Night Vision Mode

The site opens in **dark / night mode** by default: a near-black background with all text and controls rendered in shades of red. Red light does not constrict the pupil or bleach rhodopsin the way white light does, so your eyes stay dark-adapted while you navigate the log.

A pill-shaped toggle in the top-right corner switches between:

- 🌙 **Night mode** — red-on-black, for use at the telescope
- ☀ **Day mode** — dark ink on warm parchment, for planning indoors

Your preference is remembered between visits.

---

## The Main Table

The centrepiece of the site is a 110-row table listing every Messier object in **marathon search order** — the sequence that gives the best statistical chance of observing all 110 in one night, beginning with western objects visible just after dusk and ending with eastern objects visible just before dawn. This sequence is different from Messier number order.

Each row shows:

| Column | What it tells you |
|--------|-------------------|
| **#** | Position in the marathon sequence (1–110) |
| **Object** | Messier designation — click to open the image lightbox |
| **Type** | Object type with a colour-coded dot (11 types) |
| **Con** | Constellation (IAU 3-letter abbreviation) |
| **Mag** | Visual magnitude |
| **RA / Dec** | J2000.0 coordinates |
| **PSA Pg** | Page in *Pocket Sky Atlas* (Sinnott) |
| **✓** | Observation checkbox |
| **Field Notes** | Brief inline notes field + full-record button |

A colour-coded legend panel below the toolbar identifies all 11 object types: spiral galaxy, elliptical galaxy, lenticular galaxy, irregular galaxy, open cluster, globular cluster, diffuse nebula, planetary nebula, supernova remnant, star cloud, and double star / asterism.

---

## Progress Bar

A bar at the top of the table shows your running count of observed objects (e.g. **47 / 110**). It fills and updates in real time as you check objects.

---

## Filtering and Search

Two controls above the table narrow what you see without losing your place:

**Filter buttons** — click to show only one category:
All · Galaxy · Open Cluster · Globular · Nebula · Unobserved

**Search box** — type any fragment to filter by Messier number, object type, or constellation abbreviation. Clears instantly with the × button.

Filters and search combine — e.g. "Globular" filter + "Sgr" in the search shows only globular clusters in Sagittarius.

---

## Checkboxes and Field Notes

Every object has a **checkbox** and a **notes field** in the same row.

- Check the box when you have seen the object. The row dims slightly and the progress bar advances.
- Type anything in the notes field — eyepiece used, sky conditions, a quick impression.
- Both are saved automatically to browser storage and survive page refreshes and browser restarts. You never need to click Save.
- When you check an object, an **observation timestamp** is silently recorded. It appears in the full observation record and in CSV exports, but never clutters the table.

### Field Notes ↔ AL Description sync

The Field Notes field and the Description field in the full AL record are kept in sync automatically:

- **Opening the AL record** — if Field Notes has text, it seeds the Description field so you are not starting from scratch. If Field Notes is blank, the previously saved description (if any) is restored.
- **Saving the AL record** — the Description is written back to Field Notes so the inline note always reflects your latest written observation.

This means you can type a quick shorthand note in the field at the telescope, then expand it properly in the AL record form later — and the table will update to show the full text when you save.

---

## Image Lightbox

Click any object's **M-number** in the table to open the lightbox modal. It shows:

**Left panel**
- Object photograph (from a local `images/` folder; a placeholder is shown if the image is absent)
- Image credit attribution (loaded from a paired text file)

**Right sidebar**
- Full catalog data: type, constellation, magnitude, RA, Dec, PSA page
- **🗺 Finder Chart** button — opens a local chart image from `charts/M[n].png`
- **🔭 Stellarium Web** button — opens the free online sky simulator centred on this object

**Footer**
- ← Prev / position indicator / Next → to step through all 110 objects
- Keyboard shortcuts: ← → to navigate, Escape or click outside to close

---

## Full AL Observation Records

For each object, a **✎** button sits at the right edge of the notes cell.

- **Outlined** (dim) — no full record saved yet
- **Filled red** — a full record has been saved for this object

Click it to open the full observation record form capturing everything the AL requires:

**When & Where** — date, time, observing site (pre-filled from observation timestamp and Session Setup)

**Sky Conditions** — seeing (Antoniadi 1–5), transparency (1–5), limiting magnitude

**Equipment** — telescope / instrument, aperture (mm), eyepiece, magnification

**Observation** — written description (required for AL certificate), sketch notes

The hidden observation timestamp is displayed at the bottom of the modal for reference. All full records are saved automatically to `mm_fullnotes`.

---

## Session Setup

At the start of each observing session, open **☰ Log → Session Setup** to enter shared defaults that will auto-fill blank fields in every observation record you open that night.

Fields available as session defaults: observing site, seeing, transparency, limiting magnitude, telescope / instrument, aperture, eyepiece, magnification.

Session values only fill gaps — any field with existing saved data is never overwritten.

**📍 Locate button** — requests GPS from the device, then reverse-geocodes to a place name via OpenStreetMap Nominatim (free, no API key). Raw coordinates are also stored in the session.

A pulsing **Session active** indicator appears in the toolbar while any default is set. **↺ Clear Session** removes all defaults without affecting saved records.

---

## The Log Menu

A single **☰ Log ▼** button opens a dropdown with all data management options:

| Item | What it does |
|------|--------------|
| ⎙ Print | Opens the browser print dialog with a clean black-on-white stylesheet |
| ⬇ Export CSV | Saves a complete observation log as a timestamped CSV |
| ⬆ Import CSV | Loads a previously exported CSV; Merge or Overwrite choice |
| ⬇ Export AL Records | Saves all full observation records as a timestamped JSON |
| ⬆ Import AL Records | Loads a previously exported JSON; conflict count shown before confirming |
| 📄 Generate AL Report… | Generates a formatted multi-page submission PDF in the browser |
| ⚙ Session Setup… | Opens the session defaults modal |
| ↺ Reset All Data… | Clears all data with a confirmation dialog |

---

## AL Report PDF Generator

**☰ Log → Generate AL Report…** opens a small dialog where you enter your name and an optional club / location subtitle. Clicking **Generate PDF** produces a complete multi-page PDF in the browser — no internet connection needed — and saves it as `YYYY-MM-DD-HHmmss-al-report.pdf`.

The PDF contains three sections:

1. **Cover page** — title, observer name, subtitle, summary statistics (record count, date range)
2. **Observation Index** — one compact row per object
3. **Individual record pages** — one full page per saved record with all four sections in a formatted layout ready for submission

A standalone Python version (`al_report.py`) is also available in the repository.

---

## Resetting Data

**☰ Log → Reset All Data…** shows a count of saved data before asking to confirm. Clears all `mm_*` localStorage keys permanently — export first.

---

## Browser Storage

| Key | Contents |
|-----|----------|
| `mm_checks` | Which objects you have observed |
| `mm_notes` | Brief field notes (kept in sync with AL description) |
| `mm_times` | Timestamps of each observation |
| `mm_fullnotes` | Full AL observation records |
| `mm_session` | Current session defaults |
| `mm_theme` | Night or day mode preference |
| `mm_visited` | Set when "Don't show again" is ticked in the Quick Start guide |

---

## Works Offline

After first load, the entire tool works without internet: table, checkboxes, notes, full record forms, lightbox, AL Report PDF generation, CSV/JSON export and import, and printing.

Features that require network access: **📍 Locate** (OpenStreetMap), **🔭 Stellarium Web** (external link), and the jsPDF library on first load.

---

## Site Pages

| Page | Purpose |
|------|---------|
| `index.html` | Main observation log |
| `faq.html` | Frequently asked questions |
| `docs/user-guide.html` | Full in-depth user guide |
| `resources.html` | Curated external links |

All pages share the theme toggle, font system, and `mm_theme` preference.

---

*Messier Marathon Observer's Log — clear skies!*
