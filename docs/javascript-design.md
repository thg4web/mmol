# Messier Marathon Observer's Log — JavaScript Design Reference

All application logic lives in a single `<script>` block near the bottom of `index.html`.
There are no modules, no imports, and no build step. Everything executes in the global
browser scope. This document walks through every section in execution order with
heavy commentary on purpose, design choices, and gotchas.

---

## Table of Contents

1. [Mobile Detection](#1-mobile-detection)
2. [Themed Confirm Dialog](#2-themed-confirm-dialog)
3. [Static Data — COMMON_NAMES and DATA](#3-static-data--common_names-and-data)
4. [Type Badge Helpers](#4-type-badge-helpers)
5. [State Variables and Persistence](#5-state-variables-and-persistence)
6. [Table Rendering — renderTable()](#6-table-rendering--rendertable)
7. [Full Observation Record Modal](#7-full-observation-record-modal)
8. [Image Lightbox](#8-image-lightbox)
9. [Progress Bar](#9-progress-bar)
10. [Filter, Search, and Sort](#10-filter-search-and-sort)
11. [CSV Helpers](#11-csv-helpers)
12. [Export — JSON Records and CSV Log](#12-export--json-records-and-csv-log)
13. [Import — AL Records JSON](#13-import--al-records-json)
14. [Session Setup and Equipment/Location Presets](#14-session-setup-and-equipmentlocation-presets)
15. [GPS Geolocation](#15-gps-geolocation)
16. [Menu System — Top Bar and Hamburger](#16-menu-system--top-bar-and-hamburger)
17. [Print — Checklist and PDF](#17-print--checklist-and-pdf)
18. [Reset All Data](#18-reset-all-data)
19. [Three-State Theme System](#19-three-state-theme-system)
20. [Save / Restore / Import / Export Modals](#20-save--restore--import--export-modals)
21. [Initialisation Sequence](#21-initialisation-sequence)
22. [Quick Start Guide](#22-quick-start-guide)
23. [PDF Report Generator](#23-pdf-report-generator)

---

## 1. Mobile Detection

```js
(function detectMobile() {
  function applyMobile() {
    if (window.innerWidth <= 768) {
      document.documentElement.classList.add("is-mobile");
    } else {
      document.documentElement.classList.remove("is-mobile");
    }
  }
  applyMobile();                              // run once immediately on page load
  window.addEventListener("resize", applyMobile);  // re-run on every resize/rotate
})();
```

**What it does:** Adds or removes the `is-mobile` class on `<html>` based purely on
viewport width. All mobile-specific CSS rules (column hiding, description cell, compact
layout) are scoped to `html.is-mobile` selectors. The JS threshold (768px) is the
authoritative breakpoint — CSS visibility rules for the top-bar and hamburger both
reference `is-mobile` so there is never a mismatch.

**Why viewport-only detection:** Earlier versions used a triple check (width AND touch
AND user-agent). This was too fragile — any one condition failing silently would leave
a phone showing the full desktop layout. Viewport width alone is the correct signal
for layout decisions.

**Why the IIFE wrapper:** The function name `detectMobile` is only for readability
in stack traces. The IIFE prevents polluting global scope.

**Resize listener:** Ensures orientation change (portrait ↔ landscape) and browser
window resize switch the layout live without a page reload.

---

## 2. Themed Confirm Dialog

```js
function showConfirm(
  { title, body, warning, okLabel, cancelLabel } = {},
  onOk   // callback fired only if user clicks OK
) {
  // Populate the static confOverlay modal with the provided strings
  titleEl.textContent  = title;
  bodyEl.textContent   = body;
  warnEl.textContent   = warning;
  warnEl.style.display = warning ? "block" : "none";  // hide warn row if empty
  okBtn.textContent    = okLabel;
  cancelBtn.textContent= cancelLabel;

  overlay.classList.add("open");  // show the modal

  // One-shot click handlers — cleaned up whether user confirms or cancels.
  // Using onclick (not addEventListener) so repeated calls don't stack handlers.
  const cleanup = () => { okBtn.onclick = null; cancelBtn.onclick = null; };
  okBtn.onclick     = () => { cleanup(); close(); onOk(); };   // confirm path
  cancelBtn.onclick = () => { cleanup(); close(); };           // cancel path — no callback
}
```

**What it does:** Replaces all four `window.confirm()` call sites with a reusable
modal that follows the current CSS theme. The browser's native confirm dialog is
always white and always breaks night-vision mode at the telescope.

**Why `onclick` instead of `addEventListener`:** If `showConfirm` is called twice in
quick succession (e.g. double-click on a button), `addEventListener` would stack two
handlers and fire the callback twice. Assigning to `onclick` replaces the previous
handler, making the function safe to call multiple times.

**The `cleanup()` call:** Both branches call `cleanup()` before their action. This
prevents a stale handler being fired if the overlay is somehow triggered again before
garbage collection.

**Warning row:** The warning parameter is used for destructive actions ("This cannot
be undone") and is hidden entirely when empty so it doesn't leave a visual gap.

---

## 3. Static Data — COMMON_NAMES and DATA

### COMMON_NAMES

```js
const COMMON_NAMES = {
  M1: "Crab Nebula",
  M31: "Andromeda Galaxy",
  // ... 110 entries, empty string "" for objects with no common name
};
```

**What it does:** Maps Messier IDs to informal names. Used in the mobile description
cell and the lightbox header. Empty strings are used (not `null`) so the lookup is
always safe — the template literal checks `if (commonName)` before rendering.

### DATA

```js
const DATA = [
  [1, "M77", "Spiral Gal", "Cet", 8.9, "7×6", "02 42.7", "-00 01", "4, 6"],
  // ... 110 rows in marathon order
];
// Index: [0]=marathonOrder, [1]=messierID, [2]=type, [3]=constellation,
//        [4]=magnitude,      [5]=size,     [6]=ra,   [7]=dec,  [8]=psaPages
```

**What it does:** The complete catalog of 110 Messier objects. The array is ordered
by Don Machholz marathon sequence (west to east), which is the default table sort.

**Key design choices:**
- The array is **never mutated**. Sort operations work on a `[...DATA]` copy.
- `messierID` (index 1) is the primary key for all localStorage operations. It always
  equals `"M" + number` with no spaces, matching the key format `M1`, `M77`, `M110`.
- `size` (index 5) is a raw string from SEDS data — could be `"7×6"` (arcminutes),
  `"110'"` (arcminutes with tick), or `"0.8'"` (for M40 double star). It is displayed
  as-is and never parsed.
- RA is stored as `"HH MM.M"` and Dec as `"+DD MM"` or `"-DD MM"` — human-readable
  J2000 coordinates matching the Pocket Sky Atlas convention.

---

## 4. Type Badge Helpers

```js
// Maps a type string to a CSS class that controls badge colour in light/dark mode
function getTypeClass(type) {
  if (type.includes("Spiral"))      return "badge-spiral";
  if (type.includes("Elliptical"))  return "badge-ellip";
  // ... etc.
  return "badge-double";   // fallback for Asterism, Double Star
}

// Maps a type string to a single ASCII symbol for display in night mode
function getTypeSymbol(type) {
  if (type.includes("Spiral"))      return "@";   // spiral arms
  if (type.includes("Elliptical"))  return "●";   // solid ellipse
  if (type.includes("Lenticular"))  return "◑";   // half-filled
  if (type.includes("Irregular"))   return "*";   // irregular
  if (type.includes("Open"))        return "+";   // cross = cluster
  if (type.includes("Globular"))    return "◎";   // concentric circles
  if (type.includes("Diffuse"))     return "~";   // nebulous
  if (type.includes("Planetary"))   return "○";   // ring
  if (type.includes("Supernova"))   return "✕";   // remnant/explosion
  if (type.includes("Star Cloud"))  return "::";  // scattered stars
  return "·";                                      // point source / other
}

// Used by applyFilter() to match rows against the type dropdown selection
function matchesFilterType(type, filter) {
  if (filter === "Galaxy")         return type.includes("Gal");
  if (filter === "Open Cluster")   return type === "Open Cluster";
  if (filter === "Globular Clust") return type.includes("Globular");
  if (filter === "Nebula")         return type.includes("Neb")
                                       || type.includes("Supernova")
                                       || type.includes("Star Cloud");
  return true;
}
```

**Why ASCII symbols:** In night mode (red-on-black), colour alone cannot differentiate
object types — everything is a shade of red. The ASCII glyph conveys the shape/class
of the object visually without relying on hue.

**Why `includes()` not `===`:** The DATA type strings sometimes have spaces
(`"Spiral Gal"`, `"Globular Clust"`) so substring matching is more robust than
exact equality. The exception is `"Open Cluster"` in `matchesFilterType` where exact
match is needed to distinguish it from `"Globular Clust"`.

---

## 5. State Variables and Persistence

```js
// The four in-memory state objects — always kept in sync with localStorage
const SAVED_NOTES     = {};  // { "M77": "faint spiral, no core visible" }
const SAVED_CHECKS    = {};  // { "M77": true }
const SAVED_TIMES     = {};  // { "M77": "2026-03-22T02:15:00.000Z" }
const SAVED_FULLNOTES = {};  // { "M77": { date, time, site, seeing, … } }

function loadState() {
  // Reads all four keys from localStorage and merges into the in-memory objects.
  // try/catch guards against JSON parse errors and private-mode storage exceptions.
  try {
    const c = localStorage.getItem("mm_checks");
    if (c) Object.assign(SAVED_CHECKS, JSON.parse(c));
    // … same pattern for notes, times, fullnotes
  } catch(e) {}
}

function saveState() {
  // Writes all four objects back to localStorage as JSON strings.
  // Called on every mutation: checkbox change, note input, record save.
  try {
    localStorage.setItem("mm_checks",    JSON.stringify(SAVED_CHECKS));
    // … same for notes, times, fullnotes
  } catch(e) {}
}
```

**Why four separate keys:** Each key can be independently read and written, which
matters for the backup/restore system. A full backup serialises all four. An AL
records export only touches `mm_fullnotes` and `mm_times`. Splitting them avoids
loading the entire dataset just to update one checkbox.

**Why `Object.assign` on load:** The state objects are declared as `const {}` at
module scope. `Object.assign` populates them in place rather than reassigning the
variable, keeping any references that were captured before `loadState()` valid.

**Why try/catch everywhere:** `localStorage` is unavailable in some private/incognito
modes, and `JSON.parse` can throw if the data is corrupted. Silent failure is
preferable to a JS exception that would prevent the app loading.

---

## 6. Table Rendering — renderTable()

```js
function renderTable() {
  const tbody = document.getElementById("tableBody");
  tbody.innerHTML = "";   // wipe current rows

  // Sort a shallow copy for numeric (M1–M110) mode;
  // marathon mode uses the native DATA array order (already correct)
  const rows = sortMode === "numeric"
    ? [...DATA].sort((a, b) =>
        parseInt(a[1].replace("M","")) - parseInt(b[1].replace("M","")))
    : DATA;

  rows.forEach(([mo, m, type, con, mag, size, ra, dec, psa]) => {
    // Critical: find the object's TRUE index in the original DATA array.
    // This is stored as data-idx on clickable elements so openLightbox()
    // always receives the right DATA row regardless of current sort mode.
    const dataIdx = DATA.findIndex(d => d[1] === m);
    const id = m.replace(/\s/g, "");  // "M77" → "M77" (spaces removed for safety)

    const tr = document.createElement("tr");
    tr.className = "data-row";
    // Store lookup values as data-attributes so applyFilter() can read them
    // without re-parsing the cell text content
    Object.assign(tr.dataset, { mo, m, type, con, id });
    if (SAVED_CHECKS[id]) tr.classList.add("checked");  // pre-apply strikethrough

    // Build all cells as a template literal — one innerHTML assignment per row
    // is faster than 13 individual DOM insertions
    tr.innerHTML = `
      <td class="col-thn-val">
        <img class="row-thn" src="images/${id.toLowerCase()}_thn.jpg"
             loading="lazy" data-idx="${dataIdx}">
      </td>
      <td class="col-mobile-obj">  <!-- hidden on desktop, shown on mobile -->
        <div class="mob-m">${m}</div>
        ${commonName ? `<div class="mob-name">${commonName}</div>` : ""}
        <div class="mob-meta">${type} · ${con}</div>
      </td>
      <td class="col-mo-val">${mo}</td>
      <td class="col-m-val"><a class="m-link" data-idx="${dataIdx}">${m}</a></td>
      <!-- … all other cells … -->
      <td class="col-note-val">
        <div class="note-cell-wrap">
          <input class="note-input" value="${SAVED_NOTES[id]||""}">
          <!-- rec-btn shows outlined (no record) or solid red (has record) -->
          <button class="rec-btn${SAVED_FULLNOTES[id] ? " has-rec" : ""}"
                  data-id="${id}">✎</button>
        </div>
      </td>
    `;

    // Wire event listeners directly to this row's elements after innerHTML is set
    tr.querySelector(".row-thn").addEventListener("click", function() {
      openLightbox(parseInt(this.dataset.idx));
    });
    tr.querySelector(".m-link").addEventListener("click", function(e) {
      e.preventDefault();
      openLightbox(parseInt(this.dataset.idx));
    });
    tr.querySelector("input[type=checkbox]").addEventListener("change", function() {
      SAVED_CHECKS[id] = this.checked;
      if (this.checked) {
        SAVED_TIMES[id] = new Date().toISOString();  // timestamp the observation
      } else {
        delete SAVED_TIMES[id];  // un-checking removes the timestamp
      }
      tr.classList.toggle("checked", this.checked);
      saveState(); updateProgress(); applyFilter();
    });
    tr.querySelector(".note-input").addEventListener("input", function() {
      SAVED_NOTES[id] = this.value; saveState();
    });
    tr.querySelector(".rec-btn").addEventListener("click", function() {
      openRecModal(id, m, type);
    });

    tbody.appendChild(tr);
  });
}
```

**Why `DATA.findIndex()` for `dataIdx`:** When in numeric sort mode the loop variable
index no longer matches the DATA array position. Storing the real DATA index on the
element ensures `openLightbox(idx)` loads the correct object. This is the most
important correctness guarantee in the rendering code.

**Why `tbody.innerHTML = ""` then `appendChild`:** Clearing innerHTML first and then
appending rows (rather than building one giant string) allows per-row event listener
attachment. A single innerHTML string for the entire tbody would require event
delegation — the current approach is simpler and easier to follow.

**`data-idx` on both thumbnail and M# link:** Both elements open the lightbox; both
need the index. It is duplicated intentionally.

**`col-mobile-obj` cell:** Always rendered in every row, always in the DOM. CSS
`display:none` / `display:table-cell` controlled by `html.is-mobile` shows or hides
it. This avoids a re-render on orientation change.

---

## 7. Full Observation Record Modal

### Opening — openRecModal(id, m, type)

```js
let recCurrentId = null;  // tracks which object's record is open

function openRecModal(id, m, type) {
  recCurrentId = id;
  const saved = SAVED_FULLNOTES[id] || {};  // empty object if no record yet
  const sess  = SESSION;

  // Date/time seeding priority:
  // 1. Previously saved value (user edited it)
  // 2. The ISO timestamp from when the checkbox was ticked (SAVED_TIMES[id])
  // 3. Right now
  const autoTs   = SAVED_TIMES[id] ? new Date(SAVED_TIMES[id]) : new Date();
  const autoDate = autoTs.toLocaleDateString("en-CA");  // YYYY-MM-DD format
  const autoTime = autoTs.toTimeString().slice(0, 5);   // HH:MM

  document.getElementById("recDate").value = saved.date || autoDate;
  document.getElementById("recTime").value = saved.time || autoTime;

  // For all other fields: saved value → session default → empty
  document.getElementById("recSite").value    = saved.site    || sess.site    || "";
  document.getElementById("recScope").value   = saved.scope   || sess.scope   || "";
  // … same pattern for all remaining fields …

  // Description ↔ Field Notes two-way sync (on open):
  // If the inline Field Notes input has text, it is ALWAYS the source of truth —
  // seed Description from it. This catches edits made between sessions.
  // Only fall back to the previously saved description if Field Notes is blank.
  const fieldNote = (SAVED_NOTES[id] || "").trim();
  document.getElementById("recDescription").value = fieldNote || saved.description || "";

  document.getElementById("recOverlay").classList.add("open");
  document.body.style.overflow = "hidden";  // prevent background scroll
}
```

### Saving — saveRecModal()

```js
function saveRecModal() {
  if (!recCurrentId) return;
  const _prev = SAVED_FULLNOTES[recCurrentId] || {};  // preserve existing lat/lon/tz

  SAVED_FULLNOTES[recCurrentId] = {
    date: ..., time: ..., site: ..., seeing: ..., transparency: ...,
    limitMag: ..., scope: ..., aperture: ..., eyepiece: ..., mag: ...,
    description: ..., sketch: ...,
    // Location snapshot: prefer live SESSION values so the record captures
    // where the observer was at save time. Fall back to whatever was previously
    // saved so data is never silently erased on a re-save after clearing session.
    lat: SESSION.lat != null ? SESSION.lat : (_prev.lat != null ? _prev.lat : null),
    lon: SESSION.lon != null ? SESSION.lon : (_prev.lon != null ? _prev.lon : null),
    tz:  SESSION.tz  ? SESSION.tz  : (_prev.tz  || null),
  };

  // If every field is empty, delete the record entirely rather than storing a blank shell
  const hasData = Object.values(SAVED_FULLNOTES[recCurrentId]).some(v => v && v.trim && v.trim());
  if (!hasData) delete SAVED_FULLNOTES[recCurrentId];

  // Description → Field Notes sync (on save):
  // Write Description back to the inline note input so both stay in sync.
  // Also update the visible DOM input in the table row without a full re-render.
  const descNow = document.getElementById("recDescription").value.trim();
  if (descNow) {
    SAVED_NOTES[recCurrentId] = descNow;
    const recBtnEl = document.querySelector(`.rec-btn[data-id="${recCurrentId}"]`);
    if (recBtnEl) {
      const noteEl = recBtnEl.closest(".note-cell-wrap").querySelector(".note-input");
      if (noteEl) noteEl.value = descNow;
    }
  }

  saveState();

  // Update the ✎ button style (outlined vs solid) without re-rendering the row
  const btn = document.querySelector(`.rec-btn[data-id="${recCurrentId}"]`);
  if (btn) btn.classList.toggle("has-rec", !!SAVED_FULLNOTES[recCurrentId]);

  closeRecModal();
}
```

**The two-way sync:** Field Notes (inline table input) and Description (modal textarea)
represent the same data but at different levels of detail. The sync rules ensure:
- Typing in Field Notes, then opening the modal: modal pre-fills from Field Notes.
- Typing in Description, then saving: Field Notes updates to match.
- They can never silently diverge across sessions.

**Location snapshot:** The lat/lon/tz are snapshotted at save time into the record
itself. This is intentional — if the user changes their session location between
sessions, previously saved records still carry the correct location for the PDF.

**No re-render on save:** Updating the rec-btn class and note-input value directly
on the existing DOM nodes is faster and avoids re-wiring all event listeners. The
`data-id` attribute on the rec-btn is the lookup key.

---

## 8. Image Lightbox

```js
let lbCurrentIdx = 0;  // tracks which DATA index is currently displayed

function openLightbox(idx) {
  lbCurrentIdx = idx;
  populateLightbox(idx);
  document.getElementById("lbOverlay").classList.add("open");
  document.body.style.overflow = "hidden";
  document.getElementById("lbClose").focus();  // accessibility: focus trap
}

function populateLightbox(idx) {
  const [mo, m, type, con, mag, size, ra, dec, psa] = DATA[idx];

  // Fill all text fields from DATA
  document.getElementById("lbTitleM").textContent    = m;
  document.getElementById("lbSideSize").textContent  = size;
  // … etc for all 10+ display fields …

  // Build URL for Stellarium Web — supports direct Messier object lookup by name
  document.getElementById("lbStellarium").href = `https://stellarium-web.org/skysource/${m}`;

  // Local finder chart
  document.getElementById("lbLocalChart").href = `charts/${m}.png`;

  // Prev/next button state
  document.getElementById("lbPrev").disabled = idx === 0;
  document.getElementById("lbNext").disabled = idx === DATA.length - 1;

  // Image loading with fallback and optional attribution overlay
  const wrap = document.getElementById("lbImgWrap");
  wrap.innerHTML = "";        // clear previous image
  const img = new Image();   // create element before setting src (allows onload binding)

  img.onload = () => {
    wrap.appendChild(img);
    // Try to fetch a sidecar attribution text file (e.g. images/m77-attrib.txt).
    // Silently ignored if the file doesn't exist — not all images have attribution.
    fetch(`images/${m.toLowerCase()}-attrib.txt`)
      .then(r => r.ok ? r.text() : null)
      .then(text => {
        if (text && text.trim()) {
          const div = document.createElement("div");
          div.className = "lb-attribution";
          div.textContent = text.trim();
          wrap.appendChild(div);
        }
      })
      .catch(() => {});   // network failure = no attribution shown, not an error
  };
  img.onerror = () => {
    // Show a styled placeholder if the image file doesn't exist yet
    wrap.innerHTML = `<div class="lb-img-placeholder">…</div>`;
  };
  img.src = `images/${m.toLowerCase()}.jpg`;
}
```

**Keyboard navigation:** A `document.keydown` listener checks for `ArrowLeft`,
`ArrowRight`, and `Escape` when the lightbox is open, calling `populateLightbox`
with `--lbCurrentIdx` or `++lbCurrentIdx` respectively.

**Image loading pattern:** The `new Image()` element is created before `src` is
assigned so the `onload`/`onerror` handlers are already attached when the browser
starts the fetch. This prevents a race condition where a very fast cache hit could
fire `onload` before the handler was attached.

**Attribution sidecar files:** Each image can optionally have a companion `.txt` file.
`fetch()` is used rather than a `<script>` or data attribute so the attribution is
loaded lazily only when the lightbox is opened.

---

## 9. Progress Bar

```js
function updateProgress() {
  const done = Object.values(SAVED_CHECKS).filter(Boolean).length;
  // "0 / 110" counter
  document.getElementById("progressCount").textContent = `${done} / ${DATA.length}`;
  // Width of the fill bar as a percentage
  document.getElementById("progressFill").style.width = `${(done / DATA.length) * 100}%`;
}
```

Called whenever a checkbox changes, on reset, and on restore/import. The CSS
transition on `progressFill.width` provides the animated bar movement for free.

---

## 10. Filter, Search, and Sort

### State variables

```js
let activeFilter = "all";   // "all" | "unchecked" | "Galaxy" | "Open Cluster" | …
let searchTerm   = "";      // lowercase string from the search input
let sortMode     = "marathon";  // "marathon" | "numeric"
```

### applyFilter()

```js
function applyFilter() {
  document.querySelectorAll("tr.data-row").forEach(tr => {
    const { type, id, m, con, mo } = tr.dataset;  // pre-stored at render time
    const filterOk = activeFilter === "unchecked"
      ? !SAVED_CHECKS[id]                          // show only unobserved
      : matchesFilterType(type, activeFilter);      // type dropdown match
    const searchOk = !searchTerm
      || [m, type, con, mo].some(v => v.toLowerCase().includes(searchTerm));
    // toggle CSS class; CSS display:none is faster than style.display manipulation
    tr.classList.toggle("hidden-row", !filterOk || !searchOk);
  });
}
```

**Why toggle a class instead of `style.display`:** The `hidden-row` class is defined
in CSS as `display: none !important`. Using a class means the CSS can override it
(e.g. the print stylesheet overrides it to show all rows). Direct `style.display`
manipulation would override the stylesheet.

**Why `tr.dataset` instead of reading cell text:** All filter-relevant values are
stored as `data-*` attributes at render time. Reading `.dataset` is O(1) and avoids
parsing formatted cell HTML.

**Search fields:** M# (e.g. "M77"), type, constellation abbreviation, and marathon
order number are all searched simultaneously. Searching "77" matches M77. Searching
"cet" matches all Cetus objects.

### Sort

```js
function updateObjSortHeader() {
  // Updates both desktop column header and mobile sort button label
  const label = sortMode === "numeric" ? "Numeric" : "Marathon";
  document.getElementById("objSortTh").innerHTML = label + ' <span class="sort-arrow">▲</span>';
  document.getElementById("mobSortBtn").textContent = label + " ▲";
}

// Both desktop header click and mobile sort button do the same thing:
document.getElementById("objSortTh").addEventListener("click", function() {
  sortMode = sortMode === "marathon" ? "numeric" : "marathon";
  updateObjSortHeader();
  renderTable();   // full re-render with new sort order
  applyFilter();   // re-apply current filter/search to the new rows
});
```

**Why re-render instead of sort the DOM:** Sorting `<tr>` elements in the DOM is
possible but complex to implement correctly with all the event listeners attached.
A full `renderTable()` call is simpler, fast enough for 110 rows, and guaranteed
to be correct.

---

## 11. CSV Helpers

```js
// Wrap a value in quotes, escaping any internal quote characters as ""
// RFC 4180 compliant
function csvField(val) {
  const s = String(val === null || val === undefined ? "" : val);
  return `"${s.replace(/"/g, '""')}"`;
}

// Parse one CSV line, handling quoted fields that may contain commas
function parseCSVLine(line) {
  const fields = [];
  let cur = "", inQ = false;
  for (let i = 0; i < line.length; i++) {
    const ch = line[i];
    if (inQ) {
      if (ch === '"' && line[i+1] === '"') { cur += '"'; i++; }  // escaped ""
      else if (ch === '"')                 { inQ = false; }       // closing quote
      else                                 { cur += ch; }
    } else {
      if      (ch === '"') { inQ = true; }
      else if (ch === ',') { fields.push(cur); cur = ""; }
      else                 { cur += ch; }
    }
  }
  fields.push(cur);  // last field has no trailing comma
  return fields;
}
```

Used by both the CSV log export and the equipment/locations CSV import. A hand-written
parser is used rather than a library because this is a zero-dependency application.

---

## 12. Export — JSON Records and CSV Log

### JSON Records Export

```js
document.getElementById("menuExportRecords").addEventListener("click", function() {
  // Include observation timestamps (mm_times) in the export so the PDF generator
  // can access the exact observation moment even if date wasn't filled manually
  const payload = {
    exported:   now.toISOString(),
    version:    1,
    session:    sessionMeta,          // current GPS/site info
    records:    SAVED_FULLNOTES,      // all AL records
    timestamps: SAVED_TIMES           // checkbox timestamps
  };
  // Create a temporary <a> element, trigger download, then remove it
  const blob = new Blob([JSON.stringify(payload, null, 2)], { type: "application/json" });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement("a");
  a.href = url; a.download = `${stamp}-al-records.json`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);  // release memory
});
```

### CSV Log Export

```js
document.getElementById("menuGenerateCSV").addEventListener("click", function() {
  // One row per Messier object — all 110 rows always present
  DATA.forEach(([mo, m, type, con, mag, ra, dec, psa]) => {
    const observed  = SAVED_CHECKS[m] ? "Yes" : "No";
    const timestamp = SAVED_TIMES[m] ? new Date(SAVED_TIMES[m]).toLocaleString() : "";
    // Location: prefer the snapshot saved with the full record,
    // fall back to current session coordinates
    const rec = SAVED_FULLNOTES[m] || {};
    const lat = rec.lat != null ? rec.lat : (SESSION.lat != null ? SESSION.lat : "");
    rows.push([mo, m, type, con, mag, ra, dec, psa, observed, timestamp,
               lat, lon, tz, notes].map(csvField).join(","));
  });
});
```

**Why 110 rows always:** A sparse export (only observed objects) would be useful for
analysis but confusing for AL submission. A complete log with "Yes"/"No" observed
status is unambiguous.

**The `URL.createObjectURL` + `<a>` pattern:** This is the standard browser technique
for triggering a file download from in-memory data. `URL.revokeObjectURL` releases
the memory allocated by `createObjectURL`. The `<a>` element is appended, clicked,
and immediately removed.

---

## 13. Import — AL Records JSON

```js
document.getElementById("btnImportALFile").addEventListener("change", function(e) {
  const reader = new FileReader();
  reader.onload = function(evt) {
    let payload = JSON.parse(evt.target.result);
    const incoming = payload.records || payload;  // support both wrapped and bare object

    // Count conflicts (keys that exist in both incoming and current data)
    const conflicts = incomingKeys.filter(k => existingKeys.includes(k));

    // Use the themed confirm dialog — not window.confirm — to show a summary
    showConfirm({ title: "Import Records", body: msg, … }, () => {
      Object.assign(SAVED_FULLNOTES, incoming);  // incoming overwrites conflicts
      if (payload.timestamps) Object.assign(SAVED_TIMES, payload.timestamps);
      saveState();
      // Refresh all rec-btn indicators without re-rendering the entire table
      document.querySelectorAll(".rec-btn").forEach(btn => {
        btn.classList.toggle("has-rec", !!SAVED_FULLNOTES[btn.dataset.id]);
      });
    });
    e.target.value = "";  // reset file input so same file can be imported again
  };
  reader.readAsText(file);
});
```

**Why `Object.assign` for merge:** Incoming records overwrite conflicts by design —
the import file is assumed to be authoritative. The conflict count is shown in the
confirm dialog so the user knows before confirming.

**Resetting `e.target.value`:** Without this, selecting the same file a second time
fires no `change` event (browser de-duplication). Clearing the value after each import
ensures the input is always ready.

---

## 14. Session Setup and Equipment/Location Presets

### Session state

```js
const SESSION_FIELDS = ["site","seeing","transparency","limitMag","scope","aperture","eyepiece","mag"];
let SESSION = {};

function sessionIsActive() {
  // True if any session field has a non-empty trimmed value
  return SESSION_FIELDS.some(k => SESSION[k] && String(SESSION[k]).trim());
}

function updateSessionIndicator() {
  // The session indicator dot pulses when session defaults are set
  document.getElementById("sessionIndicator").classList.toggle("active", sessionIsActive());
}
```

### Preset selects

```js
function populatePresetSelects() {
  // Rebuilds the three dropdown selects in the Session Setup modal
  // from the LOCATIONS and EQUIPMENT arrays loaded from localStorage.
  // The parent element is hidden entirely if no presets exist,
  // so users with no presets imported see a clean form.
  locSel.parentElement.style.display = LOCATIONS.length ? "" : "none";
}
```

### Equipment and locations CSV import

Both use the same pattern:
1. `FileReader.readAsText()` reads the file
2. `parseCSVLine()` splits each row
3. Column positions are found by name from the header row (case-insensitive, special-chars stripped)
4. Results are stored in `EQUIPMENT` / `LOCATIONS` arrays and persisted to `mm_equipment` / `mm_locations`

**Why CSV for presets:** The user is expected to prepare their equipment list and
observing sites in a spreadsheet. CSV is the universal spreadsheet export format.
A template download is provided (`menuEqTemplate`, `menuLocTemplate`) so users have
a correctly-headed file to fill in.

---

## 15. GPS Geolocation

```js
document.getElementById("sessGeoBtn").addEventListener("click", function() {
  navigator.geolocation.getCurrentPosition(
    function(pos) {
      const lat = pos.coords.latitude;
      const lon = pos.coords.longitude;
      SESSION.lat = lat;
      SESSION.lon = lon;

      // Reverse-geocode to get a human-readable place name for the Site field
      fetch(`https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lon}&format=json`)
        .then(r => r.json())
        .then(data => {
          // Build "City, State, COUNTRY" from address components
          const parts = [addr.city||addr.town||addr.village, addr.state, addr.country_code?.toUpperCase()];
          const placeName = parts.filter(Boolean).join(", ");
          // Only auto-fill Site if the user hasn't typed anything yet
          if (placeName && !siteEl.value) siteEl.value = placeName;
        })
        .catch(() => { /* no network — coordinates saved, place name skipped */ });
    },
    function(err) {
      // Specific error messages for the three Geolocation API error codes:
      // 1 = Permission denied, 2 = Position unavailable, 3 = Timeout
      statEl.textContent = msgs[err.code] || "Could not get location.";
    },
    { timeout: 12000, maximumAge: 60000 }  // 12s timeout, accept 60s old fix
  );
});
```

**Why `maximumAge: 60000`:** Allows the browser to return a cached GPS fix up to
60 seconds old rather than starting a new acquisition. At a dark-sky site the user
likely ran GPS recently; this returns instantly instead of waiting for a new fix.

**Why only fill Site if blank:** The user may have already typed their preferred site
name. GPS auto-fill should never silently overwrite intentional input.

**Nominatim as reverse-geocoder:** Free, no API key required, works offline once
the IP is known. Failure (e.g. no cell coverage at an observatory) is handled
gracefully — coordinates are saved even if the place name lookup fails.

---

## 16. Menu System — Top Bar and Hamburger

```js
// Collect all top-bar dropdown panels and their trigger buttons
const tbDropdowns = Array.from(document.querySelectorAll(".top-bar-dropdown"));
const tbBtns      = Array.from(document.querySelectorAll(".top-bar-item"));

function closeAllMenus() {
  tbDropdowns.forEach(d => d.classList.remove("open"));
  tbBtns.forEach(b => b.classList.remove("open"));
  // Also close hamburger if open
  document.getElementById("navMenuDropdown")?.classList.remove("open");
  document.getElementById("navMenuBtn")?.classList.remove("open");
}

// Auto-wire top-bar buttons by ID convention: tbDataBtn → tbDataDrop
tbBtns.forEach(function(btn) {
  const dropId = btn.id.replace("Btn", "Drop");  // e.g. "tbDataBtn" → "tbDataDrop"
  const drop   = document.getElementById(dropId);
  if (!drop) return;
  btn.addEventListener("click", function(e) {
    e.stopPropagation();  // prevent document click from immediately closing it
    const isOpen = drop.classList.toggle("open");
    btn.classList.toggle("open", isOpen);
    // Close all other dropdowns (one-open-at-a-time)
    tbDropdowns.forEach(d => { if (d !== drop) d.classList.remove("open"); });
    tbBtns.forEach(b      => { if (b !== btn)  b.classList.remove("open"); });
  });
  drop.addEventListener("click", e => e.stopPropagation());  // keep open on item click
});

// Hamburger
navMenuBtn.addEventListener("click", function(e) {
  e.stopPropagation();
  const willOpen = !navMenuDropdown.classList.contains("open");
  closeAllMenus();  // close any open top-bar dropdowns first
  if (willOpen) {
    navMenuDropdown.classList.add("open");
    navMenuBtn.classList.add("open");
  }
});

// Global click closes everything
document.addEventListener("click", closeAllMenus);
```

**All top-bar items wire to the same action functions as hamburger items:**
```js
document.getElementById("tbSave").addEventListener("click",  () => { closeAllMenus(); openSaveModal(); });
document.getElementById("hamSave").addEventListener("click", () => { closeAllMenus(); openSaveModal(); });
// … same pattern for every action …
```

**Why the ID convention (`tbDataBtn` → `tbDataDrop`):** Avoids a lookup table and
lets new menu items be added with zero changes to the wiring code — as long as the
HTML IDs follow the convention, the `forEach` loop handles them automatically.

**`e.stopPropagation()` on the dropdown panel:** Without this, clicking any item
inside the dropdown would bubble up to the document listener and immediately close
the menu before the item's own click handler fires.

---

## 17. Print — Checklist and PDF

### Print modal

The print modal asks the user to choose between:
- **Checklist** — a simple printable HTML page with all 110 objects
- **AL Report** — the full PDF (delegates to `menuGenerateReport.click()`)

### printChecklist()

```js
function printChecklist() {
  // Read the current live table rows (respects current filter/sort)
  const rows = [];
  document.querySelectorAll("#mainTable tbody tr").forEach(tr => {
    if (tr.style.display === "none") return;  // skip filtered-out rows
    rows.push({ m, type, con, mag, checked, note });
  });

  // Build a complete self-contained HTML document as a string
  const html = `<!DOCTYPE html><html>…full stylesheet…${tableRows}…</html>`;

  // Open a new browser window and write the HTML into it
  const win = window.open("", "_blank", "width=900,height=700");
  win.document.write(html);
  win.document.close();
  win.onload = () => { win.focus(); win.print(); };  // trigger print dialog
}
```

**Why a new window:** `window.print()` prints the entire current page. A new window
allows printing only the checklist content with its own compact stylesheet, without
the app chrome (toolbar, header, legend, etc.).

**Why `window.onload` before `print()`:** `win.document.write()` is synchronous but
the browser may not have parsed the written content yet. Waiting for `onload` ensures
fonts and layout are ready before the print dialog opens.

---

## 18. Reset All Data

```js
document.getElementById("menuReset").addEventListener("click", function() {
  // Show a summary of what will be lost before confirming
  const done  = Object.values(SAVED_CHECKS).filter(Boolean).length;
  const noted = Object.values(SAVED_NOTES).filter(v => v && v.trim()).length;

  showConfirm({ title: "Reset Observation Data", warning: "This cannot be undone.", … }, () => {
    // Clear in-memory objects by deleting all keys (preserves const references)
    Object.keys(SAVED_CHECKS).forEach(k    => delete SAVED_CHECKS[k]);
    Object.keys(SAVED_NOTES).forEach(k     => delete SAVED_NOTES[k]);
    Object.keys(SAVED_TIMES).forEach(k     => delete SAVED_TIMES[k]);
    Object.keys(SAVED_FULLNOTES).forEach(k => delete SAVED_FULLNOTES[k]);
    // Also clear localStorage directly (saveState would write empty objects,
    // but explicit removeItem ensures no stale data if JSON serialisation fails)
    localStorage.removeItem("mm_checks");
    localStorage.removeItem("mm_notes");
    localStorage.removeItem("mm_times");
    localStorage.removeItem("mm_fullnotes");
    renderTable(); updateProgress(); applyFilter();
  });
});
```

**Why delete keys instead of `= {}`:** The state objects are `const`. Reassigning
them (`SAVED_CHECKS = {}`) would throw. Deleting all keys empties them in place.

**Why both object-clear and localStorage.removeItem:** `saveState()` writes all four
objects as JSON. But if the JSON serialisation itself threw (extremely rare), the
localStorage keys would retain their old values. Explicit `removeItem` guarantees
the keys are gone even if `saveState` somehow fails.

---

## 19. Three-State Theme System

```js
let _mode  = "night";   // "night" | "day"    — controlled by Night/Day slider
let _theme = "light";   // "light" | "dark"   — controlled by the theme toggle in Settings

function _applyTheme() {
  // Night mode always wins — ignores _theme entirely
  const applied = _mode === "night" ? "night" : _theme;
  html.setAttribute("data-theme", applied);

  // Update both toggles' labels (desktop and mobile)
  labelEl.textContent       = _mode === "night" ? "Night" : "Day";
  labelElMobile.textContent = _mode === "night" ? "Night" : "Day";

  // Move pill position: day/light or day/dark both show pill on right
  html.classList.toggle("mode-day", _mode === "day");

  // Update theme button label in both hamburger and top-bar Settings
  // to show what clicking it will SWITCH TO (toggle behaviour)
  const labelText = _theme === "light" ? "Dark Theme" : "Light Theme";
  document.getElementById("navMenuThemeLabel").textContent = labelText;
  document.getElementById("tbThemeLabel").textContent      = labelText;

  // Disable the theme button in night mode — it has no effect there
  const inNight = (_mode === "night");
  navThemeBtn.disabled = inNight;
  navThemeBtn.style.opacity = inNight ? ".38" : "";
  tbThemeBtn.disabled  = inNight;
  tbThemeBtn.style.opacity  = inNight ? ".38" : "";
}

function setMode(m)  { _mode = m;  localStorage.setItem("mm_mode",  m); _applyTheme(); }
function setTheme(t) { /* validates t, sets _mode or _theme, saves both */ _applyTheme(); }

// Night/Day slider — both desktop and mobile pills call the same function
toggleEl.addEventListener("click",       () => setMode(_mode === "night" ? "day" : "night"));
toggleElMobile.addEventListener("click", () => setMode(_mode === "night" ? "day" : "night"));

// Light/Dark theme — only active in Day mode
function _toggleTheme() {
  if (_mode === "night") return;  // guard: can't change theme in night mode
  _theme = _theme === "light" ? "dark" : "light";
  localStorage.setItem("mm_theme", _theme);
  _applyTheme();
}
document.getElementById("navMenuTheme").addEventListener("click", _toggleTheme);
document.getElementById("tbTheme").addEventListener("click",      _toggleTheme);
```

**Why two variables:** Night mode (`data-theme="night"`) is a completely different
environment from Day/Dark (`data-theme="dark"`). Storing them separately means the
user's day theme preference is preserved when they switch to night and back.

**Why `_applyTheme` is the single write point:** Every theme change goes through
`_applyTheme()`. This guarantees the DOM attribute, CSS class, labels, and button
states are always consistent. There is no place where only some of these are updated.

---

## 20. Save / Restore / Import / Export Modals

Each of the four Data Sync modals follows the same open/close pattern:
```js
function openSaveModal()  { document.getElementById("saveOverlay").classList.add("open"); }
function closeSaveModal() { document.getElementById("saveOverlay").classList.remove("open"); }
document.getElementById("saveClose").addEventListener("click", closeSaveModal);
document.getElementById("saveOverlay").addEventListener("click",
  e => { if (e.target === this) closeSaveModal(); });  // click-outside-to-close
```

### Save (full backup v2)

Serialises all localStorage keys into one JSON object:
```js
const payload = {
  app: "mmol", version: 2, exported: now.toISOString(),
  data: { mm_checks, mm_notes, mm_times, mm_fullnotes,
          mm_equipment, mm_locations, mm_session, mm_mode, mm_theme }
};
```

### Restore

Accepts both v2 (full backup) and legacy v1 (records-only) formats. Detects format
by checking `payload.app === "mmol" && payload.data`. Restores all data, equipment,
locations, session, and theme in one operation. Calls `renderTable()` and
`updateSessionIndicator()` after restore to reflect the new state immediately.

### Import

A chooser modal lets the user select which types to import (Records, Equipment,
Locations). Each checked type triggers its own hidden file input's `.click()`.
Importing is additive/merge — existing data is only overwritten where keys conflict.

### Export

A chooser modal similarly lets the user select which types to export. Each checked
type fires the corresponding export handler.

---

## 21. Initialisation Sequence

```js
// These run at script evaluation time, in order:

loadState();              // populate SAVED_CHECKS, SAVED_NOTES, SAVED_TIMES, SAVED_FULLNOTES
loadSession();            // populate SESSION from mm_session

// Restore theme — with legacy migration:
// Old versions stored mm_theme = "dark" to mean night mode.
// New versions use mm_mode = "night" | "day" + mm_theme = "light" | "dark".
const savedMode  = localStorage.getItem("mm_mode");
const savedTheme = localStorage.getItem("mm_theme");
if (savedMode === "day" || savedMode === "night") _mode = savedMode;
else if (savedTheme === "dark")  _mode = "night";   // legacy migration
else if (savedTheme === "light") _mode = "day";
if (savedTheme === "dark" && savedMode === "day") _theme = "dark";

_applyTheme();            // apply theme before rendering so no flash of wrong theme
renderTable();            // build all 110 rows
updateProgress();         // set progress bar and count
updateSessionIndicator(); // show/hide pulsing session dot
```

**Ordering matters:** `_applyTheme()` must come before `renderTable()` so the correct
CSS variables are in effect when the table paints. `loadState()` must come before
`renderTable()` so checkbox states and note values are available when rows are built.

**Legacy migration:** Users who saved `mm_theme = "dark"` with an old version of the
app get migrated to the new `mm_mode = "night"` system without losing their preference.

---

## 22. Quick Start Guide

```js
(function() {
  const VISITED_KEY = "mm_visited";

  function isFirstVisit() {
    // First visit = no visited flag AND no observation data stored
    try {
      if (localStorage.getItem(VISITED_KEY)) return false;
      const dataKeys = ["mm_checks","mm_notes","mm_times","mm_fullnotes"];
      return dataKeys.every(k => !localStorage.getItem(k));
    } catch(e) { return false; }
  }

  function openQS() {
    // Force light/parchment theme for the guide regardless of current setting —
    // it's a welcome screen and must be readable
    html.setAttribute("data-theme", "light");
    html.classList.add("mode-day");
    overlay.classList.add("open");
  }

  function closeQS() {
    if (dontShow.checked) {
      localStorage.setItem(VISITED_KEY, "1");  // suppress on future visits
    }
    overlay.classList.remove("open");
    _applyTheme();  // restore the user's actual theme
  }

  if (isFirstVisit()) openQS();
})();
```

**Why force light mode for the guide:** The Quick Start guide is a first-time welcome
screen. Night mode (red-on-black) would be jarring and hard to read on a screen that
was just a normal white browser. Light mode is used unconditionally, with the user's
actual theme restored on dismiss.

**Why check data keys as well as visited flag:** A user who clears site data loses the
`mm_visited` flag but also loses all their observations. The check `dataKeys.every(k =>
!localStorage.getItem(k))` prevents the guide from popping up when the user still has
observation data — e.g. after a partial storage clear.

**IIFE wrapper:** The Quick Start code captures `VISITED_KEY`, `overlay`, etc. in
closure scope to avoid polluting the global namespace.

---

## 23. PDF Report Generator

The entire PDF module is wrapped in an IIFE to scope its constants and helpers.

### Colour palette

```js
const C = {
  darkBg:   [26,  10,  0],    // near-black header/footer bands
  maroon:   [139, 42, 16],    // section title colour
  redLight: [232, 96, 74],    // accent highlights
  cream:    [250, 244, 236],  // alternate row background
  ruleLine: [200, 160, 112],  // dividers and rule lines
  textDark: [26,  15,  8],    // body text
  textMeta: [106, 74, 48],    // labels and metadata
};
```

All colours are RGB arrays passed directly to jsPDF's `setFillColor(...arr)` and
`setTextColor(...arr)`. The palette intentionally matches the parchment day-mode
aesthetic of the app.

### Page structure helpers

```js
function headerBand(doc, observerName) {
  // Dark strip across top of page with title left, observer name right
}
function footerLine(doc) {
  // Rule line above page number and generation date
}
function addPage(doc, observerName) {
  doc.addPage();
  headerBand(doc, observerName);
  footerLine(doc);
}
```

`headerBand` and `footerLine` are called on every new page including the cover.
`addPage` is a convenience wrapper used for every page after the cover.

### Cover page — buildCover()

Draws: dark top band with title, subtitle and observer name below, statistics table
(records count, descriptions count, date range, site, coordinates), dark bottom band.
The statistics table uses `doc.autoTable()` with `theme: "plain"` for clean alignment.

### Index table — buildIndex()

Adds a new page, draws a section heading, then calls `doc.autoTable()` with all
records sorted by M number. Columns: Object, Type, Constellation, Date, Site, Seeing,
Transparency, Limiting Mag, Scope/Aperture.

The `didDrawPage` callback on `autoTable` re-draws the header and footer bands on
continuation pages (when the index overflows onto a second page).

### Individual record pages — buildRecord()

Each record gets one full page. Four sections are drawn using helper functions:

```js
function section(title) {
  // Section heading in maroon caps with a ruled line extending to the right margin
}
function fieldRow(pairs) {
  // Up to 4 label+value pairs in a horizontal row
  // Each pair has a small grey label above and a larger black value below
}
```

Sections: **When & Where** (date, time, site, RA, Dec, PSA page, coordinates),
**Sky Conditions** (seeing, transparency, limiting magnitude), **Equipment**
(scope, aperture, eyepiece, magnification), **Observation Description** (free text,
word-wrapped), **Sketch Notes** (italic, only shown if non-empty).

### generateReport(observerName, subtitle)

```js
function generateReport(observerName, subtitle) {
  // Guard: check jsPDF loaded from CDN
  if (!window.jspdf || !window.jspdf.jsPDF) { … show error … return; }

  // Guard: check records exist
  if (Object.keys(SAVED_FULLNOTES).length === 0) { … show error … return; }

  // Defer heavy work with setTimeout so the "Generating…" status renders first
  setTimeout(function() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF({ orientation: "portrait", unit: "mm", format: "letter" });

    buildCover(doc, observerName, subtitle, records);
    footerLine(doc);   // cover needs footer too
    buildIndex(doc, records, catalog, observerName);
    sorted.forEach(id => buildRecord(doc, id, records[id], catalog[id], observerName));

    doc.save(`${stamp}-al-report.pdf`);
  }, 80);   // 80ms gives browser time to paint the status message
}
```

**Why `setTimeout(..., 80)`:** PDF generation blocks the main thread for several
hundred milliseconds for large record sets. Without the delay, the "Generating…"
status text would never render — the browser would paint the completion message
directly, making it look like nothing happened.

**Why jsPDF from CDN at invocation time:** Loading two ~300KB scripts on page load
would slow initial load for all users. Most users will never generate a PDF in a given
session. CDN loading is deferred until the first time the Generate button is clicked.

**`buildCatalog()`:** Converts the DATA array into a `{M1: {type, con, mag, …}}`
lookup map. The type strings are expanded from DATA abbreviations
(`"Spiral Gal"` → `"Spiral Galaxy"`) for human-readable PDF output.
