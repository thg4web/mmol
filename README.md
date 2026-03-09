#  Messier Marathon — Observation Log

A beautifully designed, fully self-contained HTML observation log for the **Messier Marathon** — the challenge of observing all 110 Messier objects in a single night. Built for use in the field with red night-vision mode, real-time sky chart links, and persistent session tracking.

![Night Mode](https://img.shields.io/badge/Night%20Mode-Red%20Vision-cc3322?style=flat-square)
![Objects](https://img.shields.io/badge/Objects-110%20Messier-gold?style=flat-square)
![Self Contained](https://img.shields.io/badge/Self--Contained-Single%20HTML%20File-4a9a6a?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

---

## Features

- **Red night-vision mode** — deep black background with full red-spectrum text to preserve your dark adaptation
- **Day / Night toggle** — switch between a warm parchment atlas (day) and red-on-black (night) with a single click; preference is saved between sessions
- **110 Messier objects** — listed in optimal marathon search order, west to east
- **Object image lightbox** — click any M# to open a modal with the object photo, full data sidebar, and navigation
- **Live sky chart links** — each object links directly to:
  - [Stellarium Web](https://stellarium-web.org) — full 3D sky simulation centred on the object
  - [Sky-Map.org](https://www.sky-map.org) — coordinate-precise finder chart with constellation lines
- **Check off objects** — checkbox per object with strikethrough; checked state persists in `localStorage`
- **Field notes** — inline text input per object for logging time, seeing, eyepiece, comments, etc.
- **Progress bar** — live count of observed vs. total objects
- **Filter buttons** — quickly narrow to Galaxies, Open Clusters, Globulars, Nebulae, or Unobserved
- **Search** — filter by M#, object type, or constellation abbreviation
- **Object type legend** — colour-coded dot key for all 11 object types
- **Print-ready** — clean black-on-white print stylesheet, no wasted ink on dark backgrounds
- **No dependencies** — single HTML file, works entirely offline (except for sky chart links and Google Fonts)
- **Pocket Sky Atlas** - The PSA field denotes the objects chart number for fast looksups on your Pocket Sky Atals 

---

## Quick Start

1. Download `index.html`
2. Open it in any modern browser
3. Optionally add object images (see [Adding Photos](#adding-photos) below)

No server, no install, no build step required.

---

## Optional: Adding Photos

The lightbox supports local object images. Place JPEGs in an `images/` folder alongside the HTML file, named in lowercase:

```
messier_marathon.html
images/
  m1.jpg
  m2.jpg
  ...
  m110.jpg
```

If an image is missing, the lightbox shows a graceful placeholder — the log works fine with a partial or empty `images/` folder.

---

## Sky Chart Links

Each object's lightbox sidebar contains two live sky chart buttons:

| Button | Service | How it works |
|--------|---------|--------------|
| Stellarium Web | [stellarium-web.org](https://stellarium-web.org) | Deep-links by Messier name (e.g. `/skysource/M42`). Opens a real-time 3D sky simulation centred on the object. Set your location and date/time for an accurate view. |
| Sky-Map.org | [sky-map.org](https://www.sky-map.org) | Deep-links by precise RA/Dec coordinates converted from the catalog data. Opens with constellation lines, grid, and a position indicator pre-enabled. |

---

## Object Search Order

Objects are listed in **marathon search order** — the sequence that gives the best chance of observing all 110 in one night, starting with western objects visible just after dusk and ending with eastern objects visible just before dawn. The marathon is typically attempted in late March when this ordering is optimal from mid-northern latitudes.

---

## Data Fields

| Column | Description |
|--------|-------------|
| **#** | Marathon search order (1–110) |
| **Object** | Messier designation (M1–M110); click to open lightbox |
| **Type** | Object type with colour-coded dot |
| **Con** | Constellation abbreviation |
| **Mag** | Visual magnitude |
| **RA** | Right Ascension (J2000.0) |
| **Dec** | Declination (J2000.0) |
| **PSA Pg** | *Pocket Sky Atlas* page reference (Roger Sinnott) |
| **✓** | Observation checkbox |
| **Field Notes** | Free-text notes input |

---

## Object Types

| Colour | Type |
|--------|------|
| 🟣 Purple | Spiral Galaxy |
| 🔵 Blue | Elliptical Galaxy |
| 🩵 Teal | Lenticular Galaxy |
| 🟣 Violet | Irregular Galaxy |
| 🟢 Green | Open Cluster |
| 🟡 Gold | Globular Cluster |
| 🔴 Red | Diffuse Nebula |
| 🟡 Yellow | Planetary Nebula |
| 🟠 Orange | Supernova Remnant |
| 🩵 Cyan | Star Cloud |
| ⚫ Grey | Double Star / Asterism |

---

## Browser Compatibility

Tested in current versions of Chrome, Firefox, Safari, and Edge. Requires a browser with `localStorage` support for note and checkbox persistence.

---

## Acknowledgements

- Many thnaks to Don Machholz (October 7, 1952 - August 9, 2022) for first coming up with the Mesier Marathon
- Many thnaks to the other groups and individuals who provided images to Wikipedia Messier Objecs and are noted with attribution
- Object data based on the **Messier Catalog** (Charles Messier, 1771)
- Marathon search order adapted from standard community observing resources
- PSA page references: *Pocket Sky Atlas* by Roger Sinnott (Sky & Telescope) - Get this now!, It's awesome!
- Sky chart links: [Stellarium Web](https://stellarium-web.org) and [freestarcharts.com](https://freestarcharts.com)
- Object photos (non-copyright images): [SEDS Messier Catalog](http://www.messier.seds.org)

---

## License

MIT — free to use, share, and modify. Clear skies!
