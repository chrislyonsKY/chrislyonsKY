# Chris Lyons — QGIS Plugin Catalog

> QGIS Processing Providers and plugins for spatial diagnostics, data quality, cloud-native workflows, and Kentucky LiDAR access  
> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)

-----

## Philosophy

The QGIS plugin ecosystem is mature for cartographic tasks but thin on the practitioner tooling that GIS analysts actually need daily: data quality gates, projection diagnostics, cloud-native data access, and regulatory workflow integration. These plugins fill that gap — they add algorithms to the Processing Toolbox so they compose naturally with QGIS’s model builder and batch processing, and work seamlessly alongside Python scripts and existing workflows.

All plugins are distributed as Processing Providers where possible. That means every algorithm is available in the Toolbox, Model Builder, batch processor, and callable from the Python console via `processing.run()` — not locked behind a bespoke UI.

-----

## QGIS Plugins (8)

### Tier 1: Diagnostics & Quality

-----

#### 1. `tissot` 🔌

**See your distortion. A visual-first geospatial diagnostics engine.**

Named after [Tissot’s indicatrix](https://en.wikipedia.org/wiki/Tissot%27s_indicatrix) — the ellipses that reveal what map projections hide. Tissot makes spatial data problems visible. One command (or one algorithm run), zero config, opens an interactive MapLibre report in your browser.

**Gap it fills:** QGIS will reproject on the fly, but it won’t tell you what distortion you’re incurring or prove that your CRS choice is wrong. No existing plugin computes per-feature distortion, generates a Tissot ellipse overlay, and recommends a better CRS with quantified evidence.

**QGIS Processing algorithms:**

- `Projection X-Ray` — per-feature area and angular distortion heatmap, Tissot ellipses at sample locations, CRS recommendation with quantified improvement
- `Data Quality Check` — geometry validity, topology, schema completeness, duplicate detection; returns a spatial findings layer
- `Map Quality Score` — 0–100 Lighthouse-style rating across CRS choice, data quality, and cloud-native format best practices
- `Spatial Diff` — compares two versions of a dataset; returns a categorized change layer (added / deleted / moved / reshaped / unchanged)

**CLI (outside QGIS):**

```
tissot xray parcels.gpkg       # Projection X-Ray
tissot check parcels.gpkg      # Data quality lint
tissot score project.qgz       # Quality score
tissot diff Q3.gpkg Q4.gpkg    # Visual diff
tissot fix parcels.gpkg --reproject  # Auto-fix CRS
tissot watch ./pipeline/output/      # Watch for changes
```

**Architecture:** Rust core (GeoRust ecosystem, Rayon parallelism), Python bindings via PyO3, visual reports via MapLibre GL JS — fully offline.

**Install:**

```bash
pip install tissot              # Step 1: Python package into QGIS Python
# Step 2: Plugins → Manage → search "Tissot Processing Provider"
```

**GitHub:** https://github.com/chrislyonsKY/tissot  
**QGIS Plugin Repo:** https://plugins.qgis.org/plugins/tissot_processing_provider/  
**PyPI:** `tissot` | **crates.io:** `tissot`  
**QGIS Plugin Registry:** Yes

-----

#### 2. `spatial-validate-qgis` 🔌

**Geometry and attribute validation as QGIS Processing algorithms.**

The QGIS surface for `spatial-validate` (Python) and `spatialvalidate` (R). Exposes composable validation checks as individual Processing algorithms that can be chained in Model Builder or run in batch mode across an entire data estate.

**Gap it fills:** QGIS has `Check Validity` (geometry only) and `Fix Geometries`. There’s nothing that checks attribute domains, required fields, CRS contracts, or topology rules in a composable, CI-friendly way with structured output suitable for a validation report.

**QGIS Processing algorithms:**

- `Validate Layer` — run all checks against a layer; returns a findings layer + summary table
- `Check Geometry` — validity, no nulls, no empties, minimum area/length
- `Check Attributes` — required fields, domain values, unique fields, no-null constraints
- `Check CRS` — matches expected EPSG, is projected vs geographic, is appropriate for extent
- `Check Topology` — no overlaps, no gaps, no self-intersections (polygon layers)
- `Fix and Validate` — `Fix Geometries` + revalidate in one step
- `Generate Validation Report` — HTML/PDF report from findings layer

**Shared with:** `spatial-validate` (Python), `spatialvalidate` (R) — same check names and YAML contract format across all three ecosystems.

**QGIS Plugin Registry:** Yes (planned)

-----

#### 3. `schema-enforcer-qgis` 🔌

**YAML schema contracts for QGIS layers.**

Define expected schemas as YAML, validate layers against them from within QGIS. The same contract files used by `schema-enforcer` (Python) and `schemaenforcer` (R) work here — write once, enforce anywhere.

**Gap it fills:** QGIS has no concept of schema contracts. Analysts manually check field names and types. This brings declarative schema enforcement to the Toolbox with CI-compatible pass/fail output.

**QGIS Processing algorithms:**

- `Validate Schema` — check a layer against a YAML contract; output: pass/fail + violations table
- `Generate Contract` — auto-generate a YAML contract from an existing layer’s schema
- `Compare Schemas` — diff the schema of two layers; useful for detecting ETL-introduced changes
- `Enforce Schema` — apply the contract: rename fields, cast types, drop unknown fields (destructive — requires confirmation)

**Shared contract format:** `.nil-schema.yml` — identical to Python and R implementations.

**QGIS Plugin Registry:** Yes (planned)

-----

### Tier 2: Cloud-Native Data Access

-----

#### 4. `cloudgeo-qgis` 🔌

**Cloud-native geospatial I/O as QGIS Processing algorithms.**

The QGIS surface for `cloudgeo` (Python). Read GeoParquet, COG, STAC, and PMTiles directly into QGIS layers without writing local copies. All cloud-native reads happen in the Processing framework — progress visible, cancellable, logged.

**Gap it fills:** QGIS can open COGs via `/vsicurl/` if you know the pattern, but there’s no unified algorithm that handles GeoParquet, STAC catalog queries, PMTiles, and S3 sources with a consistent UI and credential chain.

**QGIS Processing algorithms:**

- `Read Cloud Source` — universal reader; auto-detects format from URI (S3, GCS, HTTPS, STAC); returns vector or raster layer
- `Query GeoParquet` — bbox + attribute filter pushdown via DuckDB; returns vector layer
- `Query STAC` — search a STAC catalog by bbox + datetime + collection; load results as raster layers
- `Read PMTiles` — extract tiles for bbox/zoom from a PMTiles archive → vector layer
- `Inspect Source` — report format, CRS, extent, field schema, row count without full read
- `Write Cloud-Native` — write any layer to GeoParquet, COG, or PMTiles

**Depends on:** `cloudgeo` Python package (auto-detected on plugin load)

**QGIS Plugin Registry:** Yes (planned)

-----

#### 5. `overturer-qgis` 🔌

**Overture Maps data access for QGIS.**

The QGIS surface for `overturer` (Python/R). Query Overture Maps Foundation datasets (buildings, places, transportation, divisions, addresses) by drawing a bbox on the map canvas. Returns sf-style vector layers without downloading the full dataset.

**Gap it fills:** No QGIS plugin provides Overture Maps access. The data exists (2B+ buildings, 50M+ places) but requires raw DuckDB SQL or Python scripting today. This makes it one-click from the Toolbox.

**QGIS Processing algorithms:**

- `Get Buildings` — bbox → Overture buildings layer with height, subtype, confidence attributes
- `Get Places` — bbox + optional category filter → POI layer
- `Get Transportation` — bbox + subtype (road / rail / water) → road network layer
- `Get Divisions` — country + admin level → administrative boundary layer
- `Get Addresses` — bbox → address point layer
- `List Releases` — table of available Overture data releases with dates and sizes

**Depends on:** `overturer` Python package (auto-detected on plugin load); DuckDB with spatial extension

**QGIS Plugin Registry:** Yes (planned)

-----

#### 6. `kyfromabove-qgis` 🔌

**KyFromAbove LiDAR and elevation data access for QGIS.**

Browse, preview, and download KyFromAbove LiDAR point clouds, DEMs, and orthoimagery directly from QGIS. Connects to the KyFromAbove STAC API and S3 bucket — no account or credentials required. Equivalent to the Indiana LiDAR plugin but built on live STAC discovery rather than a bundled tile index.

**Gap it fills:** KyFromAbove data is public and free but discovery requires navigating the STAC browser or writing raw API calls. No QGIS plugin exists for Kentucky. This puts statewide 2ft DEMs, COPC point clouds, and 3-inch orthoimagery one click away for any QGIS user in Kentucky.

**QGIS Processing algorithms:**

- `Find Tiles` — draw a bbox, select product (DEM phase 1/2/3, orthoimagery, point cloud) → tile list with download URLs
- `Download Tiles` — download selected tiles to a local directory with progress + cancellation
- `Download and Load` — download + add to map canvas in one step
- `Preview Tile` — stream a single COG tile via `/vsicurl/` for preview without download
- `Get Tile Index` — return the tile grid as a vector layer for visualization

**STAC endpoint:** `https://spved5ihrl.execute-api.us-west-2.amazonaws.com/`  
**S3 bucket:** `s3://kyfromabove/` (public, us-west-2)  
**Native CRS:** EPSG:3089 (KY Single Zone, US feet)

**QGIS Plugin Registry:** Yes (planned)

-----

### Tier 3: Visualization & Accessibility

-----

#### 7. `kartocolors-qgis` 🔌

**Accessible cartographic color palettes for QGIS.**

The QGIS surface for `kartocolors` (R) and `kartocolors` (Python/npm). Adds pre-tested, WCAG 2.1 AA compliant cartographic palettes to QGIS’s color ramp picker. Every palette is colorblind-safe across deuteranopia, protanopia, and tritanopia.

**Gap it fills:** QGIS ships with ColorBrewer and viridis. ColorBrewer palettes aren’t all colorblind-safe. Viridis is excellent for continuous data but limited for qualitative and diverging cartographic use cases. There’s no palette library designed specifically for regulatory and environmental mapping.

**What it adds:**

- 30+ named palettes organized by use (terrain, landuse, hydrology, population, environmental, diverging, sequential)
- Each palette available as a QGIS color ramp (appears in all color ramp pickers across the app)
- Palette browser panel: preview, accessibility score, CVD simulation side-by-side
- `Apply Palette` Processing algorithm: apply a kartocolors palette to a vector layer’s symbology

**Palette categories:**

- `karto_terrain` — elevation, slope, aspect
- `karto_landuse` — NLCD / LULC classification
- `karto_hydro` — water quality, streamflow
- `karto_environmental` — permit status, compliance, risk
- `karto_diverging_*` — change detection, before/after comparison
- `karto_sequential_*` — density, concentration, magnitude

**Shared palette definitions:** Identical hex values across R (`kartocolors`), Python (`kartocolors`), npm (`@chrislyonsKY/kartocolors`), and this plugin.

**QGIS Plugin Registry:** Yes (planned)

-----

#### 8. `map-accessibility-qgis` 🔌

**WCAG 2.1 AA color accessibility auditing for QGIS maps.**

The QGIS surface for `map-accessibility` (Python) and `mapaccessibility` (R). Audit your map’s color choices for contrast ratio compliance and colorblind accessibility. Works on any QGIS layer’s current symbology.

**Gap it fills:** QGIS has no accessibility auditing. Regulatory agency GIS outputs must meet Section 508 requirements. Analysts currently guess or use external tools. This brings WCAG 2.1 contrast ratio checking and CVD simulation directly into the layer properties workflow.

**QGIS Processing algorithms:**

- `Audit Layer Accessibility` — analyze all colors in a layer’s symbology; return contrast ratios and pass/fail against WCAG 2.1 AA
- `Simulate CVD` — render a map preview as seen under deuteranopia, protanopia, and tritanopia
- `Score Map Palette` — 0–1 accessibility score for the current map’s color usage
- `Suggest Accessible Colors` — given a failing color, return passing alternatives that preserve hue intent

**Panel:** Accessibility panel shows live contrast ratio as you edit symbology — red/green indicator updates as you pick colors.

**Shared WCAG algorithms:** Identical contrast ratio math and CVD simulation to `map-accessibility` (Python) and `mapaccessibility` (R).

**QGIS Plugin Registry:** Yes (planned)

-----

## Plugin Structure Template

Every QGIS plugin in this org follows this structure:

```
plugin-name/
├── metadata.txt               # Plugin manifest (name, version, QGIS min version)
├── __init__.py                # classFactory(iface) entry point
├── plugin_main.py             # Main plugin class (initGui, unload)
├── CLAUDE.md                  # AI-dev context for Claude Code
├── README.md                  # User-facing docs with install instructions
├── LICENSE                    # MIT
├── .github/
│   └── workflows/
│       └── qgis-plugin-ci.yml # Lint, test, package, deploy to plugin repo
├── ai-dev/
│   ├── architecture.md        # Module design, data flow, key decisions
│   ├── spec.md                # Requirements, acceptance criteria, user stories
│   ├── agents/                # Reusable agent prompt configs
│   └── decisions/             # Architectural decision log (DL-NNN-*.md)
├── src/
│   ├── algorithms/            # One file per Processing algorithm
│   ├── provider.py            # QgsProcessingProvider subclass
│   └── utils/                 # Shared utilities
├── test/
│   ├── conftest.py
│   └── test_*.py              # Unit tests (QGIS-independent where possible)
└── inst/
    └── extdata/               # Sample data for algorithm examples
```

-----

## Cross-Ecosystem Comparison

|Domain                |Python             |R                 |QGIS Plugin             |
|----------------------|-------------------|------------------|------------------------|
|Projection diagnostics|—                  |—                 |`tissot`                |
|Validation            |`spatial-validate` |`spatialvalidate` |`spatial-validate-qgis` |
|Schema enforcement    |`schema-enforcer`  |`schemaenforcer`  |`schema-enforcer-qgis`  |
|Cloud-native I/O      |`cloudgeo`         |`cloudgeo` (R)    |`cloudgeo-qgis`         |
|Overture Maps         |`overturer`        |`overturer`       |`overturer-qgis`        |
|KyFromAbove LiDAR     |—                  |`aboveR`          |`kyfromabove-qgis`      |
|Accessible palettes   |`kartocolors`      |`kartocolors`     |`kartocolors-qgis`      |
|Accessibility auditing|`map-accessibility`|`mapaccessibility`|`map-accessibility-qgis`|

-----

## Build Priority

|Phase|Plugin                  |Rationale                                              |
|-----|------------------------|-------------------------------------------------------|
|**1**|`tissot`                |Already in development; QGIS plugin repo badge live    |
|**1**|`kyfromabove-qgis`      |Unique niche, active scaffolding, clear KY user base   |
|**2**|`kartocolors-qgis`      |Low complexity; drives adoption of the palette standard|
|**2**|`spatial-validate-qgis` |High practitioner value; mirrors Python/R companions   |
|**3**|`cloudgeo-qgis`         |Depends on `cloudgeo` Python package being stable      |
|**3**|`overturer-qgis`        |Depends on `overturer` Python package; high visibility |
|**4**|`schema-enforcer-qgis`  |Depends on schema-enforcer Python package              |
|**4**|`map-accessibility-qgis`|Depends on map-accessibility Python package            |
