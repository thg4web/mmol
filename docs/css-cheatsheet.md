# CSS Cheat Sheet — mm-shared.css

Reference for `mm-shared.css` — the design-system stylesheet shared across all pages. Every new page on this site should link this file, then add only page-specific styles in its own `<style>` block.

---

## Contents

1. [Linking the stylesheet](#1-linking-the-stylesheet)
2. [Theme system](#2-theme-system)
3. [Colour variables](#3-colour-variables)
4. [Layout classes](#4-layout-classes)
5. [Theme toggle](#5-theme-toggle)
6. [Main page header](#6-main-page-header-indexhtml)
7. [Prose page header](#7-prose-page-header-faq-etc)
8. [Footer](#8-footer)
9. [Section title](#9-section-title)
10. [Legend](#10-legend)
11. [Type badge colours](#11-type-badge-colours)
12. [Nav link](#12-nav-link)

---

## 1 · Linking the Stylesheet

Place the self-hosted font `@font-face` declarations and the stylesheet link in `<head>`, before any page-specific `<style>` block. The fonts must load before the CSS or the Cinzel headings will flash unstyled.

```html
<style>
  @font-face { font-family: "Cinzel"; src: url("fonts/Cinzel-Regular.ttf") format("truetype"); font-weight: 400; font-display: swap; }
  @font-face { font-family: "Cinzel"; src: url("fonts/Cinzel-SemiBold.ttf") format("truetype"); font-weight: 600; font-display: swap; }
  @font-face { font-family: "Cinzel"; src: url("fonts/Cinzel-Bold.ttf") format("truetype"); font-weight: 700; font-display: swap; }
  @font-face { font-family: "Share Tech Mono"; src: url("fonts/ShareTechMono-Regular.ttf") format("truetype"); font-weight: 400; font-display: swap; }
  @font-face { font-family: "Crimson Pro"; src: url("fonts/CrimsonPro-Light.ttf") format("truetype"); font-weight: 300; font-display: swap; }
  @font-face { font-family: "Crimson Pro"; src: url("fonts/CrimsonPro-Regular.ttf") format("truetype"); font-weight: 400; font-display: swap; }
  @font-face { font-family: "Crimson Pro"; src: url("fonts/CrimsonPro-LightItalic.ttf") format("truetype"); font-weight: 300; font-style: italic; font-display: swap; }
</style>
<link rel="stylesheet" href="mm-shared.css">
```

Then open a `<style>` tag for anything specific to that page. Never duplicate variables or base resets — they're already in the shared file.

> **File location:** `mm-shared.css` sits alongside `index.html`. All pages are in the same directory, so the relative path is always just `mm-shared.css`.

---

## 2 · Theme System

The two themes are driven by a `data-theme` attribute on `<html>`. Set it in the opening tag so the correct theme loads before paint:

```html
<html lang="en" data-theme="dark">
```

The toggle JavaScript reads and writes `localStorage` key `mm_theme`, then updates the attribute. Paste this script block at the bottom of every page's `<body>`:

```html
<script>
// Restore saved theme before first paint (ideally inline this in <head>)
const saved = localStorage.getItem('mm_theme');
if (saved) document.documentElement.setAttribute('data-theme', saved);

const pill  = document.getElementById('themeToggle');
const label = document.getElementById('toggleLabel'); // optional

function applyTheme(t) {
  document.documentElement.setAttribute('data-theme', t);
  localStorage.setItem('mm_theme', t);
  if (label) label.textContent = t === 'dark' ? 'Night' : 'Day';
}

pill.addEventListener('click', () => {
  const next = document.documentElement.getAttribute('data-theme') === 'dark'
    ? 'light' : 'dark';
  applyTheme(next);
});
pill.addEventListener('keydown', e => {
  if (e.key === 'Enter' || e.key === ' ') pill.click();
});
</script>
```

> The `toggleLabel` span is optional — only `index.html` uses it. On prose pages the null check prevents errors.

---

## 3 · Colour Variables

All colours are CSS custom properties. Never hardcode hex values in page-specific styles — always reference a variable so the theme swap works automatically.

| Variable | Dark | Light | Use |
|---|---|---|---|
| `--bg` | `#030305` | `#f4ece0` | Page background |
| `--bg2` | `#07070d` | `#faf4ea` | Cards, table bg, modals |
| `--bg3` | `#0c0b14` | `#efe5d2` | Table head, sidebar, code bg |
| `--bg-row-alt` | `#0a0912` | `#f6f0e2` | Alternating table rows |
| `--red-bright` | `#ff4f38` | `#b82a16` | Headings, accents, checked items |
| `--red-mid` | `#c03328` | `#921e0e` | Toggle pill, focus borders |
| `--red-dim` | `#8a2a1c` | `#781808` | Progress fill start, strikethrough |
| `--red-faint` | `#3c1210` | `#f4dbd4` | Toggle bg, filter active bg |
| `--text-head` | `#ff4f38` | `#8b1a0a` | h1, site title |
| `--text-data` | `#e87060` | `#180e04` | Body text, table data cells |
| `--text-sub` | `#c03830` | `#7a3020` | Subheadings, FAQ questions, links |
| `--text-meta` | `#904030` | `#9a4030` | Body copy (Crimson Pro), footer notes |
| `--text-dim` | `#4a1e18` | `#b87060` | Secondary labels, section titles |
| `--m-color` | `#ff5540` | `#9b2012` | M# column in table |
| `--con-color` | `#ff6048` | `#7a4010` | Constellation column in table |
| `--border` | `#251512` | `#d6c4ae` | Faintest borders (table row dividers) |
| `--border2` | `#3e1f18` | `#c0aa90` | Standard borders (cards, headers) |
| `--border3` | `#5a2820` | `#a08870` | Stronger borders (header rule, active) |
| `--header-line` | `#5a2820` | `#b08868` | Gradient rule under main header |
| `--starfield` | `rgba(255,90,60,.38)` | `rgba(80,30,10,.10)` | Star texture dots in body::before |
| `--filter-active-bg` | `#3c1210` | `#f4dbd4` | Active filter btn bg, checked checkbox |
| `--row-hover` | `rgba(200,50,30,.09)` | `rgba(155,32,18,.07)` | Table row hover background |

---

## 4 · Layout Classes

Two wrapper classes control page width and padding. Use `.wrapper` for data-heavy pages, `.wrapper-prose` for text pages.

```css
/* Wide layout – index.html (table + lightbox) */
.wrapper        { max-width: 1200px; padding: 32px 24px 60px; }

/* Narrow layout – faq.html */
.wrapper-prose  { max-width: 820px;  padding: 32px 20px 60px; }
```

Both are `position: relative; z-index: 1` to sit above the fixed starfield layer on `body::before`.

> The starfield is generated automatically on every page via `body::before` — nothing to add. On light theme it is very subtle; on dark theme it is the faint red-dot constellation effect.

---

## 5 · Theme Toggle

Two variants exist — **full size** (index.html, absolutely positioned top-right) and **inline** (prose pages, inside `.page-header`). Both use the same pill element; only the wrapper placement differs.

```html
<!-- Full-size toggle (index.html) -->
<div class="theme-toggle-wrap">
  <span class="theme-icon moon">🌙</span>
  <div class="toggle-pill" id="themeToggle" role="button" tabindex="0"></div>
  <span class="theme-icon sun">☀</span>
  <span class="toggle-label" id="toggleLabel">Night</span>  <!-- optional -->
</div>
```

```html
<!-- Inline toggle (faq.html – inside .page-header) -->
<div class="theme-toggle-wrap">
  <span class="theme-icon moon">🌙</span>
  <div class="toggle-pill" id="themeToggle" role="button" tabindex="0"></div>
  <span class="theme-icon sun">☀</span>
</div>
```

---

## 6 · Main Page Header (index.html)

The centred hero header used on the observation log. Contains the eyebrow line, `h1`, subtitle, horizontal rule, and progress bar. Only needed on the main page.

```html
<div class="header">
  <div class="header-eyebrow">Astronomical League · Messier Observing Program</div>
  <h1>Messier Marathon</h1>
  <div class="subtitle">Observer's Log</div>
  <span class="header-rule"></span>
  <!-- progress bar markup goes here -->
</div>
```

> The `h1` glow shadow is applied automatically on dark theme via `[data-theme="dark"] h1` — no extra class needed.

---

## 7 · Prose Page Header (FAQ, etc.)

Two-column flex bar used at the top of secondary pages. Left side holds the site title (link back to index) and current page subtitle. Right side holds the toggle.

```html
<header class="page-header">
  <div>
    <div class="site-title"><a href="index.html">Messier Marathon Observer's Log</a></div>
    <div class="page-subtitle">Page Title Here</div>
  </div>
  <!-- theme-toggle-wrap goes here -->
</header>
```

---

## 8 · Footer

Flex row, space-between. Left: italic note in Crimson Pro. Right: small-caps signature block. Add a `.nav-link` inside the sig block to link to related pages.

```html
<footer class="footer">
  <div class="footer-note">Italic note in Crimson Pro.</div>
  <div class="footer-sig">
    Page Name<br>Subtitle
    <br><a href="other.html" class="nav-link">Other Page →</a>
  </div>
</footer>
```

> The `.footer` class and `.footer-sig a` styles are in the shared CSS, so links inside `.footer-sig` inherit the dim/hover colour automatically — no inline styles needed.

---

## 9 · Section Title

Use `.section-title` to divide content sections on prose pages. Renders as a small-caps label with a bottom border. Give it an `id` for TOC anchor links.

```html
<div class="section-title" id="my-section">Section Name</div>
```

> The FAQ page uses the alias class `faq-section-title` — this is identical and can be consolidated to `section-title` in a future cleanup.

---

## 10 · Legend

The object-type key that appears below the observation table. Also usable on any page that needs a colour-keyed legend row.

```html
<div class="legend">
  <div class="legend-title">Object Type Key</div>
  <div class="legend-item">
    <div class="legend-dot badge-spiral"></div>Spiral Galaxy
  </div>
  <!-- repeat .legend-item for each type -->
</div>
```

---

## 11 · Type Badge Colours

Applied to any `7–8px` circle element. Used as dots in the table (`.type-badge`) and in the legend (`.legend-dot`). Not theme-sensitive — same in both modes.

| Class | Colour |
|---|---|
| `.badge-spiral` | Soft blue |
| `.badge-ellip` | Warm gold |
| `.badge-lenticular` | Muted teal |
| `.badge-irregular` | Olive |
| `.badge-open` | Cyan |
| `.badge-globular` | Amber |
| `.badge-nebula` | Magenta-pink |
| `.badge-planetary` | Aqua |
| `.badge-snr` | Red-orange |
| `.badge-starcloud` | Pale yellow |
| `.badge-double` | Light grey |

```html
<!-- In a table cell -->
<span class="type-badge badge-globular"></span>Globular Cluster

<!-- In the legend -->
<div class="legend-dot badge-globular"></div>Globular Cluster
```

---

## 12 · Nav Link

A subtle utility class for cross-page navigation links — typically used in footers or page headers. Monospace, small-caps, dotted underline.

```html
<a href="faq.html" class="nav-link">FAQ →</a>
<a href="index.html" class="nav-link">← Back to Log</a>
```
