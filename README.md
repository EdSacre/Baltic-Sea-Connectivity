# Baltic Connectivity Explorer

An interactive web map of species connectivity in the Baltic Sea (375 species, 3 km hex grid).

The app is the `docs/` folder: `docs/index.html` plus the data files it loads. It runs without a server, so you can double-click `docs/index.html` to open it locally, and it can be published as is with GitHub Pages.

## Using the app

- **Species:** pick a species from the list, or browse with the **↑ / ↓** (or **← / →**) keys. **PgUp/PgDn** jumps 10 species, **Home/End** goes to the first or last, and **/** opens the search (scientific, English or Swedish names). The first species alphabetically is selected when the app opens.
- **Panel:** everything sits in one panel on the right. At the top: the species' scientific name, photo, common and Swedish names, group, mobility, archetype and the photo credit. Below that: the colour legend and the display settings. The arrow button collapses the panel (on phones it is a bottom sheet, collapsed at first).
- **Filters:** filter by group (fish, invertebrates, macrophytes), mobility or archetype. The arrow keys only step through the species that match.
- **Variable:** choose any column from the GeoPackages as *Source* or *Sink*; *Strength (Tukey)* is the default. **S** switches between the two and **1–6** picks a row.
- **Colours:** 21 colour schemes (RdYlBu by default) and a reverse button (**R**). Colours always run linearly from the selected species' minimum to its maximum, so the legend just reads Low – Medium – High (values are relative within each species; hover or click a cell for the actual value).
- **Cells:** hover a cell to see its value. Click a cell to see all variables for it and its percentile within the species. Cells inside the model domain without data for the species are shown in grey, and cell outlines appear when zoomed in.
- **Map:** Light (default), Dark or Satellite basemap. If these can't be loaded (no internet), the app falls back to the model land mask automatically.
- **Export image:** saves a portrait map image (900 × 1000 at 1×, up to 2700 × 3000 px at 3×) as PNG or JPG. Choose *Whole Baltic* (default) or *Current view* (centred on what the map shows now). The image adds the species name, common name, group and mobility, a Low – Medium – High legend and a scale bar. It has no basemap credit, so when you publish it, credit the basemap in the caption: © OpenStreetMap contributors © CARTO (Light/Dark) or Esri (Satellite).
- **Copy link:** creates a URL that reopens the current view (species, variable, colours and map position).

## Publishing on GitHub Pages

1. Commit `docs/` to a GitHub repository. Keep the large `products/` and `inputs/` folders out of the repository; `.gitignore` already excludes them.
2. On GitHub, go to **Settings → Pages**, choose **Deploy from a branch**, then select your branch and the **/docs** folder.
3. After a minute the app is live at `https://<user>.github.io/<repo>/`.

`docs/` is about 75 MB (56 MB of data plus 19 MB of species photos). Each species file (0.1–1.7 MB) is downloaded only when that species is viewed.

## Updating the data

When the GeoPackages in `products/strength_gpkg/` or the CSVs in `inputs/` change, rebuild the data files from the project root:

```bash
Rscript tools/build_data.R
```

This needs the R packages `sf`, `terra`, `RSQLite`, `data.table`, `jsonlite`, `base64enc` and `geojsonsf`, and takes about 40 seconds. It writes:

| File | Contents |
| --- | --- |
| `docs/data/manifest.js` | species list (display names from `inputs/WP3_species_list_full.csv`, group, mobility, archetype), archetype descriptions, columns, statistics for each column across all species |
| `docs/data/grid.js` | the shared hex grid, stored as lattice indices (the app rebuilds the hexagons exactly) |
| `docs/data/land.js` | simplified land mask for the offline basemap |
| `docs/data/species/*.js` | one file per species: which cells it occupies, plus every column stored as 16-bit values (counts exactly; other values on a log scale, max. error 0.02 %) |

The species photo window has its own script, because resizing the photos takes a few minutes:

```bash
Rscript tools/build_details.R
```

It reads `inputs/species_details/species_common_names_images.csv` (with `name_overrides.csv` applied) and the photos in `inputs/species_details/species_images/`, and writes `docs/data/details.js` (names and photo credits) and `docs/data/img/<species>.jpg` (thumbnails, 640 px on the longest side, about 19 MB in total). It needs `data.table`, `jsonlite`, `jpeg` and `png`. Thumbnails are only remade when the source photo is newer, so delete `docs/data/img/` to force a full rebuild.

New columns in the GeoPackages show up in the app automatically. To give them a nicer label, description or unit, edit `COLUMN_INFO` and `METRIC_ROWS` near the top of the script in `docs/index.html`. The column descriptions there were written from the column names, so check them before sharing.

## Testing locally over HTTP (optional)

Opening `index.html` directly works fine. To test exactly what GitHub Pages will serve, run:

```bash
Rscript tools/serve.R
```

Then open http://localhost:8765.

## Technical notes

- The hexes are drawn by a custom WebGL layer in MapLibre GL JS 4.7.1, which is included in `docs/lib/` so the app has no CDN dependency. All 57,948 cells are on the GPU as a single draw call. Changing species, variable, scale or colours only uploads a small value buffer or a 256-pixel colour texture.
- The data files are JavaScript rather than JSON because browsers block `fetch()` for pages opened from disk (`file://`). Loading them as scripts works in both settings.
- Basemaps come from CARTO (Light/Dark) and Esri (Satellite imagery and its place-name labels). If they can't be reached, the app switches to the offline land mask automatically.
