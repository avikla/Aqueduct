# CLAUDE.md — Jerusalem Aqueduct Water System

## What this is
A personal research site mapping the Roman-era aqueduct system that supplied
Jerusalem from Solomon's Pools. Single-maintainer project (Avi Klayman),
deployed as a static site via GitHub Pages with a custom domain.

## Architecture
- **index.html** — the page wrapper: title, explanatory text (with an EN/HE
  toggle for this prose), a color legend, the embedded map, "Instructions for
  Use", a "Source Documents" list, and the academic Bibliography.
- **Aqueduct.html** — the actual interactive map. Exported from QGIS via the
  qgis2web plugin (Leaflet-based). Embedded into index.html via `<iframe
  src="Aqueduct.html">`. **Do not hand-edit this file's map/layer logic** —
  re-running the QGIS export would overwrite manual changes. Non-map-logic
  tweaks (e.g. the viewport meta tag) are fine.
- **data/\*.js** — one JS file per map layer (lines, tunnels, shafts, points,
  etc.), each assigning a GeoJSON-like feature collection to a global `var`.
  Also qgis2web exports. Each feature's `Link to Article` property holds a
  ready-made `<a href="pdfs/...">` snippet rendered directly into the
  Leaflet popup (via `autolinker.link(...)` in Aqueduct.html, which passes
  existing anchor tags through unchanged). If a PDF is renamed or moved,
  update the href here — nothing else in the codebase reconstructs the path.
- **pdfs/** — the source-document PDFs referenced from `data/*.js` and listed
  in index.html's "Source Documents" section.
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
  the pinch-zoom viewport fix) after a re-export.

## Deployment
GitHub Pages, custom domain via `CNAME` → `jerusalem.meteor.co.il`.

## License
CC BY-NC-ND (see README.md) — attribution required, non-commercial only, no
derivatives.
