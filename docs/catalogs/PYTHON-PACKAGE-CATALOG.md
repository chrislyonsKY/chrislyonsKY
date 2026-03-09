# Chris Lyons — Python Package Catalog

> 20 open-source Python packages for cloud-native geospatial workflows  
> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)

---

## Philosophy

The Python geospatial ecosystem (GeoPandas, Shapely, PySTAC, rio-cogeo, DuckDB) is powerful but fragmented. These packages fill specific gaps where practitioners — particularly those working in environmental science, regulatory GIS, and cloud-native workflows — lack unified, well-documented tooling. They don't reinvent the stack; they wire it together cleanly.

All packages follow modern Python conventions: type-annotated, pip/uv installable, tested with pytest, documented with MkDocs or Sphinx.

---

## Landscape Analysis

| Package | What it does | Gap |
|---------|-------------|-----|
| **geopandas** | Vector data in Python | No cloud-native read path; no GeoParquet predicate pushdown |
| **pystac-client** | STAC API queries | Returns raw dicts; no tidy download/processing pipeline |
| **rio-cogeo** | COG validation and creation | Low-level; no unified read API for mixed cloud sources |
| **duckdb** | Analytical SQL engine | No spatial convenience layer; raw SQL for GeoParquet |
| **overturemaps-py** | Overture Maps CLI tool | CLI only; no Python API for programmatic access |
| **shapely** | Geometry engine | No change detection, no validation framework |
| **mapclassify** | Choropleth classification | No accessibility scoring or WCAG checking |
| **lonboard** | High-perf vector rendering | Rendering only; no cartographic elements |
| **pmtiles** | PMTiles spec (JS-first) | No Python reader/writer with GeoPandas integration |
| **langchain-community** | LLM tool integrations | No geospatial-specific tools or coordinate parsers |

---

## Packages (20)

---

### Tier 1: Cloud-Native I/O

---

#### 1. `cloudgeo` 📦

**The unified cloud-native geospatial reader/writer for Python.**

One function — `cg_read()` — accepts any path (local, S3, GCS, HTTPS, STAC URL) and returns a GeoDataFrame or xarray DataArray. Handles format detection, credential chains, and backend dispatch internally.

**Gap it fills:** Python users stitch together geopandas, pystac-client, rio-cogeo, and duckdb with custom glue code for every project. No unified entry point exists.

**Key functions:**
- `cg_read(source, bbox=None, where=None)` → GeoDataFrame | xr.DataArray
- `cg_write(data, path, format=None)` → None (auto-detects format from extension)
- `cg_info(source)` → dict (schema, CRS, extent, row count, format)
- `cg_stac_cube(url, collection, bbox, datetime, bands, res)` → xr.DataArray

```python
import cloudgeo as cg

# Read GeoParquet from S3 — returns GeoDataFrame
permits = cg.cg_read("s3://ky-open-data/permits.parquet")

# Read COG from HTTP — returns xarray DataArray
dem = cg.cg_read("https://data.example.gov/dem.tif", bbox=(-84.5, 38.0, -84.0, 38.5))

# STAC time series cube
ndvi = cg.cg_stac_cube(
    url="https://planetarycomputer.microsoft.com/api/stac/v1",
    collection="sentinel-2-l2a",
    bbox=(-84.5, 38.0, -84.0, 38.5),
    datetime="2024-06/2024-08",
    bands=["B04", "B08"],
    res=10
)

# Write cloud-native
cg.cg_write(permits, "output/permits.parquet")
cg.cg_info("s3://bucket/data.parquet")
# → {"format": "GeoParquet", "rows": 2_400_000, "crs": "EPSG:4326", "size_mb": 340}
```

**Depends:** geopandas, pyarrow, pystac-client, rioxarray, duckdb  
**PyPI:** Yes (planned)

---

#### 2. `overturer` 📦

**First-class Overture Maps access from Python.**

Query Overture Maps Foundation datasets (buildings, places, transportation, divisions, addresses) from Python. DuckDB backend with S3 predicate pushdown — only downloads matching row groups.

**Gap it fills:** `overturemaps-py` is a CLI tool only. Python users write raw DuckDB SQL. No API-level package exists for programmatic access.

**Key functions:**
- `ov_buildings(bbox, release="latest")` → GeoDataFrame
- `ov_places(bbox, categories=None)` → GeoDataFrame
- `ov_transportation(bbox, subtypes=None)` → GeoDataFrame
- `ov_divisions(country, admin_level)` → GeoDataFrame
- `ov_addresses(bbox)` → GeoDataFrame
- `ov_releases()` → pd.DataFrame of available releases
- `ov_query(sql)` → GeoDataFrame (raw SQL escape hatch)

```python
from overturer import ov_buildings, ov_places, ov_releases

bbox = (-84.5, 38.0, -84.3, 38.2)

buildings = ov_buildings(bbox=bbox, release="latest")
restaurants = ov_places(bbox=bbox, categories=["restaurant", "cafe"])
print(ov_releases())
```

**Depends:** duckdb, geopandas, pyarrow  
**PyPI:** Yes (planned)

---

#### 3. `pmtilespy` 📦

**PMTiles reader/writer for Python with GeoPandas integration.**

Read PMTiles archives via HTTP range requests, extract vector tiles as GeoDataFrames, write PMTiles from GeoDataFrames. No existing Python package provides GeoPandas-native PMTiles I/O.

**Gap it fills:** Zero Python packages read PMTiles into GeoDataFrames. The JS ecosystem has `pmtiles`; Python has nothing.

**Key functions:**
- `pm_info(url)` → dict (zoom range, bounds, tile count, layers)
- `pm_read(url, bbox, zoom)` → GeoDataFrame
- `pm_tiles(url, bbox, zoom)` → list of raw MVT bytes
- `pm_write(gdf, path, min_zoom=0, max_zoom=14)` → None

```python
from pmtilespy import pm_info, pm_read, pm_write

info = pm_info("https://overture-tiles.s3.amazonaws.com/buildings.pmtiles")
# {"zoom": [0, 14], "tile_count": 2_300_000, "bounds": [-180, -90, 180, 90]}

buildings = pm_read(
    "https://overture-tiles.s3.amazonaws.com/buildings.pmtiles",
    bbox=(-84.5, 38.0, -84.3, 38.2),
    zoom=14
)
pm_write(buildings, "local_buildings.pmtiles", min_zoom=8, max_zoom=14)
```

**Depends:** httpx, mapbox-vector-tile, geopandas  
**PyPI:** Yes (planned)

---

#### 4. `stacpipe` 📦

**Tidy STAC workflows with a pipeline-friendly API.**

Wraps pystac-client with a clean, chainable API. Returns DataFrames, not nested dicts. Ships with a registry of known STAC endpoints (Planetary Computer, Earth Search, USGS).

**Gap it fills:** pystac-client returns raw Python dicts. No package provides a tidy DataFrame API with a built-in catalog registry and direct path to download/cube workflows.

**Key functions:**
- `stac_search(catalog, collection, bbox, datetime, query)` → pd.DataFrame
- `stac_collections(url)` → pd.DataFrame
- `stac_download(items_df, assets, output_dir)` → list of paths
- `stac_to_cube(items_df, bands, res)` → xr.DataArray
- `stac_catalogs()` → pd.DataFrame of known endpoints
- `stac_map(items_df)` → folium map of footprints

```python
from stacpipe import stac_search, stac_to_cube, stac_catalogs

items = stac_search(
    catalog="planetary_computer",
    collection="sentinel-2-l2a",
    bbox=(-84.5, 38.0, -84.0, 38.5),
    datetime="2024-06/2024-08",
    query={"eo:cloud_cover": {"lt": 20}}
)
# Returns DataFrame: id, datetime, cloud_cover, bbox, asset_urls

cube = stac_to_cube(items, bands=["B04", "B08"], res=10)
```

**Depends:** pystac-client, pandas, odc-stac or stackstac  
**PyPI:** Yes (planned)

---

### Tier 2: Data Quality & Validation

---

#### 5. `spatial-validate` 📦

**Geometry and attribute validation for GeoDataFrames.**

Composable validation checks that return structured DataFrames of findings. Pipe-friendly for integration with pandas/geopandas workflows.

**Gap it fills:** `shapely.is_valid()` exists but there's no unified validation framework checking geometry + attributes + CRS + domains + topology in one pass with structured output.

**Key functions:**
- `validate(gdf, checks)` → ValidationResult (findings DataFrame + summary)
- `check_valid_geometry()`, `check_no_null_geometry()`, `check_crs(epsg)`
- `check_no_nulls(fields)`, `check_unique(field)`, `check_domain(field, values)`
- `check_topology(rule)`, `fix_geometry(gdf)` → repaired GeoDataFrame

```python
from spatial_validate import validate, checks, fix_geometry

result = validate(permits_gdf, [
    checks.valid_geometry(),
    checks.no_null_geometry(),
    checks.crs(4326),
    checks.no_nulls(["permit_id", "status"]),
    checks.unique("permit_id"),
    checks.domain("status", ["active", "inactive", "pending"]),
    checks.topology("no_overlaps"),
])

print(result.valid)       # bool
print(result.findings)    # DataFrame: check, field, feature_idx, message, severity
repaired = fix_geometry(permits_gdf)
```

**Depends:** geopandas, shapely, pandas  
**PyPI:** Yes (planned)

---

#### 6. `schema-enforcer` 📦

**Schema contract enforcement for spatial datasets.**

Define expected schemas as YAML, validate GeoDataFrames against them. CI-friendly. Same YAML format as the R and .NET companions — write once, enforce anywhere.

**Gap it fills:** `pandera` does DataFrame validation but doesn't understand CRS, geometry types, or spatial domains.

**Key functions:**
- `load_contract(path)` → SchemaContract
- `check_schema(gdf, contract)` → SchemaResult
- `generate_contract(gdf, path)` → writes YAML
- `SchemaContract.check(gdf)` → pass/fail with violations

```python
from schema_enforcer import load_contract, generate_contract

contract = load_contract("permits_schema.yml")
result = contract.check(permits_gdf)

if not result.valid:
    for v in result.violations:
        print(f"{v.field}: expected {v.expected}, got {v.actual}")

# Auto-generate from existing data
generate_contract(permits_gdf, "permits_schema.yml")
```

**Depends:** geopandas, pyyaml, pandas  
**PyPI:** Yes (planned)

---

#### 7. `geoinventory` 📦

**Scan, document, and diff any spatial data source.**

Scans GeoPackage, Shapefile, GeoParquet, PostGIS, STAC catalogs, COG — produces tidy DataFrames with schema, CRS, extent, and field profiling. Markdown/HTML reports.

**Gap it fills:** No Python package inventories an entire spatial data estate across formats with profiling, diffing, and reporting in one tool.

**Key functions:**
- `scan_source(path)` → InventoryResult (DataFrame)
- `scan_geoparquet(path)` → metadata without full read
- `scan_stac(url)` → collection inventory DataFrame
- `diff_inventory(old, new)` → schema diff DataFrame
- `report_inventory(inv, format="html")` → HTML/Markdown report

```python
from geoinventory import scan_source, diff_inventory, report_inventory

inv = scan_source("data/permits.gpkg")
print(inv.layers)     # DataFrame: name, geom_type, crs, row_count, extent
print(inv.fields)     # DataFrame: layer, field, dtype, null_count, unique_count

old_inv = scan_source("permits_2023.gpkg")
new_inv = scan_source("permits_2024.gpkg")
diff = diff_inventory(old_inv, new_inv)
report_inventory(new_inv, format="html", output="inventory_report.html")
```

**Depends:** geopandas, pyarrow, pystac-client, jinja2  
**PyPI:** Yes (planned)

---

#### 8. `geodata-health` 📦

**Data staleness and health monitoring for geodatabases.**

Scans spatial data sources for last-modified dates, record counts, and metadata completeness. Produces health reports with Active/Stale/Critical/Unknown classification.

**Gap it fills:** Enterprise GIS shops manually track dataset currency. No Python package standardizes staleness classification and health reporting for spatial data estates.

**Key functions:**
- `scan_health(path, thresholds)` → HealthReport
- `classify_staleness(days, thresholds)` → status string
- `health_report(report, format="html")` → rendered report
- `health_badge(report)` → summary string

```python
from geodata_health import scan_health, StalenessThresholds

thresholds = StalenessThresholds(active_days=30, stale_days=90, critical_days=180)
report = scan_health("data/enterprise.gdb", thresholds=thresholds)

print(report.summary)  # "Active: 12, Stale: 3, Critical: 1, Unknown: 0"
for layer in report.layers:
    print(f"{layer.name}: {layer.status} (last modified {layer.last_modified})")
```

**Depends:** geopandas, pandas, jinja2  
**PyPI:** Yes (planned)

---

### Tier 3: ETL & Pipelines

---

#### 9. `spatial-etl` 📦

**Declarative spatial ETL pipelines with audit trails.**

Build reproducible spatial data pipelines using a chainable API. Tracks row counts, timing, and validation results through every step. Each run produces a structured audit log.

**Gap it fills:** Python has great spatial tools but no pipeline framework providing structured audit trails for ETL. Teams write ad-hoc scripts; this makes them reproducible and auditable.

**Key functions:**
- `Pipeline(name)` — chainable: `.extract()`, `.transform()`, `.load()`
- `.extract(reader)` → Pipeline
- `.transform(*steps)` → Pipeline (reproject, clip, buffer, spatial_join, validate)
- `.load(writer)` → Pipeline
- `.run()` → PipelineResult (per-step metrics, row counts, timing)
- Built-in steps: `Reproject`, `Clip`, `Buffer`, `SpatialJoin`, `DropNulls`, `ValidateGeometry`, `RenameFields`

```python
from spatial_etl import Pipeline, steps, readers, writers

result = (
    Pipeline("permits-etl")
    .extract(readers.geoparquet("s3://bucket/raw/permits.parquet"))
    .transform(
        steps.Reproject(4326),
        steps.DropNulls(["permit_id"]),
        steps.ValidateGeometry(fix=True),
        steps.Clip(bbox=(-84.5, 38.0, -84.0, 38.5)),
    )
    .load(writers.geopackage("output/permits.gpkg", layer="permits"))
    .run()
)

print(result.summary)
# Step 1 extract:   2,400,000 rows in 3.2s
# Step 2 reproject: 2,400,000 rows in 1.1s
# Step 3 drop_nulls: 2,398,441 rows (-1,559) in 0.2s
# Step 4 validate:  2,398,441 rows (12 fixed) in 4.5s
# Step 5 clip:      48,213 rows (-2,350,228) in 0.8s
# Step 6 load:      48,213 rows written in 2.1s
```

**Depends:** geopandas, pyarrow, pandas  
**PyPI:** Yes (planned)

---

#### 10. `format-bridge` 📦

**One-function format conversion for spatial data.**

Convert between any GDAL/Fiona-readable format with CRS reprojection, encoding detection, and geometry validation. The ogr2ogr of Python with proper error handling and reporting.

**Gap it fills:** `geopandas.to_file()` works but doesn't handle encoding detection, validation, format-specific quirks, or produce conversion reports.

**Key functions:**
- `convert(input, output, crs=None, validate=True)` → ConversionResult
- `detect_encoding(path)` → str
- `validate_conversion(input, output)` → ComparisonResult
- `supported_formats()` → list

```python
from format_bridge import convert, detect_encoding

result = convert(
    "legacy_permits.shp",
    "permits.gpkg",
    crs=4326,
    validate=True
)
print(result.summary)
# Input:  2,400,000 rows, Shapefile, CP1252, EPSG:3089
# Output: 2,400,000 rows, GeoPackage, UTF-8, EPSG:4326
# Validation: row_count ✓  schema ✓  extent ✓
# Duration: 8.4s

encoding = detect_encoding("old_data.dbf")  # "CP1252"
```

**Depends:** geopandas, fiona, pyproj  
**PyPI:** Yes (planned)

---

#### 11. `oracle-spatial-etl` 📦

**Oracle Spatial / SDO_GEOMETRY ETL for Python.**

Read and write Oracle Spatial tables as GeoDataFrames. Handles SDO_GEOMETRY type conversion, SDE versioning, and enterprise geodatabase patterns. The missing bridge between cx_Oracle/python-oracledb and geopandas.

**Gap it fills:** No maintained Python package handles Oracle Spatial → GeoDataFrame with SDO_GEOMETRY type mapping, SDE versioning, and enterprise GDB patterns.

**Key functions:**
- `read_oracle_spatial(conn, table, bbox=None, where=None)` → GeoDataFrame
- `write_oracle_spatial(gdf, conn, table, if_exists="replace")` → None
- `list_spatial_tables(conn)` → pd.DataFrame
- `sdo_to_wkb(sdo_geom)` → bytes
- `wkb_to_sdo(wkb_bytes, srid)` → oracle SDO object

```python
from oracle_spatial_etl import read_oracle_spatial, write_oracle_spatial

import oracledb
conn = oracledb.connect(dsn="gis_db/gisuser@server/orcl")

permits = read_oracle_spatial(conn, "GIS.PERMITS", where="STATUS = 'Active'")
write_oracle_spatial(permits_updated, conn, "GIS.PERMITS_STAGING")
```

**Depends:** python-oracledb, geopandas, shapely  
**PyPI:** Yes (planned)

---

#### 12. `arcpy-etl-patterns` 📦

**Production-ready ArcPy ETL pattern library.**

Composable, tested ArcPy patterns for enterprise geodatabase ETL: truncate-and-load, upsert, delta detection, versioned editing, and domain validation. Each pattern is a reusable class with logging, error handling, and rollback.

**Gap it fills:** ArcPy patterns are scattered across tribal knowledge and Stack Exchange. No package standardizes the patterns that every ArcGIS shop implements differently.

**Key functions:**
- `TruncateAndLoad(target_fc, source_gdf)` → ETLResult
- `UpsertByKey(target_fc, source_gdf, key_field)` → ETLResult
- `DeltaDetect(old_fc, new_fc, key_field)` → DeltaResult (added, deleted, changed)
- `VersionedEdit(workspace, version)` → context manager
- `DomainValidator(workspace, domain_name)` → validates values against GDB domain

```python
from arcpy_etl_patterns import TruncateAndLoad, UpsertByKey, DeltaDetect

result = TruncateAndLoad(
    target_fc=r"C:\GIS\Enterprise.sde\GIS.PERMITS",
    source_gdf=new_permits_gdf,
    log_path="logs/etl_run.log"
).run()

delta = DeltaDetect(
    old_fc=r"C:\GIS\Archive.gdb\PERMITS_2023",
    new_fc=r"C:\GIS\Enterprise.sde\GIS.PERMITS",
    key_field="PERMIT_ID"
)
print(f"Added: {len(delta.added)}, Deleted: {len(delta.deleted)}, Changed: {len(delta.changed)}")
```

**Depends:** arcpy (ArcGIS Pro), geopandas  
**PyPI:** Yes (planned, ArcGIS Pro Python env only)

---

### Tier 4: LLM / GeoAI

---

#### 13. `geoai-toolkit` 📦

**LLM utility functions for geospatial workflows.**

Coordinate parsing, GeoDataFrame description for LLM context windows, spatial prompt templates, and LangChain/LlamaIndex tool integrations.

**Gap it fills:** No Python package standardizes the patterns for bridging LLMs with spatial data — coordinate parsing, dataset description, spatial prompt engineering.

**Key functions:**
- `parse_coordinates(text)` → dict (lat, lon, format, confidence)
- `describe_geodataframe(gdf, max_tokens=500)` → str
- `dms_to_decimal(deg, min, sec, direction)` → float
- `decimal_to_dms(decimal, axis)` → tuple
- `SpatialTool` — LangChain BaseTool subclass
- `geo_prompt(template, **kwargs)` → str

```python
from geoai_toolkit import parse_coordinates, describe_geodataframe, SpatialTool

coords = parse_coordinates("38°02'53\"N 84°30'04\"W")
# {"lat": 38.048, "lon": -84.501, "format": "DMS", "confidence": 0.99}

description = describe_geodataframe(permits_gdf)
# "GeoDataFrame: 1,247 Polygon features, EPSG:4326,
#  Bounds: (-84.50, 38.00, -84.30, 38.20),
#  Columns: permit_id (str, 1247 unique), status (str, 3 unique: active, inactive, pending)..."

# LangChain integration
from langchain.agents import initialize_agent
tools = [SpatialTool(gdf=permits_gdf)]
```

**Depends:** geopandas, pandas  
**Suggests:** langchain, llama-index  
**PyPI:** Yes (planned)

---

### Tier 5: Visualization & Cartography

---

#### 14. `georender` 📦

**Publication-ready static map rendering in Python.**

Opinionated matplotlib/cartopy wrapper for cartographic output. `render_map()` is the one-liner that adds scale bars, north arrows, inset maps, legends, and accessible palettes. Static maps only — print-ready at 300 DPI.

**Gap it fills:** matplotlib + cartopy requires significant boilerplate for cartographic elements. No Python package provides a one-liner publication map with proper cartographic conventions.

**Key functions:**
- `render_map(gdf, fill, title, basemap, palette, scalebar, north_arrow, inset, output, dpi)` → Figure
- `render_panels(gdfs_dict, fill, shared_legend, ncol, output)` → Figure
- `render_inset(gdf, region="state")` → Figure
- `add_scalebar(ax)`, `add_north_arrow(ax)`, `add_title_block(ax, title, source)`

```python
from georender import render_map, render_panels

render_map(
    permits_gdf,
    fill="STATUS",
    title="Active Mining Permits — Pike County, KY",
    basemap="terrain",
    palette="karto_environmental",
    scalebar=True,
    north_arrow=True,
    inset="state",
    output="permits_map.png",
    dpi=300
)
```

**Depends:** matplotlib, cartopy, geopandas  
**Suggests:** contextily (basemap tiles), kartocolors  
**PyPI:** Yes (planned)

---

#### 15. `kartocolors` 📦

**Curated cartographic color palettes with built-in accessibility.**

A palette package specifically designed for cartographic use. Every palette is pre-tested for WCAG 2.1 AA compliance and colorblind safety. Includes sequential, diverging, and qualitative schemes. Integrates with matplotlib, folium, and lonboard.

**Gap it fills:** matplotlib colormaps aren't all colorblind-safe. No Python package provides cartography-specific palettes with built-in accessibility guarantees.

**Key functions:**
- `karto_pal(name, n)` → list of hex strings
- `karto_cmap(name, n)` → matplotlib ListedColormap
- `karto_palettes()` → pd.DataFrame of all palettes with accessibility scores
- Palette categories: terrain, landuse, hydro, population, environmental, diverging, sequential

```python
from kartocolors import karto_pal, karto_cmap, karto_palettes

colors = karto_pal("terrain", 7)
cmap = karto_cmap("environmental", 5)

import matplotlib.pyplot as plt
permits_gdf.plot(column="STATUS", cmap=cmap)
plt.show()

print(karto_palettes())
# DataFrame: name, type, wcag_score, cvd_safe, min_n, max_n
```

**Depends:** matplotlib  
**PyPI:** Yes (planned)

---

#### 16. `map-accessibility` 📦

**WCAG 2.1 AA color accessibility for cartographic palettes.**

Check color palettes for contrast ratios, simulate colorblind vision, and score palette accessibility. Works with matplotlib and any hex-color-based workflow.

**Gap it fills:** `accessible-pygments` exists for code, but no Python package provides WCAG contrast checking + CVD simulation specifically for cartographic use.

**Key functions:**
- `contrast_ratio(fg, bg)` → float
- `check_wcag(fg, bg, level="AA", size="normal")` → bool
- `simulate_cvd(colors, cvd_type)` → list of hex strings
- `score_palette(colors)` → dict (overall, contrast_score, cvd_score)
- `pal_accessible(n)` → list of accessible hex colors

```python
from map_accessibility import contrast_ratio, check_wcag, simulate_cvd, score_palette

ratio = contrast_ratio("#1B4F72", "#FFFFFF")  # 10.5
passes = check_wcag("#1B4F72", "#FFFFFF", level="AA")  # True

simulated = simulate_cvd(["#FF0000", "#00FF00"], "deuteranopia")
score = score_palette(["#1B4F72", "#2E86C1", "#AED6F1"])
# {"overall": 0.87, "contrast_score": 0.92, "cvd_score": 0.82}
```

**Depends:** (none — pure Python)  
**PyPI:** Yes (planned)

---

### Tier 6: Vector Tile Tooling

---

#### 17. `mvtutils` 📦

**Mapbox Vector Tile (MVT) utilities for Python.**

Encode and decode MVT tiles, inspect tile contents, batch-process tile directories, and convert between MVT and GeoDataFrame. Fills the gap between mapbox-vector-tile (low-level) and practical use.

**Gap it fills:** `mapbox-vector-tile` is the only Python MVT library and it's low-level. No package provides tidy encode/decode with GeoDataFrame integration and tile directory management.

**Key functions:**
- `decode_tile(mvt_bytes, layer=None)` → GeoDataFrame
- `encode_tile(gdf, layer_name, extent=4096)` → bytes
- `inspect_tile(mvt_bytes)` → dict (layers, feature counts, geometry types)
- `tile_to_geojson(mvt_bytes)` → GeoJSON dict
- `batch_decode(tile_dir, zoom)` → GeoDataFrame (merged)

```python
from mvtutils import decode_tile, encode_tile, inspect_tile

with open("14_4523_6234.mvt", "rb") as f:
    data = f.read()

info = inspect_tile(data)
# {"layers": ["buildings", "roads"], "buildings": {"count": 347, "geom_type": "Polygon"}}

gdf = decode_tile(data, layer="buildings")
encoded = encode_tile(gdf, layer_name="buildings")
```

**Depends:** mapbox-vector-tile, geopandas, shapely  
**PyPI:** Yes (planned)

---

#### 18. `tilesystem` 📦

**Tile math and XYZ/TMS/quadkey utilities for Python.**

Bounding box to tile index conversion, tile coordinate systems (XYZ, TMS, quadkey, Bing), zoom level calculations, and tile grid generation. Zero-dependency utility library.

**Gap it fills:** `mercantile` exists but is unmaintained (last release 2021). No modern, typed, actively-maintained Python tile math library exists.

**Key functions:**
- `bbox_to_tiles(bbox, zoom)` → list of Tile(x, y, z)
- `tile_to_bbox(x, y, z)` → BoundingBox
- `xy_to_quadkey(x, y, z)` → str
- `quadkey_to_xy(quadkey)` → Tile
- `tiles_for_zoom(bbox, zoom)` → iterator of Tile
- `tile_grid(bbox, zoom)` → GeoDataFrame of tile polygons

```python
from tilesystem import bbox_to_tiles, tile_to_bbox, tile_grid

tiles = bbox_to_tiles((-84.5, 38.0, -84.3, 38.2), zoom=14)
# [Tile(x=4523, y=6234, z=14), Tile(x=4524, y=6234, z=14), ...]

bbox = tile_to_bbox(4523, 6234, 14)
# BoundingBox(west=-84.506, south=38.014, east=-84.484, north=38.031)

grid_gdf = tile_grid((-84.5, 38.0, -84.3, 38.2), zoom=12)
# GeoDataFrame of tile polygons with x, y, z columns
```

**Depends:** (none — pure Python)  
**Suggests:** geopandas (for tile_grid)  
**PyPI:** Yes (planned)

---

### Tier 7: Developer Tools

---

#### 19. `fgdc-metadata` 📦

**Create, validate, and score FGDC metadata records.**

Generate FGDC-compliant XML metadata for GeoDataFrames and file-based datasets. Validate existing records against the FGDC Content Standard. Score completeness. Designed for US government GIS workflows.

**Gap it fills:** FGDC metadata tooling in Python is essentially nonexistent. ArcGIS Pro can generate FGDC metadata but there's no Python-native library for creating, validating, or scoring FGDC records programmatically.

**Key functions:**
- `create_metadata(gdf, title, abstract, contact)` → FGDCRecord
- `validate_metadata(xml_path)` → ValidationResult
- `score_metadata(xml_path)` → float (0-1 completeness score)
- `FGDCRecord.to_xml(path)`, `FGDCRecord.to_dict()`
- `template_metadata(template="basic")` → FGDCRecord stub

```python
from fgdc_metadata import create_metadata, validate_metadata, score_metadata

record = create_metadata(
    gdf=permits_gdf,
    title="Kentucky Mining Permits 2024",
    abstract="Active surface and underground mining permits...",
    contact={"name": "Chris Lyons", "org": "KY Energy & Environment Cabinet"},
    purpose="Regulatory permit tracking",
    keywords=["mining", "permits", "Kentucky", "regulatory"]
)
record.to_xml("permits_metadata.xml")

score = score_metadata("permits_metadata.xml")  # 0.87
result = validate_metadata("permits_metadata.xml")
```

**Depends:** lxml, geopandas  
**PyPI:** Yes (planned)

---

#### 20. `gis-project-scaffold` 📦

**Project scaffolding CLI for GIS/spatial Python projects.**

Generates opinionated project structures for spatial Python projects with CLAUDE.md, ai-dev/ folder, GitHub Actions for spatial testing, and pre-configured linting for ArcPy/geopandas codebases.

**Gap it fills:** No spatial-aware project scaffolding tool exists. `cookiecutter-data-science` is generic; this understands GIS project conventions (SDE connections, projection files, data directories, FGDC metadata stubs).

**Key functions (CLI):**
- `gis-scaffold new <project-name> --template [arcpy|geopandas|cloud-native]`
- `gis-scaffold add-agent <agent-type>` — adds ai-dev/agents/ config
- `gis-scaffold add-decision <title>` — creates numbered DL-NNN file
- `gis-scaffold check` — validates scaffold completeness

```bash
# CLI usage
gis-scaffold new my-permit-etl --template cloud-native
gis-scaffold add-agent arcpy-expert
gis-scaffold add-decision "use-geoparquet-over-shapefile"
gis-scaffold check
# ✓ CLAUDE.md present
# ✓ ai-dev/architecture.md present
# ✓ ai-dev/guardrails/ present
# ✗ Missing: ai-dev/spec.md
```

**Depends:** click, jinja2, pyyaml  
**PyPI:** Yes (planned)

---

## Cross-Ecosystem Companion Matrix

| Domain | Python | R | NuGet | npm |
|--------|--------|---|-------|-----|
| Cloud-native I/O | `cloudgeo` | `cloudgeo` (R) | — | — |
| Overture Maps | `overturer` | `overturer` | — | `@chrislyonsKY/overture-query` |
| PMTiles | `pmtilespy` | `pmtilesr` | — | `@chrislyonsKY/pmtiles-utils` |
| STAC workflows | `stacpipe` | `stacr` | — | `@chrislyonsKY/stac-client` |
| Validation | `spatial-validate` | `spatialvalidate` | `SpatialValidate` | `@chrislyonsKY/geojson-validate` |
| Schema enforcement | `schema-enforcer` | `schemaenforcer` | `SchemaEnforce` | `@chrislyonsKY/geojson-schema` |
| Data inventory | `geoinventory` | `geoinventoryr` | `GeoInventory` | — |
| Data health | `geodata-health` | `geodatahealth` | `GeoDataHealth` | — |
| ETL pipelines | `spatial-etl` | `spatialetl` | — | — |
| Format conversion | `format-bridge` | `formatbridge` | `FormatBridge` | — |
| Oracle Spatial | `oracle-spatial-etl` | — | — | — |
| ArcPy patterns | `arcpy-etl-patterns` | — | — | — |
| GeoAI/LLM | `geoai-toolkit` | `geoaitools` | `GeoAI` | `@chrislyonsKY/geo-llm` |
| Static maps | `georender` | `georender` | — | — |
| Carto palettes | `kartocolors` | `kartocolors` | `KartoColors` | `@chrislyonsKY/karto-palettes` |
| Color accessibility | `map-accessibility` | `mapaccessibility` | `MapAccessibility` | `@chrislyonsKY/map-a11y` |
| MVT utilities | `mvtutils` | — | — | — |
| Tile math | `tilesystem` | — | — | — |
| FGDC metadata | `fgdc-metadata` | — | — | — |
| Project scaffold | `gis-project-scaffold` | — | — | — |

## Build Priority

| Phase | Packages | Rationale |
|-------|----------|-----------|
| **1** | `map-accessibility`, `kartocolors`, `tilesystem` | Zero/minimal dependencies, pure Python, high value |
| **2** | `geoai-toolkit`, `spatial-validate`, `schema-enforcer` | Unique gaps, direct cross-ecosystem companions |
| **3** | `geoinventory`, `geodata-health`, `fgdc-metadata` | Data documentation tooling |
| **4** | `cloudgeo`, `overturer`, `stacpipe`, `pmtilespy` | Cloud-native I/O, higher dependency count |
| **5** | `spatial-etl`, `format-bridge`, `mvtutils`, `georender` | ETL and visualization infrastructure |
| **6** | `oracle-spatial-etl`, `arcpy-etl-patterns`, `gis-project-scaffold` | Domain-specific, enterprise-focused |
