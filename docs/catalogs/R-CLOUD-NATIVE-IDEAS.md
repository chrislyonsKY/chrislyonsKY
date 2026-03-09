# Chris Lyons — R Cloud-Native Geospatial Package Ideas

> Filling the gaps between rstac, geoarrow, gdalcubes, duckspatial, and sf

---

## The Landscape Right Now

| Package | What it does | Gap |
|---------|-------------|-----|
| **rstac** | STAC API queries | Returns raw lists, not tidy. No built-in download/processing pipeline. |
| **gdalcubes** | Raster data cubes from image collections + STAC | Raster only. No vector. No GeoParquet. |
| **geoarrow** | GeoParquet ↔ sf via Arrow | Low-level. No tidy read/write convenience. No cloud path handling. |
| **sfarrow** | sf ↔ Parquet/Feather | Still "initial implementation" warnings. Limited partitioned dataset support. |
| **duckspatial** | DuckDB spatial extension for R | Brand new (CRAN Jan 2026). Great for local DuckDB, but no cloud-native integration. |
| **terra** | Raster/vector via GDAL | Can read COGs via /vsicurl/ but not cloud-native by design. No STAC. |
| **sf** | Vector data standard | No GeoParquet, no STAC, no cloud paths. |

**The gap:** No unified "just give me the data" package for cloud-native geospatial in R. Python has this via GeoPandas + PyArrow + pystac-client working together seamlessly. R users stitch together 4-5 packages with glue code.

---

## Package Ideas

### 1. `cloudgeo` — Cloud-Native Geospatial I/O for R

**The unified cloud-native reader/writer.** One package, one API: give it a path (local, S3, GCS, HTTP, STAC catalog URL) and get back sf or terra objects. It handles the format detection, credential chains, and package dispatch internally.

```r
library(cloudgeo)

# Read GeoParquet from S3 — returns sf
permits <- cg_read("s3://ky-open-data/permits.parquet")

# Read COG from HTTP — returns SpatRaster
dem <- cg_read("https://data.ky.gov/dem/tile_001.tif")

# Query STAC, download matching COGs, mosaic into cube
ndvi <- cg_stac_cube(
  stac_url = "https://planetarycomputer.microsoft.com/api/stac/v1",
  collection = "sentinel-2-l2a",
  bbox = c(-84.5, 38.0, -84.0, 38.5),
  datetime = "2024-06-01/2024-08-31",
  bands = c("B04", "B08"),
  fun = function(red, nir) (nir - red) / (nir + red),  # NDVI
  res = 10
)

# Write cloud-native
cg_write(permits, "output/permits.parquet", format = "geoparquet")
cg_write(dem, "output/dem.tif", format = "cog")

# Discover what's in a cloud path
cg_info("s3://bucket/data.parquet")
# → GeoParquet, 2.4M rows, Polygon, EPSG:4326, 340MB
```

**What makes it different from existing packages:** Unified `cg_read()` / `cg_write()` that auto-detects format and dispatches to the right backend (geoarrow for Parquet, terra for COG, sf for vectors, rstac+gdalcubes for STAC cubes). Handles S3/GCS/Azure credential chains via `{aws.s3}` or environment variables. Returns the type the user expects (sf for vectors, SpatRaster for rasters).

**Depends:** sf, terra, arrow, geoarrow, rstac, gdalcubes

---

### 2. `overturer` — Overture Maps Data Access for R

**First-class Overture Maps access from R.** Overture releases massive GeoParquet datasets (buildings, places, transportation, land use, addresses) on S3. Python users access them easily via DuckDB + GeoPandas. R users have no dedicated tooling — they're writing raw DuckDB SQL or struggling with geoarrow.

```r
library(overturer)

# Get buildings in a bounding box
buildings <- ov_buildings(
  bbox = c(-84.5, 38.0, -84.3, 38.2),
  release = "latest"
)

# Get places (POIs) by category
restaurants <- ov_places(
  bbox = c(-84.5, 38.0, -84.3, 38.2),
  categories = "restaurant"
)

# Get road network
roads <- ov_transportation(
  bbox = c(-84.5, 38.0, -84.3, 38.2),
  subtypes = "road"
)

# Get administrative boundaries
counties <- ov_divisions(
  country = "US",
  admin_level = 2  # counties
)

# All functions return sf tibbles, ready for analysis
ggplot(buildings) + geom_sf(fill = "steelblue", alpha = 0.3)
```

**Why this matters:** Overture Maps is becoming the open-source OpenStreetMap successor backed by Meta, Microsoft, AWS, and TomTom. 2+ billion buildings, 50+ million places. Nobody has an R package for it. There's a `overturemapsr` on GitHub but it's minimal. A polished, tidy, well-documented package would get serious traction.

**Depends:** duckdb, sf, arrow

---

### 3. `stacr` — Tidy STAC Workflows for R

**rstac is powerful but verbose.** stacr wraps rstac with a tidy, pipe-friendly API for common workflows: search → filter → download → process. Returns tibbles, not nested lists.

```r
library(stacr)

# Search and get results as a tidy tibble
items <- stac_search(
  url = "https://planetarycomputer.microsoft.com/api/stac/v1",
  collection = "sentinel-2-l2a",
  bbox = c(-84.5, 38.0, -84.0, 38.5),
  datetime = "2024-06-01/2024-08-31",
  query = list("eo:cloud_cover" = list("lt" = 20))
)
# Returns: tibble with id, datetime, cloud_cover, bbox, asset_urls

# Preview results
stac_map(items)  # Interactive leaflet map of footprints

# Download specific assets
stac_download(items, assets = c("B04", "B08"), output_dir = "data/")

# Create a gdalcubes data cube directly
cube <- stac_to_cube(items, bands = c("B04", "B08"), res = 10)

# STAC collection inventory
collections <- stac_collections("https://planetarycomputer.microsoft.com/api/stac/v1")
# Returns: tibble with id, title, description, temporal_extent, spatial_extent
```

**Depends:** rstac, gdalcubes, sf, leaflet

---

### 4. `duckgeo` — DuckDB Spatial Workflows with Tidy Syntax

**duckspatial is new and solid but low-level.** duckgeo provides higher-level workflows: read remote GeoParquet directly into sf via DuckDB, run spatial SQL on cloud data, and manage DuckDB spatial databases with tidy verbs.

```r
library(duckgeo)

# Read remote GeoParquet via DuckDB (fast, memory-efficient)
permits <- dg_read_parquet(
  "s3://ky-open-data/permits.parquet",
  where = "STATUS = 'Active'",
  bbox = c(-84.5, 38.0, -84.0, 38.5)
)

# Spatial SQL on cloud data without downloading
result <- dg_query(
  "SELECT p.PERMIT_ID, p.OPERATOR, b.height
   FROM 's3://bucket/permits.parquet' p
   JOIN 's3://overture/buildings.parquet' b
   ON ST_Intersects(p.geometry, b.geometry)
   WHERE p.STATUS = 'Active'"
)

# Create a persistent spatial database
db <- dg_connect("my_analysis.duckdb")
dg_import(db, "permits", "data/permits.gpkg")
dg_import(db, "buildings", "s3://overture/buildings.parquet", bbox = c(...))
dg_spatial_join(db, "permits", "buildings", predicate = "intersects")
```

**Depends:** duckdb, sf, dbplyr

---

### 5. `pmtilesr` — PMTiles Reader/Writer for R

**Zero R packages handle PMTiles.** PMTiles is a single-file vector tile archive that can be served from static storage (S3, CDN) without a tile server. Overture Maps distributes their building footprints as PMTiles. This package reads PMTiles headers and metadata, extracts tiles for a given bbox/zoom, and creates PMTiles from sf objects.

```r
library(pmtilesr)

# Inspect a PMTiles archive
info <- pm_info("https://overture-tiles.s3.amazonaws.com/buildings.pmtiles")
print(info)
# → Zoom: 0-14, Tile count: 2.3M, Format: MVT, Bounds: [-180,-90,180,90]

# Extract tiles for an area as sf
buildings <- pm_read(
  "https://overture-tiles.s3.amazonaws.com/buildings.pmtiles",
  bbox = c(-84.5, 38.0, -84.3, 38.2),
  zoom = 14
)

# View in leaflet with direct PMTiles protocol
pm_leaflet("buildings.pmtiles")

# Create PMTiles from sf (via tippecanoe or DuckDB GDAL driver)
pm_write(permits_sf, "permits.pmtiles", min_zoom = 8, max_zoom = 14)
```

**Depends:** httr2, protolite (for MVT/protobuf), sf, leaflet

---

### 6. `envdatar` — Environmental Open Data Access for R

**One-stop access to US environmental/regulatory open data.** Wraps EPA, USGS, NWS, and state environmental APIs into a consistent tidy interface. Every GIS analyst in an environmental agency Googles the same APIs every week.

```r
library(envdatar)

# EPA ECHO facility search
facilities <- epa_facilities(
  state = "KY",
  program = "NPDES",
  status = "active"
)

# USGS water data
streamflow <- usgs_streamflow(
  site = "03287500",  # Kentucky River at Frankfort
  start_date = "2024-01-01",
  end_date = "2024-12-31"
)

# NWS weather alerts by county
alerts <- nws_alerts(state = "KY", county = "Fayette")

# USGS StreamStats watershed delineation
watershed <- usgs_delineate(point = c(-84.5, 38.04))

# KY-specific: mining permits (if public API exists)
permits <- ky_mining_permits(county = "Pike", status = "active")

# All return sf tibbles or tidy tibbles
```

**Depends:** httr2, sf, jsonlite

---

### 7. `geochange` — Spatiotemporal Change Detection in R

**Detect and quantify spatial changes between two epochs.** Works with both vector and raster data. Vector: find features that were added, removed, moved, or changed shape. Raster: compute difference maps, classify change, generate change polygons.

```r
library(geochange)

# Vector change detection (e.g., permit boundaries over time)
changes <- detect_vector_change(
  old = permits_2020_sf,
  new = permits_2024_sf,
  key = "PERMIT_ID",
  geometry_tolerance = 1.0  # meters
)
# Returns: added, deleted, moved, reshaped, unchanged

# Raster change detection (e.g., pre/post mining from satellite)
change_map <- detect_raster_change(
  before = rast("ndvi_2020.tif"),
  after = rast("ndvi_2024.tif"),
  method = "difference",  # or "ratio", "regression", "classification"
  threshold = 0.2
)
plot(change_map)

# Change polygons from raster change
change_polys <- change_to_polygons(change_map, min_area_m2 = 100)

# Time series change (>2 epochs)
trend <- detect_trend(
  rasters = list("2018" = r1, "2020" = r2, "2022" = r3, "2024" = r4),
  method = "linear"  # or "mann_kendall"
)
```

**Depends:** sf, terra, Shapely concepts via sf

---

### 8. `georender` — Publication-Ready Map Rendering in R

**Opinionated map rendering that produces print-quality cartographic output.** Wraps ggplot2+sf with sensible cartographic defaults: proper scale bars, north arrows, inset maps, multi-panel layouts, and built-in accessible color palettes (integrating with kartocolors).

```r
library(georender)

# One-liner publication map
render_map(
  permits_sf,
  fill = "STATUS",
  title = "Active Mining Permits — Pike County, KY",
  basemap = "terrain",
  palette = "karto_environmental",
  scalebar = TRUE,
  north_arrow = TRUE,
  inset = "state",  # Auto-generates Kentucky inset with study area box
  output = "permits_map.png",
  dpi = 300
)

# Multi-panel temporal comparison
render_panels(
  list("2018" = permits_2018, "2020" = permits_2020, "2024" = permits_2024),
  fill = "STATUS",
  shared_legend = TRUE,
  ncol = 3,
  output = "permit_change_panels.png"
)
```

**Depends:** ggplot2, sf, ggspatial, patchwork, kartocolors

---

## Which Ones to Build?

| Package | Uniqueness | Effort | Audience | sletter Value |
|---------|-----------|--------|----------|-----------------|
| **cloudgeo** | High — nobody has this in R | High | Broad R spatial | Very high |
| **overturer** | Very high — first polished R package for Overture | Medium | Data scientists, urban analytics | Very high |
| **stacr** | Medium — wraps rstac with tidy API | Medium | Remote sensing R users | Medium |
| **duckgeo** | Medium — extends duckspatial | Medium | Analytics, big spatial data | High |
| **pmtilesr** | Very high — zero R packages for PMTiles | Medium | Web mapping from R | High |
| **envdatar** | High — no unified env data access in R | Medium | Environmental agencies (your people) | High |
| **geochange** | High — no unified change detection | Medium-High | Remote sensing, mining | High (aboveR tie-in) |
| **georender** | Medium — better ggplot2 maps | Medium | Everyone who makes maps in R | Very high |

My recommendations for your profile:
1. **overturer** — First-mover advantage, massive dataset, high visibility
2. **pmtilesr** — Completely unoccupied niche, small scope, impressive demos
3. **cloudgeo** — The "glue" package everyone needs, positions you as a cloud-native leader
4. **envdatar** — Directly serves your domain community
