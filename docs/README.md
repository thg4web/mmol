# 🔭 Messier Marathon — Observer's Log

A beautifully designed, fully self-contained observation log for the **Messier Marathon** and the **Astronomical League Messier Observing Program**. Built for the field — red night-vision mode, offline-capable, no install, no account, no server.

![Night Mode](https://img.shields.io/badge/Night%20Mode-Red%20Vision-cc3322?style=flat-square)
![Objects](https://img.shields.io/badge/Objects-110%20Messier-gold?style=flat-square)
![AL Program](https://img.shields.io/badge/AL-Messier%20Club-8844aa?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

---

## Two Use Cases

| Use Case | What you need | How it works |
|----------|--------------|--------------|
| **Casual Messier Checklist** | Just track what you've seen | Check objects off, add brief notes, export CSV when done |
| **AL Messier Certificate** | Full records for award submission | Fill the ✎ form per object, use Session Setup to pre-fill equipment, generate a PDF report for your club coordinator |

---

## Features

- **Red night-vision mode** — deep black background with full red-spectrum text to preserve dark adaptation; toggle to warm parchment day mode for planning indoors
- **110 Messier objects** in optimal marathon search order, west to east
- **Per-object checkboxes and field notes** — persisted in `localStorage` across sessions
- **Automatic observation timestamp** — silently recorded when you check an object
- **Progress bar** — live count of observed vs. total objects
- **Filter and search** — filter by Galaxy / Open Cluster / Globular / Nebula / Unobserved; search by M#, type, or constellation
- **Image lightbox** — click any M# to open the object photo, full data sidebar, local finder chart, and Stellarium Web link
- **Full AL observation records** — per-object form (✎ button) with all AL-required fields; solid red icon when a record is saved
- **Field Notes ↔ Description sync** — notes seed the AL description on open; description writes back to notes on save
- **In-browser AL Report PDF** — generates a complete formatted submission PDF via the Log menu; no Python required
- **Session Setup** — set location and equipment once per night; auto-fills every new record; includes GPS locate
- **Quick Start Guide** — shown on first visit, explains the two use cases and links to the User Guide
- **Log menu** — Print, Export CSV, Import CSV, Export AL Records (JSON), Import AL Records (JSON), Generate AL Report, Session Setup, Reset All Data
- **Resources page** — curated links to databases, observing guides, marathon planners, and logbooks

---

## Quick Start

```
index.html              ← open this in any modern browser
faq.html                ← frequently asked questions
resources.html          ← external resource links
mm-shared.css           ← shared stylesheet (must be alongside index.html)
docs/user-guide.html    ← full user guide
fonts/                  ← self-hosted typefaces (see Fonts section below)
images/                 ← optional object photos
charts/                 ← optional local finder charts
```

No server, no build step, no dependencies.

---

## Fonts

Typography is self-hosted — no Google Fonts or CDN calls at runtime. Download the following from [Google Fonts](https://fonts.google.com) and place them in a `fonts/` folder:

| File | Family |
|------|--------|
| `Cinzel-Regular.ttf` | [Cinzel](https://fonts.google.com/specimen/Cinzel) |
| `Cinzel-SemiBold.ttf` | Cinzel |
| `Cinzel-Bold.ttf` | Cinzel |
| `ShareTechMono-Regular.ttf` | [Share Tech Mono](https://fonts.google.com/specimen/Share+Tech+Mono) |
| `CrimsonPro-Light.ttf` | [Crimson Pro](https://fonts.google.com/specimen/Crimson+Pro) |
| `CrimsonPro-Regular.ttf` | Crimson Pro |
| `CrimsonPro-LightItalic.ttf` | Crimson Pro |

---

## Adding Photos

Place JPEGs in an `images/` folder named `M1.jpg` … `M110.jpg`.

```bash
# Linux / macOS — wget
mkdir -p images && for i in $(seq 1 110); do
  wget -q --show-progress --tries=3 --timeout=15 \
    -O "images/M${i}.jpg" "http://www.messier.seds.org/Jpg/m${i}.jpg" \
    || echo "⚠ Failed: M${i}.jpg"
done
```

```bash
# macOS — curl
mkdir -p images && for i in $(seq 1 110); do
  curl -s -o "images/M${i}.jpg" "http://www.messier.seds.org/Jpg/m${i}.jpg" \
    && echo "✓ M${i}.jpg" || echo "⚠ Failed: M${i}.jpg"
done
```

Generate attribution text files: `chmod +x create_attributions.sh && ./create_attributions.sh`

---

## Finder Charts

Place PNGs in a `charts/` folder (`M1.png` … `M110.png`). Use `generate_charts.py` to produce red-on-black night-vision charts for all 110 objects:

```bash
pip install matplotlib numpy
python generate_charts.py
```

---

## Data Export and Import

All user data is managed via **☰ Log** in the toolbar.

**Observation Log (CSV)** — exports `YYYY-MM-DD-HHmmss-observing-log.csv` with marathon order, catalog data, observed status, observation timestamp, and field notes. On import choose Overwrite or Merge.

**AL Records (JSON)** — exports `YYYY-MM-DD-HHmmss-al-records.json` with all full observation records. Import merges with existing records and shows a conflict count before confirming.

---

## AL Observation Records

Click **✎** in any object's notes cell to open the full record form:

- **When & Where** — date, time, observing site (with optional GPS → place name lookup)
- **Sky Conditions** — seeing (Antoniadi 1–5), transparency, limiting magnitude
- **Equipment** — telescope, aperture, eyepiece, magnification
- **Observation** — written description (required for AL certificate), sketch notes

The ✎ icon is outlined when empty and fills solid red when a record is saved. The description field is always seeded from Field Notes on open, and written back to Field Notes on save.

---

## AL Report PDF

**☰ Log → Generate AL Report…** builds a complete submission PDF in the browser:

1. **Cover page** — title, observer name, club / location, summary statistics
2. **Observation Index** — one-row-per-object table
3. **Individual record pages** — one full page per saved record

Saved as `YYYY-MM-DD-HHmmss-al-report.pdf`. A Python command-line version is also available:

```bash
pip install reportlab
python al_report.py --records al-records.json \
  --observer "Your Name" --subtitle "Your Club" --output report.pdf
```

---

## Session Setup

**☰ Log → Session Setup** sets defaults that auto-fill blank fields in every record opened during the session. The **📍 Locate** button gets GPS coordinates from the device and reverse-geocodes to a place name via OpenStreetMap. A pulsing **Session active** pill appears in the toolbar while defaults are active.

---

## localStorage Keys

| Key | Contents |
|-----|----------|
| `mm_checks` | Observed state per object |
| `mm_notes` | Brief field notes per object (synced with AL description) |
| `mm_times` | ISO timestamp of each observation |
| `mm_fullnotes` | Full AL record data per object |
| `mm_session` | Current session defaults |
| `mm_theme` | `"dark"` or `"light"` |
| `mm_visited` | Set when "Don't show again" is ticked in Quick Start |

---

## GitHub Pages Deployment

1. Rename `messier_marathon.html` → `index.html`
2. Push to GitHub
3. **Settings → Pages → Source → GitHub Actions**

The included `.github/workflows/deploy.yml` publishes only public-facing files. Internal docs (`features.md`, `project-design-summary.md`, `css-cheatsheet.md`) and scripts are excluded from the live site. `docs/user-guide.html` **is** published.

---

## Browser Compatibility

Chrome, Firefox, Safari, and Edge (current versions). Requires `localStorage`. Geolocation requires HTTPS or `localhost`. The in-browser PDF generator requires the jsPDF library to load from cdnjs on first use.

---

## Acknowledgements

- Object data: **Messier Catalog** (Charles Messier, 1771)
- Marathon order: standard AL / community observing sequence
- PSA references: *Pocket Sky Atlas*, Roger Sinnott (Sky & Telescope)
- Sky simulation: [Stellarium Web](https://stellarium-web.org)
- Reverse geocoding: [OpenStreetMap Nominatim](https://nominatim.openstreetmap.org)
- Object photos: [SEDS Messier Catalog](http://www.messier.seds.org) (optional, downloaded separately)
- In-browser PDF: [jsPDF](https://github.com/parallax/jsPDF) + [jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable)

---

## License

MIT — free to use, share, and modify. Clear skies! 🌌
