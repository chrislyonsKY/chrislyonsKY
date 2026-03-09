# Chris Lyons — R Cloud-Native Package Architectures

> 8 packages, each with architecture, module design, key decisions, and dependency graph  
> Based on design decisions confirmed 2026-03-08

---

## 1. cloudgeo — Cloud-Native Geospatial I/O for R

### Architecture Summary

`cg_read()` is the universal entry point. It inspects the input (file extension, URI scheme, URL pattern) and dispatches to the right backend. Returns familiar R spatial objects: sf for vectors, SpatRaster for rasters. Never returns an unfamiliar type without warning.

### Key Decision

STAC URLs return a **SpatRaster of the most recent matching scene**, with a cli message: `ℹ Found 47 matching scenes. Showing most recent. Use cg_stac_cube() for time series.` This teaches the cube workflow progressively without ambushing new users.

### Module Design

```
R/
├── cg_read.R              # Universal reader — dispatches by format/URI
├── cg_write.R             # Universal writer — format from extension
├── cg_info.R              # Inspect remote/local source without reading
├── cg_stac_cube.R         # STAC → gdalcubes data cube (explicit call)
├── detect.R               # Format detection from path/URI/headers
├── backends/
│   ├── backend_vector.R   # sf::st_read, geoarrow, duckdb dispatch
│   ├── backend_parquet.R  # GeoParquet via geoarrow::read_geoparquet_sf()
│   ├── backend_raster.R   # terra::rast() with /vsicurl/ for cloud
│   ├── backend_stac.R     # rstac query → terra or gdalcubes
│   └── backend_pmtiles.R  # PMTiles via pmtilesr (Suggests)
├── auth.R                 # Credential chain: env vars → aws.s3 → default
├── utils.R                # URI parsing, path normalization
└── zzz.R                  # .onLoad — check optional deps
```

### Data Flow

```
cg_read(source)
    │
    ├─ detect_format(source)
    │   ├─ "s3://...parquet"  → backend_parquet → sf
    │   ├─ "s3://...tif"      → backend_raster  → SpatRaster
    │   ├─ "https://...stac"  → backend_stac    → SpatRaster (latest scene)
    │   ├─ "postgresql://..."  → sf::st_read     → sf
    │   ├─ local .gpkg/.shp   → sf::st_read     → sf
    │   └─ local .tif         → terra::rast      → SpatRaster
    │
    └─ return sf | SpatRaster
```

### Dependencies

- **Imports:** sf, terra
- **Suggests:** arrow, geoarrow, rstac, gdalcubes, aws.s3, cli

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | SpatRaster for STAC single scenes, cg_stac_cube() for time series | Progressive disclosure — familiar return types first |
| AD-02 | Format detection from extension + URI scheme + HTTP content-type | Covers local, S3, GCS, HTTP, STAC patterns |
| AD-03 | Credential chain: env vars → aws.s3 config → GDAL /vsicurl/ | Matches what users already have configured |

---

## 2. overturer — Overture Maps Data Access for R

### Architecture Summary

DuckDB-only backend. Every `ov_*()` function generates a DuckDB SQL query against the Overture S3 GeoParquet, executes it, and returns an sf tibble. The user never sees SQL unless they want to via `ov_query()`. DuckDB's S3 predicate pushdown means only the bbox-matching row groups are downloaded.

### Key Decision

**DuckDB only.** Overture's partition scheme and bbox columns are optimized for DuckDB predicate pushdown. No arrow fallback — keeps the codebase simple and the performance predictable.

### Module Design

```
R/
├── ov_buildings.R          # ov_buildings(bbox, ...) → sf
├── ov_places.R             # ov_places(bbox, categories, ...) → sf
├── ov_transportation.R     # ov_transportation(bbox, subtypes, ...) → sf
├── ov_divisions.R          # ov_divisions(country, admin_level, ...) → sf
├── ov_addresses.R          # ov_addresses(bbox, ...) → sf
├── ov_base.R               # ov_base(bbox, type, ...) → sf (land, water, etc.)
├── ov_query.R              # ov_query(sql) → sf — raw SQL escape hatch
├── ov_releases.R           # ov_releases() → tibble of available releases
├── connection.R            # DuckDB connection pool, S3 config, spatial ext
├── sql_builder.R           # Build parameterized SQL from R args
├── bbox_utils.R            # Bbox validation, Overture bbox column filtering
└── zzz.R                   # .onLoad — install/load DuckDB spatial extension
```

### Data Flow

```
ov_buildings(bbox = c(-84.5, 38.0, -84.3, 38.2))
    │
    ├─ validate_bbox(bbox)
    ├─ build_sql(theme = "buildings", type = "building", bbox, ...)
    │   → "SELECT ... FROM read_parquet('s3://overturemaps.../theme=buildings/...')
    │      WHERE bbox.xmin BETWEEN ... AND bbox.ymin BETWEEN ..."
    ├─ duckdb_execute(sql) → arrow Table
    ├─ arrow_to_sf(table) → sf tibble
    └─ return sf
```

### Dependencies

- **Imports:** duckdb, sf, DBI, cli
- **Suggests:** arrow (for large result optimization)

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | DuckDB only, no arrow fallback | Overture's partitions are optimized for DuckDB predicate pushdown |
| AD-02 | One function per Overture theme (ov_buildings, ov_places, etc.) | Discoverable API — autocomplete shows what's available |
| AD-03 | Release pinning: default "latest", explicit version optional | Reproducibility when needed, convenience by default |
| AD-04 | Connection pool — reuse DuckDB connection across calls | Avoids repeated spatial extension loading overhead |

---

## 3. stacr — Tidy STAC Workflows for R

### Architecture Summary

Wraps rstac with a tidy, pipe-friendly API. All results return tibbles. Ships with a catalog registry of known-good STAC endpoints (Planetary Computer, Earth Search, USGS, etc.) but works with any STAC API.

### Key Decision

**Generic API + stac_catalogs() registry.** The registry provides convenience (`stac_search(catalog = "planetary_computer", ...)`) while the generic path works with any STAC URL. Registry is a tibble shipped as package data, updatable via `stac_update_registry()`.

### Module Design

```
R/
├── stac_search.R           # stac_search(url, collection, bbox, ...) → tibble
├── stac_collections.R      # stac_collections(url) → tibble of collections
├── stac_items.R            # stac_items(url, collection, ...) → tibble of items
├── stac_download.R         # stac_download(items, assets, output_dir)
├── stac_to_cube.R          # stac_to_cube(items, bands, res) → gdalcubes cube
├── stac_map.R              # stac_map(items) → leaflet map of footprints
├── stac_catalogs.R         # stac_catalogs() → tibble of known endpoints
├── registry.R              # Registry management (load, update, add custom)
├── tidy_stac.R             # Convert rstac results to tidy tibbles
└── utils.R                 # Bbox handling, datetime parsing
```

### Data Flow

```
stac_search(catalog = "planetary_computer",
            collection = "sentinel-2-l2a",
            bbox = c(-84.5, 38.0, -84.0, 38.5),
            datetime = "2024-06/2024-08")
    │
    ├─ resolve_catalog("planetary_computer") → URL from registry
    ├─ rstac::stac(url) |> stac_search(...) |> post_request()
    ├─ tidy_stac(result) → tibble with columns:
    │   id, datetime, cloud_cover, bbox, thumbnail_url, asset_urls
    └─ return tibble
```

### Dependencies

- **Imports:** rstac, tibble, cli
- **Suggests:** gdalcubes, leaflet, sf

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | Registry of known catalogs as package data | Convenience for common catalogs, discoverable |
| AD-02 | All results as tibbles, never nested lists | Tidy principle — everything is a data frame |
| AD-03 | stac_to_cube() bridges to gdalcubes explicitly | Clear handoff point, not hidden magic |

---

## 4. duckgeo — DuckDB Spatial Workflows with Tidy Syntax

### Architecture Summary

Convenience functions first, dbplyr backend as v2 goal. Provides `dg_read_parquet()`, `dg_spatial_join()`, `dg_query()`, and database management helpers. All functions return sf or tibble. Uses cli for progress on long queries.

### Key Decision

**Convenience functions first.** The 80% case is: read remote GeoParquet with a bbox filter, do a spatial join, get sf back. That doesn't need full dbplyr. v2 can add `tbl_duckgeo()` for lazy evaluation chains.

### Module Design

```
R/
├── dg_connect.R            # Create/reuse DuckDB connections with spatial ext
├── dg_read_parquet.R       # Read GeoParquet (local/S3) → sf, with bbox/where
├── dg_read_file.R          # Read any GDAL format via DuckDB → sf
├── dg_query.R              # Run raw spatial SQL → sf or tibble
├── dg_spatial_join.R       # Spatial join between two sources → sf
├── dg_import.R             # Import sf/file into persistent DuckDB table
├── dg_export.R             # Export DuckDB table to GeoParquet/GPKG/etc.
├── dg_summary.R            # Spatial summary stats via DuckDB aggregations
├── connection.R            # Connection pool, spatial extension auto-loading
├── sql_helpers.R           # SQL builder utilities, parameterized queries
└── utils.R                 # sf ↔ DuckDB geometry conversion
```

### Dependencies

- **Imports:** duckdb, DBI, sf
- **Suggests:** dbplyr, arrow, cli

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | Convenience functions first, dbplyr v2 | Ship fast, learn which operations people chain |
| AD-02 | Auto-install/load DuckDB spatial extension | Users shouldn't have to remember INSTALL/LOAD spatial |
| AD-03 | Connection pool across function calls | Avoids overhead of repeated connection setup |

---

## 5. pmtilesr — PMTiles Reader/Writer for R

### Architecture Summary

Reads PMTiles headers via HTTP range requests. `pm_read()` decodes MVT tiles into sf. `pm_tiles()` returns raw MVT for direct leaflet rendering. `pm_write()` creates PMTiles from sf (via DuckDB GDAL driver or external tippecanoe).

### Key Decision

**Dual return types:** `pm_read()` → sf (decoded), `pm_tiles()` → raw MVT list (for leaflet). Most users want sf. Leaflet integration users want raw tiles.

### Module Design

```
R/
├── pm_info.R               # Inspect header: zoom range, bounds, format, tile count
├── pm_read.R               # Extract tiles for bbox/zoom → decode MVT → sf
├── pm_tiles.R              # Extract raw MVT tiles → list (for leaflet)
├── pm_write.R              # sf → PMTiles (via DuckDB GDAL or tippecanoe)
├── pm_leaflet.R            # Render PMTiles directly in leaflet via protomaps
├── header.R                # PMTiles header parsing (binary format)
├── mvt.R                   # MVT/protobuf decoding → sf geometries
├── http.R                  # HTTP range requests for tile extraction
└── utils.R                 # Tile math (bbox → tile indices, zoom levels)
```

### Data Flow

```
pm_read("https://overture-tiles.s3.amazonaws.com/buildings.pmtiles",
        bbox = c(-84.5, 38.0, -84.3, 38.2), zoom = 14)
    │
    ├─ pm_info() → read header via HTTP range request (first ~16KB)
    ├─ bbox_to_tiles(bbox, zoom) → tile indices (x, y, z)
    ├─ for each tile: HTTP range request → raw MVT bytes
    ├─ decode_mvt(bytes) → sf geometries + attributes
    ├─ rbind all tiles → single sf tibble
    ├─ clip to exact bbox (tiles are rectangular, bbox may not align)
    └─ return sf
```

### Dependencies

- **Imports:** httr2, sf, protolite (MVT/protobuf decoding)
- **Suggests:** leaflet, leaflet.extras, duckdb, cli

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | Dual API: pm_read() → sf, pm_tiles() → raw MVT | Serves both analysis (sf) and visualization (leaflet) users |
| AD-02 | HTTP range requests for tile extraction | No full download needed — read only the tiles in your bbox |
| AD-03 | pm_write() delegates to DuckDB GDAL driver or tippecanoe | Both methods exist; DuckDB is R-native, tippecanoe is faster |

---

## 6. envdatar — Environmental Open Data Access for R

### Architecture Summary

Modular API client architecture. Each data source is a module with consistent interface: `source_function(params) → tibble | sf`. HTTP calls via httr2 with retry logic. Responses cached via memoise or local file cache. All sources: EPA, USGS, KY state, NOAA/NWS.

### Key Decision

**All four sources are first-class.** EPA, USGS, KY state, and NOAA/NWS each get their own module. Consistent function naming: `epa_*()`, `usgs_*()`, `ky_*()`, `nws_*()`.

### Module Design

```
R/
├── epa_facilities.R        # EPA ECHO facility search → sf
├── epa_water_quality.R     # EPA STORET/WQX water quality data → tibble
├── epa_air_quality.R       # EPA AQS air quality data → tibble
├── epa_sdwis.R             # EPA drinking water system info → tibble
├── usgs_streamflow.R       # USGS NWIS instantaneous/daily values → tibble
├── usgs_water_quality.R    # USGS water quality samples → tibble
├── usgs_delineate.R        # USGS StreamStats watershed delineation → sf
├── usgs_nhdplus.R          # NHDPlus feature access → sf
├── ky_mining_permits.R     # KY mining permit data → sf (if public API)
├── ky_water_quality.R      # KY Division of Water data → tibble
├── ky_air_quality.R        # KY air quality monitoring → tibble
├── nws_alerts.R            # NWS active alerts by state/county → sf
├── nws_forecast.R          # NWS point forecast → tibble
├── nws_observations.R      # NWS recent observations → tibble
├── http_client.R           # httr2 wrapper with retry, rate limiting, caching
├── cache.R                 # Local file cache for repeated queries
└── utils.R                 # Coordinate parsing, date handling, bbox helpers
```

### Dependencies

- **Imports:** httr2, jsonlite, tibble, sf, cli
- **Suggests:** memoise (caching), dataRetrieval (USGS bridge)

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | All 4 sources first-class with consistent naming | You use all of them; no second-class citizens |
| AD-02 | httr2 for HTTP (not httr) | Modern, pipe-friendly, built-in retry/rate-limiting |
| AD-03 | Local file cache for repeated queries | Environmental APIs are slow; don't re-fetch unchanged data |
| AD-04 | Return sf when data has coordinates, tibble when tabular | Spatial by default, non-spatial when it makes sense |

---

## 7. geochange — Spatiotemporal Change Detection in R

### Architecture Summary

Two engines: **terra for 2-epoch comparisons** (simple, fast, familiar), **gdalcubes for multi-epoch time series** from cloud-native sources (STAC→cube→trend). Vector change detection via sf. All results are terra/sf objects with change classification attributes.

### Key Decision

**terra + gdalcubes dual backend.** terra for the common case (before/after comparison). gdalcubes for cloud-native time series when the user needs trend analysis across many dates without downloading everything.

### Module Design

```
R/
├── detect_raster_change.R  # 2-epoch raster change: terra in, terra out
├── detect_vector_change.R  # Vector change by key field: sf in, sf out
├── detect_trend.R          # Multi-epoch trend analysis via terra or gdalcubes
├── detect_cloud_change.R   # STAC → gdalcubes → change map (cloud-native path)
├── classify_change.R       # Threshold/classify change maps into categories
├── change_to_polygons.R    # Raster change → vector polygons (min area filter)
├── change_report.R         # Summary statistics from change results
├── methods/
│   ├── difference.R        # Simple subtraction
│   ├── ratio.R             # Before/after ratio
│   ├── regression.R        # Per-pixel linear regression over time
│   └── mann_kendall.R      # Non-parametric trend test
├── utils.R                 # Alignment, resampling, nodata handling
└── zzz.R                   # Check for gdalcubes availability
```

### Dependencies

- **Imports:** terra, sf
- **Suggests:** gdalcubes, rstac, cli

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | terra for 2-epoch, gdalcubes for multi-epoch/cloud | Right tool for each scale of analysis |
| AD-02 | Vector change detection via key-field matching | Enterprise spatial data has stable IDs — match on them |
| AD-03 | Change methods as pluggable functions | difference, ratio, regression, mann_kendall — same interface |
| AD-04 | change_to_polygons() with min_area filter | Avoids noise polygons from single-pixel changes |

---

## 8. georender — Publication-Ready Map Rendering in R

### Architecture Summary

Opinionated ggplot2 wrapper for cartographic output. `render_map()` is the one-liner that adds proper scale bars, north arrows, inset maps, legends, and accessible palettes. Integrates with kartocolors. **Static maps only** — do one thing well.

### Key Decision

**ggplot2 only.** No leaflet, no tmap. Publication-quality static maps with proper cartographic elements. Every output should be print-ready at 300 DPI without tweaking.

### Module Design

```
R/
├── render_map.R            # One-liner map rendering with sensible defaults
├── render_panels.R         # Multi-panel temporal/thematic comparison layouts
├── render_inset.R          # Auto-generate state/region inset with study area box
├── elements/
│   ├── scalebar.R          # Cartographic scale bar (metric + imperial)
│   ├── north_arrow.R       # North arrow styles
│   ├── legend.R            # Enhanced legend placement and formatting
│   ├── title_block.R       # Title, subtitle, source attribution block
│   └── graticule.R         # Lat/lon grid overlay
├── themes/
│   ├── theme_carte.R       # Default georender theme (clean, print-ready)
│   ├── theme_terrain.R     # Terrain/topographic style
│   └── theme_minimal_map.R # Ultra-minimal for data-forward maps
├── basemaps.R              # Annotation basemap tiles (Stamen, CartoDB, etc.)
├── palettes.R              # Integration with kartocolors accessible palettes
└── export.R                # Export helpers: PNG, PDF, SVG at print DPI
```

### Dependencies

- **Imports:** ggplot2, sf, ggspatial
- **Suggests:** patchwork (multi-panel), kartocolors, rosm (basemap tiles)

### Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| AD-01 | ggplot2 only, no interactive maps | Do one thing well. leaflet/mapview exist for interactive. |
| AD-02 | render_map() has 80% defaults, 20% overridable | Sensible out of the box, customizable when needed |
| AD-03 | Auto-inset from state/country boundaries | Most regulatory maps need a location context inset |
| AD-04 | kartocolors integration for accessible palettes | Accessibility built in, not bolted on |
| AD-05 | All outputs at 300 DPI by default | Print-ready without remembering to set DPI |

---

## Dependency Graph (All 8 Packages)

```
                    ┌─────────────────┐
                    │    cloudgeo      │  ← unified entry point
                    │ (sf, terra,      │
                    │  arrow, rstac,   │
                    │  gdalcubes)      │
                    └────────┬────────┘
                             │ uses
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
      ┌──────────┐   ┌──────────┐   ┌──────────┐
      │  stacr   │   │ duckgeo  │   │ pmtilesr │
      │ (rstac,  │   │ (duckdb, │   │ (httr2,  │
      │  tibble) │   │  sf)     │   │  sf)     │
      └──────────┘   └────┬─────┘   └──────────┘
                          │ powers
                          ▼
                   ┌──────────┐
                   │ overturer │  ← DuckDB backend only
                   │ (duckdb, │
                   │  sf)     │
                   └──────────┘

    INDEPENDENT                    INDEPENDENT
    ───────────                    ───────────
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ envdatar │   │geochange │   │georender │
    │ (httr2,  │   │ (terra,  │   │(ggplot2, │
    │  sf)     │   │  sf,     │   │ sf,      │
    │          │   │  gdalcubes│   │ kartocolors)
    └──────────┘   └──────────┘   └──────────┘
```

## Cross-Cutting Conventions

| Convention | User-Facing Packages | Library Packages |
|-----------|---------------------|-----------------|
| **Messaging** | cli (colored, progress bars) | base R message()/warning() |
| **HTTP** | httr2 | httr2 |
| **Vector return** | sf tibble | sf tibble |
| **Raster return** | terra SpatRaster | terra SpatRaster |
| **Tabular return** | tibble | tibble |
| **Error handling** | cli::cli_abort() with context | rlang::abort() |

**User-facing (cli):** cloudgeo, overturer, stacr, envdatar, georender  
**Library (base R):** pmtilesr, duckgeo, geochange
