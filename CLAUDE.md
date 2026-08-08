# CLAUDE.md — Jerusalem Aqueduct Water System

## What this is
A personal research site mapping the Roman-era aqueduct system that supplied
Jerusalem from Solomon's Pools. Single-maintainer project (Avi Klayman),
deployed as a static site via GitHub Pages with a custom domain.

## Architecture
- **index.html** — the page wrapper: title, explanatory text (with an EN/HE
  toggle for this prose), a color legend, the embedded map, "Instructions for
  Use", and a single merged Bibliography (see below).
- **Aqueduct.html** — the actual interactive map. Exported from QGIS via the
  qgis2web plugin (Leaflet-based). Embedded into index.html via `<iframe
  src="Aqueduct.html">`. **Do not hand-edit this file's map/layer logic** —
  re-running the QGIS export would overwrite manual changes. Non-map-logic
  tweaks (e.g. the viewport meta tag) are fine. The QGIS source project
  (`.qgz`/shapefiles/rasters) lives in a separate repo, not under this
  folder: `Aqueduct-Latest` (GitHub: `avikla/Aqueduct-Latest`), locally at
  `Follow-up Past\GIS\GIS Projects\QGIS Projects\Aqueduct-Latest\Aqueduct`.
- **data/\*.js** — one JS file per map layer (lines, tunnels, shafts, points,
  etc.), each assigning a GeoJSON-like feature collection to a global `var`.
  Also qgis2web exports. Each feature's `Link to Article` property holds a
  ready-made `<a href="pdfs/...">` snippet rendered directly into the
  Leaflet popup (via `autolinker.link(...)` in Aqueduct.html, which passes
  existing anchor tags through unchanged). If a PDF is renamed or moved,
  update the href here — nothing else in the codebase reconstructs the path.
  To add a one-off photo/image link to a point feature without touching
  Aqueduct.html's map logic, add a new Feature to a *live* points layer
  (e.g. `DigsArticles_15.js`) with a `Link to Article` property holding an
  `<a href="images/...">` snippet — same popup mechanism, no re-export
  needed (see the Central Solomon / Mamila Pool marker features for the
  exact pattern).
- **pdfs/** — the source-document PDFs referenced from `data/*.js` and listed
  in index.html's Bibliography.
- **css/, js/, webfonts/, legend/** — third-party Leaflet/qgis2web assets,
  vendored as part of the export. Don't hand-edit; they'd be overwritten on
  re-export too.
- **CNAME, robots.txt, sitemap.xml, googled6c81a80d5eb67c3.html** — GitHub
  Pages / SEO requirements. These must stay at the repo root to function.

## Conventions
- The site is a single HTML page (index.html) plus one embedded map page
  (Aqueduct.html) — no build step, no framework, no bundler.
- Translatable wrapper text uses paired `data-en`/`data-he` (or
  `data-en-html`/`data-he-html` for fragments with inline markup) attributes,
  swapped via the toggle script at the bottom of index.html's `<body>`. The
  map itself and the Bibliography are not part of this toggle.
- Regenerating the map from QGIS will overwrite `Aqueduct.html` and
  `data/*.js` — re-apply any manual fixes (e.g. `pdfs/` path prefixes,
  the pinch-zoom viewport fix, the custom home/reset-view control, and the
  hover-tooltip styling on the map's Leaflet controls) after a re-export.
- The Bibliography (`<ol>` inside the collapsible `<details>` in index.html)
  is a single merged list, sorted chronologically oldest → newest. Each entry
  that has a matching local PDF starts with a title-link line
  (`<a href="pdfs/...">Surname – short description (year) [HE/EN]</a><br>`)
  followed by the full academic citation in a `<p dir="auto">`. Entries with
  no local PDF (a handful of web references/blog posts) just have the
  citation `<p>`, no title-link line. Any trailing external link (DOI,
  JSTOR, archive.org, a plain URL) is on its own line, wrapped in
  `<span dir="ltr">` so it never gets reordered by RTL text around it. When
  adding a new source: pick the correct chronological slot, and decide
  whether it needs a PDF (add to `pdfs/`) or is citation-only.

## Gotchas
- `body { height: 100vh; overflow: auto; }` swallows `margin-bottom` on the
  last in-flow element — it won't show up when scrolled to the bottom. Use
  an explicit spacer element with `height` instead.
- To give English text a different font while keeping Hebrew on Arial, just
  list the new font first with Arial as fallback (e.g. `'Trebuchet MS',
  Arial, sans-serif`) — most non-Hebrew fonts lack Hebrew glyphs, so the
  browser auto-falls-back to Arial per character, even mid-line. No
  lang-scoped CSS needed.
- `file://` is blocked by the Playwright browser tool's CSP — serve the
  site with `python -m http.server <port>` first, then navigate to
  `localhost`.
- `dir="auto"` on a parent element (e.g. a Bibliography `<li>`) scans *all*
  descendant text to pick a direction — including a sibling English
  title-link before the citation. If the label starts with Latin text, the
  whole `<li>` (citation included) resolves to LTR even when the citation is
  Hebrew, producing ragged/inconsistent alignment. Fix: put `dir="auto"`
  directly on the citation `<p>` itself so it resolves independently of any
  preceding label.
- A raw LTR URL embedded inline in RTL Hebrew text gets visually reordered
  by the bidi algorithm and looks mangled. Put the link on its own line
  (`<br>` before it) wrapped in `<span dir="ltr">` to isolate it.
- `pdftotext` (available via Git Bash) needs `-enc UTF-8` to extract Hebrew
  text correctly — without it, Hebrew glyphs come out as blank/garbled
  spaces while English text extracts fine. `pdftoppm` (needed for the Read
  tool's PDF-page-image rendering) is not installed in this environment, so
  page images can't be previewed — use `pdftotext -enc UTF-8` instead to
  pull citation/bibliographic info out of a new PDF.
- Not every file in `data/` is live. `Aqueduct.html` only `<script>`-loads a
  specific subset (grep `\.js["']` in Aqueduct.html to see which — currently
  15 files, IDs 2–16). The rest are stale leftovers from earlier qgis2web
  export runs sharing a base layer name with a different ID suffix (e.g.
  `Places_12/13/14.js` next to the live `Places_16.js`). Editing a non-live
  file has no effect on the site — check the script tags before assuming a
  `data/*.js` file matters.
- Hand-editing the Bibliography's `<li>` entries with the Edit tool is
  fragile: word-wrapped lines use inconsistent tab counts, so a
  hand-typed multi-line `old_string` frequently fails to match even when it
  looks right. For any edit touching more than one or two entries, write a
  small Node script that locates each `<li>` by a short unique text marker
  and manipulates it programmatically instead of matching exact
  whitespace.

## Deployment
GitHub Pages, custom domain via `CNAME` → `jerusalem.meteor.co.il`.

## License
CC BY-NC-ND (see README.md) — attribution required, non-commercial only, no
derivatives.
