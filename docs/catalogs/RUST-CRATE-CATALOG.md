# Chris Lyons — Rust Crate Catalog

> 20 open-source Rust crates for high-performance, cloud-native geospatial workflows  
> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)

---

## Philosophy

The Rust geospatial ecosystem (`geo`, `geo-types`, `proj`, `geozero`) provides solid geometry primitives but has significant gaps in cloud-native I/O, practical developer tooling, and WASM-ready utilities. These crates fill those gaps — high-performance algorithms, pure-Rust cloud-native readers, CLI tools that practitioners actually use, and WASM builds for browser deployment. All crates are `no_std`-compatible where possible, and WASM targets are first-class.

---

## Ecosystem Landscape

| Crate | What it does | Gap |
|-------|-------------|-----|
| **geo** | Core geometry types and algorithms | Foundational — but no cloud I/O, no tile math |
| **geo-types** | Geometry primitives | Low-level only |
| **geozero** | Zero-copy geo format processing | Excellent for streaming but no tidy high-level API |
| **proj** | Coordinate reference system transforms | Binds to PROJ C library — heavy dep |
| **gdal** | GDAL bindings | C bindings — hard to cross-compile, no WASM |
| **flatgeobuf** | FlatGeobuf format | Format-specific only |
| **pmtiles** | PMTiles (Rust) | Minimal — no spatial query API |
| **object_store** | Cloud object storage | Generic — no geo-aware abstractions |
| **polars** | DataFrame library | No geospatial extension |
| **duckdb (Rust)** | DuckDB bindings | No spatial convenience layer |

---

## Crates (20)

---

### Tier 1: High-Performance Geometry Algorithms

---

#### 1. `tissot` 🦀

**Projection distortion analysis and spatial diagnostics engine.**

Computes per-feature area distortion, angular distortion, and Tissot indicatrix ellipses for any CRS. Recommends better projections with quantified improvement. Powers the `tissot` QGIS plugin and Python CLI via PyO3 bindings.

**Gap it fills:** No Rust crate (or any open-source tool) quantifies per-feature projection distortion and produces CRS recommendations. GIS practitioners choose projections by convention, not by data.

**Key types:**
- `DistortionReport` — per-feature area scale, angular distortion, Tissot ellipse params
- `CrsRecommendation` — candidate CRS with distortion improvement metrics
- `SpatialDiff` — feature-level change classification (added, deleted, moved, reshaped)
- `QualityScore` — 0-100 score across CRS, data quality, and format dimensions

```rust
use tissot::{DistortionAnalyzer, CrsAdvisor};

let analyzer = DistortionAnalyzer::new(4326); // source CRS
let report = analyzer.analyze(&geometries)?;

println!("Mean area distortion: {:.2}%", report.mean_area_distortion * 100.0);
println!("Max angular distortion: {:.2}°", report.max_angular_distortion);

let recommendations = CrsAdvisor::recommend(&geometries, &report);
for rec in &recommendations {
    println!("{}: {:.1}% improvement", rec.epsg, rec.distortion_improvement * 100.0);
}
```

**Depends:** geo, geo-types, proj  
**PyO3 bindings:** Yes (Python CLI and QGIS plugin)  
**crates.io:** Yes (published)

---

#### 2. `geo-algorithms` 🦀

**Extended geometry algorithms beyond the `geo` crate.**

Hausdorff distance, Fréchet distance, polygon skeletonization, minimum bounding rectangle, convex hull variants, and shape similarity metrics. All algorithms operate on `geo-types` primitives.

**Gap it fills:** The `geo` crate covers common algorithms but lacks advanced shape analysis algorithms that practitioners need for change detection, generalization, and shape matching.

**Key functions:**
- `hausdorff_distance(a: &Geometry, b: &Geometry) -> f64`
- `frechet_distance(a: &LineString, b: &LineString) -> f64`
- `minimum_bounding_rectangle(geom: &Geometry) -> Polygon`
- `skeleton(polygon: &Polygon, tolerance: f64) -> MultiLineString`
- `shape_similarity(a: &Geometry, b: &Geometry) -> f64` (0.0–1.0)
- `oriented_envelope(geom: &Geometry) -> Polygon`

```rust
use geo_algorithms::{hausdorff_distance, shape_similarity, minimum_bounding_rectangle};

let dist = hausdorff_distance(&polygon_2020, &polygon_2024);
let sim = shape_similarity(&parcel_a, &parcel_b); // 0.94 — nearly identical
let mbr = minimum_bounding_rectangle(&irregular_polygon);
```

**Depends:** geo, geo-types  
**crates.io:** Yes (planned)

---

#### 3. `spatial-index` 🦀

**High-performance spatial indexing with a clean query API.**

R-tree, k-d tree, and grid-based spatial indexes with a unified query interface. Designed for in-memory indexes over large feature collections with thread-safe query access.

**Gap it fills:** `rstar` provides R-tree functionality but requires implementing the `RstarObject` trait. No crate provides a plug-and-play spatial index that works directly with `geo-types` geometries.

**Key types:**
- `SpatialIndex<T>` — generic index over any `geo-types` geometry
- `IndexType::RTree | KDTree | Grid`
- `.query_bbox(bbox)` → `Vec<&T>`
- `.query_nearest(point, k)` → `Vec<(&T, f64)>`
- `.query_within(geom, predicate)` → `Vec<&T>`

```rust
use spatial_index::{SpatialIndex, IndexType};

let index = SpatialIndex::build(&features, IndexType::RTree)?;

let results = index.query_bbox(&bbox);
let nearest = index.query_nearest(&point, 5); // 5 nearest features
let intersecting = index.query_within(&search_polygon, |a, b| a.intersects(b));
```

**Depends:** geo-types, rstar  
**crates.io:** Yes (planned)

---

#### 4. `coord-parse` 🦀

**Coordinate string parsing for every format humans write.**

Parses DMS, DDM, decimal degrees, UTM, MGRS, USNG, and GEOREF from freeform strings. Returns typed `Coordinate` with confidence score. WASM-compatible — zero C dependencies.

**Gap it fills:** No Rust crate handles the full range of coordinate string formats that appear in regulatory documents, field notes, and user input. `proj` does transforms but not parsing.

**Key functions:**
- `parse(input: &str) -> Result<ParsedCoordinate>`
- `parse_dms(input: &str) -> Result<Coordinate>`
- `parse_utm(input: &str) -> Result<Coordinate>`
- `parse_mgrs(input: &str) -> Result<Coordinate>`
- `dms_to_decimal(deg: f64, min: f64, sec: f64, dir: Direction) -> f64`
- `decimal_to_dms(decimal: f64, axis: Axis) -> DmsCoordinate`

```rust
use coord_parse::parse;

let coord = parse("38°02'53\"N 84°30'04\"W")?;
// ParsedCoordinate { lat: 38.048, lon: -84.501, format: CoordFormat::DMS, confidence: 0.99 }

let coord2 = parse("16S 722714 4213456")?;
// ParsedCoordinate { lat: 38.048, lon: -84.501, format: CoordFormat::UTM, zone: "16S" }
```

**Depends:** (none — pure Rust)  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

### Tier 2: Cloud-Native I/O

---

#### 5. `cog-reader` 🦀

**Pure Rust Cloud Optimized GeoTIFF reader.**

Reads COG files from HTTP (range requests), S3, and local paths. Returns raster tiles as ndarray arrays. No GDAL dependency — pure Rust implementation of the COG/GeoTIFF spec.

**Gap it fills:** The only Rust GeoTIFF option is GDAL bindings. No pure-Rust COG reader exists. This enables COG reading in WASM and cross-compiled targets where GDAL is unavailable.

**Key types:**
- `CogReader::open(source: impl Source) -> Result<CogReader>`
- `.overview(level: usize) -> Result<Array2<f32>>`
- `.tile(x: u64, y: u64) -> Result<Array2<f32>>`
- `.window(bbox: &BoundingBox, overview: usize) -> Result<Array2<f32>>`
- `.info() -> CogInfo` (CRS, extent, overviews, nodata, dtype)
- `Source` trait: `HttpSource`, `S3Source`, `FileSource`

```rust
use cog_reader::{CogReader, HttpSource};

let source = HttpSource::new("https://data.example.gov/dem.tif");
let reader = CogReader::open(source).await?;

let info = reader.info();
println!("CRS: {}, Size: {}x{}", info.crs, info.width, info.height);

let window = reader.window(&bbox, overview=2).await?;
// Returns Array2<f32> for the requested region at overview level 2
```

**Depends:** tokio, bytes, ndarray  
**Async:** Yes (tokio)  
**crates.io:** Yes (planned)

---

#### 6. `geoparquet-rs` 🦀

**Pure Rust GeoParquet reader/writer.**

Read and write GeoParquet files with geometry column decoding. Builds on `parquet2` or `arrow2` — no Python or GDAL required. Returns `geo-types` geometries alongside Arrow record batches.

**Gap it fills:** `geoarrow-rs` is in development but GeoParquet-specific tooling in pure Rust with a simple API doesn't exist. Rust applications reading Overture Maps or cloud GeoParquet have no clean option.

**Key types:**
- `GeoParquetReader::open(path: impl AsRef<Path>) -> Result<Self>`
- `.schema() -> Schema`
- `.bbox_filter(bbox: &BoundingBox) -> Self` (pushdown)
- `.read_all() -> Result<GeoDataFrame>`
- `GeoParquetWriter::new(path, schema) -> Result<Self>`
- `.write_batch(batch: &RecordBatch, geometries: &[Geometry]) -> Result<()>`

```rust
use geoparquet_rs::{GeoParquetReader, BoundingBox};

let bbox = BoundingBox::new(-84.5, 38.0, -84.3, 38.2);
let reader = GeoParquetReader::open("buildings.parquet")?
    .bbox_filter(&bbox);

let frame = reader.read_all()?;
println!("{} features", frame.len());

for (geom, props) in frame.iter() {
    // geo-types Geometry + Arrow row
}
```

**Depends:** parquet2 or arrow2, geo-types, wkb  
**crates.io:** Yes (planned)

---

#### 7. `pmtiles-rs` 🦀

**Full-featured PMTiles reader/writer in pure Rust.**

Read PMTiles archives via HTTP range requests, S3, or local file. Decode MVT tiles to `geo-types` geometries. Write PMTiles from feature collections. WASM-compatible for browser-side tile reading.

**Gap it fills:** The existing `pmtiles` Rust crate is minimal — header reading only. No crate decodes MVT content or writes PMTiles archives.

**Key types:**
- `PmtilesReader::open(source: impl Source) -> Result<Self>`
- `.info() -> PmtilesInfo` (zoom range, bounds, tile count, format)
- `.get_tile(z: u8, x: u64, y: u64) -> Result<Vec<u8>>`
- `.query_bbox(bbox: &BoundingBox, zoom: u8) -> Result<Vec<DecodedTile>>`
- `PmtilesWriter::new(path) -> Result<Self>`
- `.add_tile(z, x, y, data: &[u8]) -> Result<()>`
- `.finish() -> Result<()>`

```rust
use pmtiles_rs::{PmtilesReader, HttpSource};

let reader = PmtilesReader::open(HttpSource::new("buildings.pmtiles")).await?;
let info = reader.info();
println!("Zoom: {}-{}, Tiles: {}", info.min_zoom, info.max_zoom, info.tile_count);

let tiles = reader.query_bbox(&bbox, zoom=14).await?;
for tile in &tiles {
    for feature in &tile.features {
        // geo-types Geometry + properties HashMap
    }
}
```

**Depends:** tokio, bytes, geo-types  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

#### 8. `flatgeobuf-stream` 🦀

**Streaming FlatGeobuf reader with spatial index utilization.**

Builds on the `flatgeobuf` crate with a higher-level API: bbox filtering, async HTTP streaming, and `geo-types` output. Designed for cloud-hosted FGB files.

**Gap it fills:** The `flatgeobuf` crate is low-level. No crate provides a clean async streaming API with bbox filtering for cloud-hosted FlatGeobuf that returns `geo-types` directly.

**Key types:**
- `FgbReader::open(source: impl Source) -> Result<Self>`
- `.bbox_filter(bbox: &BoundingBox) -> Self`
- `.stream() -> impl Stream<Item = Result<Feature>>`
- `.collect_gdf() -> Result<Vec<(Geometry, Properties)>>`

```rust
use flatgeobuf_stream::{FgbReader, HttpSource};
use futures::StreamExt;

let reader = FgbReader::open(HttpSource::new("https://example.com/permits.fgb"))
    .await?
    .bbox_filter(&bbox);

let mut stream = reader.stream();
while let Some(feature) = stream.next().await {
    let (geom, props) = feature?;
    // process geo-types Geometry
}
```

**Depends:** flatgeobuf, tokio, geo-types, futures  
**crates.io:** Yes (planned)

---

### Tier 3: Spatial Indexing & Query

---

#### 9. `tile-math` 🦀

**Tile coordinate math for XYZ, TMS, quadkey, and MGRS systems.**

Bounding box ↔ tile conversions, zoom level calculations, tile grid generation, and coordinate system translations. Zero dependencies — pure Rust. WASM-first design.

**Gap it fills:** No actively maintained Rust crate covers the full range of tile coordinate systems (XYZ, TMS, quadkey, Bing Maps) with a clean API. `tilejson` handles metadata; nothing handles math.

**Key functions:**
- `bbox_to_tiles(bbox: &BoundingBox, zoom: u8) -> Vec<Tile>`
- `tile_to_bbox(tile: &Tile) -> BoundingBox`
- `tile_to_quadkey(tile: &Tile) -> String`
- `quadkey_to_tile(quadkey: &str) -> Result<Tile>`
- `zoom_for_resolution(meters_per_pixel: f64, latitude: f64) -> u8`
- `tiles_in_bbox_count(bbox: &BoundingBox, zoom: u8) -> u64`

```rust
use tile_math::{bbox_to_tiles, tile_to_bbox, BoundingBox};

let bbox = BoundingBox::new(-84.5, 38.0, -84.3, 38.2);
let tiles = bbox_to_tiles(&bbox, 14);
println!("{} tiles at zoom 14", tiles.len());

for tile in &tiles {
    let tile_bbox = tile_to_bbox(tile);
    println!("z={} x={} y={} → {:?}", tile.z, tile.x, tile.y, tile_bbox);
}
```

**Depends:** (none — pure Rust)  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

#### 10. `geo-query` 🦀

**Spatial query engine over in-memory feature collections.**

Combines `spatial-index` with predicate evaluation: within, contains, intersects, touches, crosses, overlaps, dwithin. Returns typed query results with optional attribute filtering.

**Gap it fills:** Performing spatial queries on in-memory feature collections in Rust requires assembling an R-tree, writing predicate closures, and managing lifetimes. This provides a query engine abstraction.

**Key types:**
- `FeatureCollection<T>` — indexed collection with attribute access
- `.query(SpatialPredicate, Option<AttributeFilter>) -> QueryResult<T>`
- `SpatialPredicate`: `Within`, `Contains`, `Intersects`, `DWithin(f64)`, `Nearest(usize)`
- `QueryResult<T>` — iterator over matching features

```rust
use geo_query::{FeatureCollection, SpatialPredicate};

let fc = FeatureCollection::from_geodataframe(features)?;
fc.build_index()?;

let results = fc.query(
    SpatialPredicate::DWithin { geom: &point, distance_m: 500.0 },
    Some(|props| props["status"] == "active")
);

for feature in results {
    println!("{}: {}", feature.id, feature.geometry.wkt());
}
```

**Depends:** geo-types, spatial-index (this org), geo  
**crates.io:** Yes (planned)

---

### Tier 4: Raster Processing

---

#### 11. `raster-ops` 🦀

**Raster algebra and change detection operations in pure Rust.**

Per-pixel arithmetic, focal statistics, zonal statistics, and raster change detection. Operates on `ndarray` arrays. No GDAL required.

**Gap it fills:** Rust has no raster algebra library. Operations that users do in GDAL `calc`, numpy, or terra require rolling their own in Rust. This provides the common operations as a library.

**Key functions:**
- `difference(a: &Array2<f32>, b: &Array2<f32>) -> Array2<f32>`
- `ratio(a: &Array2<f32>, b: &Array2<f32>) -> Array2<f32>`
- `focal_mean(raster: &Array2<f32>, radius: usize) -> Array2<f32>`
- `zonal_stats(raster: &Array2<f32>, zones: &Array2<u32>) -> HashMap<u32, ZonalResult>`
- `threshold_change(diff: &Array2<f32>, threshold: f32) -> Array2<u8>`
- `resample(raster: &Array2<f32>, target_shape: (usize, usize), method: ResampleMethod) -> Array2<f32>`

```rust
use raster_ops::{difference, threshold_change, zonal_stats};

let ndvi_2020: Array2<f32> = /* ... */;
let ndvi_2024: Array2<f32> = /* ... */;

let change = difference(&ndvi_2024, &ndvi_2020);
let significant = threshold_change(&change, 0.2);
let stats = zonal_stats(&change, &watershed_zones);
```

**Depends:** ndarray  
**crates.io:** Yes (planned)

---

#### 12. `terrain-rs` 🦀

**Terrain analysis from elevation rasters in pure Rust.**

Slope, aspect, hillshade, curvature, and viewshed calculations from DEM arrays. Designed for COG-sourced DEMs with tiled processing to avoid loading full rasters into memory.

**Gap it fills:** No Rust crate provides terrain analysis algorithms. GDAL bindings expose `gdaldem` but require GDAL. This enables terrain analysis in WASM and minimal-dependency environments.

**Key functions:**
- `slope(dem: &Array2<f32>, cell_size: f64) -> Array2<f32>`
- `aspect(dem: &Array2<f32>, cell_size: f64) -> Array2<f32>`
- `hillshade(dem: &Array2<f32>, cell_size: f64, azimuth: f64, altitude: f64) -> Array2<f32>`
- `curvature(dem: &Array2<f32>, cell_size: f64) -> CurvatureResult`
- `viewshed(dem: &Array2<f32>, observer: &Point, observer_height: f64) -> Array2<bool>`

```rust
use terrain_rs::{slope, hillshade, viewshed};

let dem: Array2<f32> = cog_reader.window(&bbox, overview=0).await?;
let slope_map = slope(&dem, cell_size=1.0); // degrees
let shade = hillshade(&dem, cell_size=1.0, azimuth=315.0, altitude=45.0);
```

**Depends:** ndarray  
**crates.io:** Yes (planned)

---

### Tier 5: Vector Tile Encoding/Decoding

---

#### 13. `mvt-rs` 🦀

**Mapbox Vector Tile encoding and decoding in pure Rust.**

Encode `geo-types` geometries to MVT protobuf bytes. Decode MVT tiles back to `geo-types`. Handles coordinate quantization, clipping, and layer management. WASM-compatible.

**Gap it fills:** `vector-tile` and `mvt` crates exist but are unmaintained or have incomplete APIs. No crate provides clean round-trip MVT encoding with `geo-types` integration.

**Key functions:**
- `encode(layers: &[Layer]) -> Result<Vec<u8>>`
- `decode(bytes: &[u8]) -> Result<Tile>`
- `Layer::new(name, extent)` → builder with `.add_feature(geometry, properties)`
- `quantize(geometry: &Geometry, extent: u32, bbox: &BoundingBox) -> QuantizedGeometry`
- `clip_to_tile(geometry: &Geometry, tile: &TileCoord) -> Option<Geometry>`

```rust
use mvt_rs::{encode, decode, Layer};

// Encode
let mut layer = Layer::new("buildings", 4096);
for (geom, props) in &features {
    layer.add_feature(geom, props)?;
}
let tile_bytes = encode(&[layer])?;

// Decode
let tile = decode(&tile_bytes)?;
for layer in &tile.layers {
    for feature in &layer.features {
        let geom: geo_types::Geometry<f64> = feature.geometry()?;
    }
}
```

**Depends:** prost (protobuf), geo-types  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

#### 14. `tippecanoe-rs` 🦀

**Tile pyramid generation from feature collections in Rust.**

Generate vector tile pyramids (PMTiles or tile directories) from large feature collections. Implements feature dropping, simplification, and attribute selection per zoom level. The Rust alternative to the tippecanoe CLI.

**Gap it fills:** `tippecanoe` is a C++ CLI. No Rust library provides programmatic tile pyramid generation that can be embedded in applications or called from Rust-based pipelines.

**Key types:**
- `TilesetBuilder::new()` → builder
- `.add_layer(name, features, options)` → Self
- `.zoom_range(min: u8, max: u8)` → Self
- `.simplification(level: SimplificationLevel)` → Self
- `.build_pmtiles(output: &Path) -> Result<PmtilesStats>`
- `.build_directory(output: &Path) -> Result<TileStats>`

```rust
use tippecanoe_rs::{TilesetBuilder, SimplificationLevel};

let stats = TilesetBuilder::new()
    .add_layer("buildings", &building_features, Default::default())
    .add_layer("roads", &road_features, Default::default())
    .zoom_range(0, 14)
    .simplification(SimplificationLevel::Moderate)
    .build_pmtiles("output.pmtiles")?;

println!("Generated {} tiles across {} zoom levels", stats.tile_count, stats.zoom_levels);
```

**Depends:** mvt-rs (this org), pmtiles-rs (this org), geo  
**crates.io:** Yes (planned)

---

### Tier 6: CLI Tools

---

#### 15. `geolint` 🦀

**CLI spatial data linter and validator.**

Command-line tool for validating spatial data files. Checks geometry validity, CRS, schema, topology, and file format best practices. Returns machine-readable JSON or human-readable reports. CI-ready exit codes.

**Gap it fills:** No dedicated spatial data linting CLI exists. `ogr2ogr` can check validity but not schema contracts, topology rules, or format recommendations.

**Key commands:**
- `geolint check <file>` — full validation report
- `geolint schema <file> --contract schema.yml` — validate against YAML contract
- `geolint fix <file> --output fixed.gpkg` — fix geometry issues
- `geolint info <file>` — metadata and field profile
- `geolint diff <before> <after>` — detect changes between versions

```bash
geolint check permits.gpkg
# ✓ Geometry: all valid (2,400,000 features)
# ✓ CRS: EPSG:4326 detected
# ⚠ Topology: 3 overlapping polygons (features 12, 847, 1203)
# ✗ Schema: field 'PERMIT_ID' has 12 null values (required)
# Exit code: 1

geolint check permits.gpkg --format json | jq '.findings'
```

**Depends:** geo-types, geoparquet-rs (this org), clap  
**Binary:** Yes (crates.io installable via `cargo install geolint`)  
**crates.io:** Yes (planned)

---

#### 16. `geodiff-cli` 🦀

**CLI for detecting and reporting spatial data changes.**

Compares two versions of a spatial dataset and produces a change report: added, deleted, moved, reshaped, attribute-changed. Outputs GeoJSON, HTML report, or machine-readable JSON.

**Gap it fills:** No open-source CLI tool performs feature-level spatial diffing with change classification. ArcGIS has "Detect Feature Changes" but nothing open-source matches it.

**Key commands:**
- `geodiff <before> <after> --key PERMIT_ID` — detect all changes
- `geodiff <before> <after> --format html --output diff_report.html`
- `geodiff <before> <after> --geometry-tolerance 1.0` — tolerance in meters
- `geodiff summary <before> <after>` — counts only

```bash
geodiff permits_2023.gpkg permits_2024.gpkg --key PERMIT_ID
# Added:     47 features
# Deleted:   12 features
# Moved:      8 features (centroid > 1m)
# Reshaped:  23 features (geometry changed)
# Attr-only: 156 features (geometry unchanged, attributes changed)
# Unchanged: 2,391,754 features
```

**Depends:** geo, geo-algorithms (this org), clap, geozero  
**Binary:** Yes  
**crates.io:** Yes (planned)

---

#### 17. `cog-cli` 🦀

**CLI for inspecting and processing Cloud Optimized GeoTIFFs.**

Inspect COG metadata, validate COG structure, extract windows, generate overviews, and convert GeoTIFFs to COG format. No GDAL required — pure Rust implementation.

**Gap it fills:** `rio-cogeo` (Python) is the standard tool but requires Python. No standalone Rust/native binary for COG validation and processing exists.

**Key commands:**
- `cog info <file|url>` — metadata, overviews, CRS, extent, statistics
- `cog validate <file|url>` — verify COG structure (overview order, tiling)
- `cog window <url> --bbox <...> --zoom <n>` — extract window as GeoTIFF
- `cog convert <input> <output>` — convert GeoTIFF to COG
- `cog stats <file|url>` — raster statistics per band

```bash
cog info https://data.example.gov/dem.tif
# Format: GeoTIFF (COG) ✓
# CRS: EPSG:3089, Size: 40960x40960
# Overviews: 5 levels (2 4 8 16 32)
# Bands: 1 (Float32, nodata: -9999)
# Extent: (-84.8, 37.8, -83.9, 38.6)

cog validate https://data.example.gov/dem.tif
# ✓ Tiled: 512x512
# ✓ Overviews present and ordered
# ✓ HTTP range request compatible
```

**Depends:** cog-reader (this org), clap, tokio  
**Binary:** Yes  
**crates.io:** Yes (planned)

---

### Tier 7: WASM-Ready Utilities

---

#### 18. `geo-wasm` 🦀

**WASM-compiled geospatial utilities for browser and edge environments.**

Bundles `coord-parse`, `tile-math`, `mvt-rs`, and core spatial predicates into a single WASM package. Provides a JavaScript/TypeScript API via `wasm-bindgen`. Designed as the Rust-powered core for browser geospatial applications.

**Gap it fills:** No single WASM package provides coordinate parsing, tile math, MVT decoding, and spatial predicates for browser use. Applications assemble these from multiple JS libraries.

**Key WASM exports:**
- `parseCoordinate(input: string) -> CoordResult`
- `bboxToTiles(west, south, east, north, zoom) -> TileArray`
- `decodeMvt(bytes: Uint8Array, layer?: string) -> GeoJSONString`
- `tileIntersects(tile, bbox) -> boolean`
- `simplify(geojson: string, tolerance: f64) -> GeoJSONString`

```typescript
// JavaScript usage after npm install @chrislyonsKY/geo-wasm
import init, { parseCoordinate, bboxToTiles, decodeMvt } from '@chrislyonsKY/geo-wasm';

await init();

const coord = parseCoordinate("38°02'53\"N 84°30'04\"W");
// { lat: 38.048, lon: -84.501, format: "DMS", confidence: 0.99 }

const tiles = bboxToTiles(-84.5, 38.0, -84.3, 38.2, 14);
// [{ z: 14, x: 4523, y: 6234 }, ...]
```

**Depends:** coord-parse, tile-math, mvt-rs, geo (all this org or upstream), wasm-bindgen  
**WASM target:** Yes (primary target)  
**npm:** `@chrislyonsKY/geo-wasm`  
**crates.io:** Yes (planned)

---

#### 19. `spatial-hash` 🦀

**Geohash, S2, and H3 spatial hashing utilities in pure Rust.**

Encode/decode geohash, S2 cell IDs, and H3 cell indexes. Grid generation, neighbor finding, and k-ring utilities. No C bindings — pure Rust implementations. WASM-compatible.

**Gap it fills:** `geohash` crate exists for geohash only. S2 and H3 Rust implementations are either missing or bind to C libraries. No crate provides all three with a unified API.

**Key functions:**
- `geohash_encode(lat: f64, lon: f64, precision: usize) -> String`
- `geohash_decode(hash: &str) -> Result<BoundingBox>`
- `geohash_neighbors(hash: &str) -> [String; 8]`
- `s2_encode(lat: f64, lon: f64, level: u64) -> u64`
- `s2_covering(bbox: &BoundingBox, level: u64) -> Vec<u64>`
- `h3_encode(lat: f64, lon: f64, resolution: u8) -> u64`
- `h3_k_ring(cell: u64, k: u32) -> Vec<u64>`
- `h3_to_boundary(cell: u64) -> Polygon`

```rust
use spatial_hash::{geohash_encode, h3_encode, h3_k_ring};

let hash = geohash_encode(38.048, -84.501, 7); // "dnjzk3g"
let h3_cell = h3_encode(38.048, -84.501, 9);
let ring = h3_k_ring(h3_cell, 2); // all H3 cells within 2 steps
```

**Depends:** (minimal — pure Rust implementations)  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

#### 20. `wkt-io` 🦀

**Fast WKT/WKB reader and writer for `geo-types`.**

Parse Well-Known Text and Well-Known Binary to `geo-types` geometries. Serialize `geo-types` to WKT/WKB. Handles 2D, 3D, and 4D coordinates. WASM-compatible.

**Gap it fills:** `wkt` and `wkb` crates exist but each handles only one format. No single crate handles WKT + WKB + WKT-Z/M + EWKB (PostGIS) in one package with `geo-types` integration.

**Key functions:**
- `parse_wkt(wkt: &str) -> Result<Geometry>`
- `parse_wkb(bytes: &[u8]) -> Result<Geometry>`
- `parse_ewkb(bytes: &[u8]) -> Result<(Geometry, Option<u32>)>` (with SRID)
- `to_wkt(geom: &Geometry) -> String`
- `to_wkb(geom: &Geometry) -> Vec<u8>`
- `to_ewkb(geom: &Geometry, srid: u32) -> Vec<u8>`

```rust
use wkt_io::{parse_wkt, parse_ewkb, to_wkt, to_ewkb};

let geom = parse_wkt("POLYGON((-84.5 38.0, -84.3 38.0, -84.3 38.2, -84.5 38.2, -84.5 38.0))")?;
let wkt_str = to_wkt(&geom);

// PostGIS EWKB round-trip
let (geom, srid) = parse_ewkb(&ewkb_bytes)?;
let ewkb = to_ewkb(&geom, srid.unwrap_or(4326));
```

**Depends:** geo-types  
**WASM:** Yes  
**crates.io:** Yes (planned)

---

## Dependency Graph (Internal)

```
geo-wasm
  ├── coord-parse        (pure Rust)
  ├── tile-math          (pure Rust)
  └── mvt-rs
        └── prost (protobuf)

tippecanoe-rs
  ├── mvt-rs
  └── pmtiles-rs
        └── tokio, bytes

geo-query
  ├── spatial-index
  │     └── rstar
  └── geo-algorithms

tissot
  ├── geo
  └── proj

raster-ops            (ndarray only)
terrain-rs            (ndarray only)

geolint (CLI)
  ├── geoparquet-rs
  └── clap

geodiff-cli (CLI)
  └── geo-algorithms

cog-cli (CLI)
  └── cog-reader
        └── tokio, bytes, ndarray
```

## Cross-Ecosystem Role

| Domain | Rust Crate | Powers |
|--------|-----------|--------|
| Projection diagnostics | `tissot` | QGIS plugin (PyO3), tissot CLI |
| Coordinate parsing | `coord-parse` | `geo-wasm` (npm WASM package) |
| Tile math | `tile-math` | `geo-wasm`, `tilesystem` (Python) |
| MVT encoding | `mvt-rs` | `geo-wasm`, `tippecanoe-rs` |
| PMTiles | `pmtiles-rs` | Python `pmtilespy` (potential PyO3 backend) |
| Geometry algorithms | `geo-algorithms` | `geodiff-cli`, `geo-query` |
| Spatial hashing | `spatial-hash` | Standalone utility |
| WASM bundle | `geo-wasm` | npm `@chrislyonsKY/geo-wasm` |

## Build Priority

| Phase | Crates | Rationale |
|-------|--------|-----------|
| **1** | `tissot`, `coord-parse`, `tile-math`, `wkt-io` | tissot is live; others are pure Rust, high utility |
| **2** | `mvt-rs`, `spatial-hash`, `geo-algorithms` | WASM-ready, foundational for other crates |
| **3** | `pmtiles-rs`, `geoparquet-rs`, `spatial-index` | Cloud-native I/O tier |
| **4** | `geo-wasm`, `geo-query`, `raster-ops` | Composite crates depending on phase 1-3 |
| **5** | `cog-reader`, `terrain-rs`, `flatgeobuf-stream` | Async I/O crates |
| **6** | `geolint`, `geodiff-cli`, `cog-cli`, `tippecanoe-rs` | CLI tools — depend on most of the above |
