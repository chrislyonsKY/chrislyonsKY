# aboveR — R Package Plan

> LiDAR terrain analysis for environmental, infrastructure, and natural resource applications in R  
> Built on KyFromAbove — applicable to any high-resolution LiDAR program

-----

## Why Not Just Use lidR?

**lidR is excellent** — 1,000+ citations, mature, well-maintained. But it’s built around forestry workflows: canopy height models, tree segmentation, timber inventory metrics. State agencies, consulting firms, and researchers working with statewide LiDAR programs like KyFromAbove need terrain-focused analysis tools that lidR doesn’t provide:

|lidR (forestry)             |aboveR (terrain & environmental)                                                |
|----------------------------|--------------------------------------------------------------------------------|
|Canopy height models        |Terrain change detection (any cause — mining, construction, landslides, erosion)|
|Individual tree segmentation|Cut/fill volume estimation between survey epochs                                |
|Forest inventory metrics    |Floodplain and channel morphology analysis                                      |
|Timber volume estimation    |Impoundment, pond, and reservoir capacity curves                                |
|Crown delineation           |Infrastructure corridor profiling (roads, pipelines, transmission lines)        |
|Forest gap analysis         |Landslide and slope failure detection                                           |
|Biomass estimation          |Building footprint and structure height extraction                              |
|Leaf area index             |Surface roughness and terrain stability assessment                              |

**aboveR builds on top of lidR** (and terra/sf), not replacing it. It uses lidR for LAS I/O and base point cloud operations, then adds terrain-focused domain logic on top.

-----

## Package Overview

**Name:** `aboveR` (named for KyFromAbove — but works with any LiDAR data)

**Tagline:** LiDAR terrain analysis for environmental, infrastructure, and natural resource applications

**Hex sticker:** Violet tier (Processing). Mountain cross-section silhouette with LiDAR scan lines (parallel diagonal lines) emanating from above. One scan line highlighted in violet.

**Depends:** lidR, terra, sf

**Suggests:** rgl, mapview, ggplot2, whitebox, rstac

-----

## Core Capabilities

### 1. Terrain Change Detection

Compare two or more LiDAR acquisitions to detect terrain changes from any cause — mining, construction, erosion, landslides, stream migration, or development.

```r
library(aboveR)

# Load two epochs
pre <- readLAS("2018_survey.laz")
post <- readLAS("2024_survey.laz")

# Generate DTMs
dtm_pre <- rasterize_terrain(pre, res = 1, tin())
dtm_post <- rasterize_terrain(post, res = 1, tin())

# Compute change
change <- terrain_change(dtm_pre, dtm_post)
plot(change)  # Red = cut, blue = fill, white = no change

# Summarize by any polygon zone (permits, parcels, watersheds, etc.)
zones <- st_read("analysis_zones.gpkg")
summary <- change_by_zone(change, zones, id_field = "ZONE_ID")
# Returns: zone_id, area_m2, cut_volume_m3, fill_volume_m3, net_change_m3

# Classify change by magnitude
classified <- classify_change(
  change,
  thresholds = c(-2, -0.5, 0.5, 2),  # meters
  labels = c("major_cut", "minor_cut", "stable", "minor_fill", "major_fill")
)
```

### 2. Volume Estimation

Calculate cut/fill volumes, pond/reservoir capacity, stockpile volumes, and earthwork quantities from point clouds or DEMs. Applicable to mining spoil, construction earthwork, stormwater ponds, farm ponds, flood storage, and more.

```r
# Volume between two surfaces (cut/fill for any earthwork)
volume <- estimate_volume(
  surface = dtm_post,
  reference = dtm_pre,  # or a flat plane at elevation X
  boundary = project_boundary_sf,
  method = "trapezoidal"  # or "simpson", "triangulated"
)
# Returns: volume_m3, area_m2, mean_depth_m, max_depth_m

# Impoundment/reservoir capacity curve
# Works for: mine impoundments, farm ponds, stormwater basins, reservoirs
capacity <- impoundment_curve(
  dem = dtm_post,
  pour_point = c(x, y),
  max_stage_m = 15,
  interval_m = 0.5
)
plot(capacity)  # Stage-storage curve

# Stockpile / spoil pile volume above ground surface
pile_vol <- stockpile_volume(
  surface_dtm = dtm_post,
  ground_dtm = dtm_pre,     # or interpolated ground under pile
  boundary = pile_boundary_sf
)
```

### 3. Terrain Profiling

Extract cross-sectional profiles along any transect — stream channels, road corridors, pipeline routes, dam axes, slopes, or property boundaries.

```r
# Profile along a transect line
profile <- terrain_profile(
  las_or_dtm = dtm_post,
  transect = transect_sf,  # sf LINESTRING
  interval_m = 1.0
)
plot(profile)  # Elevation vs distance

# Profile along a boundary (permit, parcel, ROW, etc.)
boundary_profile <- boundary_terrain_profile(
  dtm = dtm_post,
  boundary = boundary_sf,
  buffer_m = 50  # Profile extends 50m inside and outside
)

# Stream channel cross-sections at regular intervals
xs_profiles <- channel_cross_sections(
  dtm = dtm_post,
  centerline = stream_centerline_sf,
  spacing_m = 100,       # Cross-section every 100m
  width_m = 50           # 50m wide cross-sections
)
plot(xs_profiles)

# Road / corridor profile with grade analysis
corridor <- corridor_profile(
  dtm = dtm_post,
  centerline = road_centerline_sf,
  width_m = 20
)
# Returns: station, elevation, grade_pct, cross_slope_pct
```

### 4. Slope & Landslide Analysis

Identify steep slopes, potential landslide areas, and slope stability indicators from high-resolution DEMs. Kentucky’s terrain — particularly in eastern KY — is prone to landslides.

```r
# Classify slopes by steepness
slope_class <- classify_slopes(
  dtm = dtm_post,
  breaks_deg = c(0, 15, 30, 45, 60, 90),
  labels = c("flat", "gentle", "moderate", "steep", "very_steep")
)

# Detect steep slope features (highwalls, bluffs, road cuts, landslide scarps)
steep_features <- detect_steep_features(
  dtm = dtm_post,
  slope_threshold_deg = 50,
  min_height_m = 3,
  min_length_m = 10
)

# Landslide susceptibility indicators
susceptibility <- landslide_indicators(
  dtm = dtm_post,
  slope_threshold_deg = 25,
  curvature_threshold = -0.05,  # concave
  contributing_area_threshold = 500
)
# Returns: SpatRaster with susceptibility index

# Surface roughness (post-disturbance stability assessment)
roughness <- surface_roughness(dtm_post, window_m = 5)
```

### 5. Floodplain & Hydro-Terrain Analysis

LiDAR-derived terrain analysis for floodplain mapping, channel morphology, and hydrologic assessment.

```r
# Floodplain delineation from DEM
floodplain <- delineate_floodplain(
  dtm = dtm_post,
  stream = stream_sf,
  height_above_stream_m = 3.0  # HAND method
)

# Channel morphology metrics
morphology <- channel_morphology(
  dtm = dtm_post,
  centerline = stream_centerline_sf,
  spacing_m = 50
)
# Returns per cross-section: bankfull_width, bankfull_depth,
#   width_depth_ratio, entrenchment_ratio, sinuosity

# Quantify sediment in a pond, basin, or channel reach
sediment <- sedimentation_volume(
  current_dtm = dtm_post,
  as_built_dtm = dtm_original,
  boundary = pond_sf
)
# Returns: sediment_volume_m3, capacity_remaining_m3, capacity_lost_pct

# Detect erosion channels / gullies from high-resolution DEM
channels <- detect_gullies(
  dtm = dtm_post,
  min_depth_m = 0.3,
  min_length_m = 10,
  flow_accumulation_threshold = 100
)
```

### 6. Building & Structure Analysis

Extract building footprints, heights, and roof characteristics from classified LiDAR point clouds — useful for tax assessment, damage assessment, urban planning, and flood risk analysis.

```r
# Extract building footprints and heights from classified point cloud
buildings <- extract_buildings(
  las = las_classified,            # Needs ASPRS class 6 (buildings)
  min_area_m2 = 20,
  min_height_m = 2.0
)
# Returns: sf with footprint polygons + height, area, volume attributes

# Building height statistics within zones
bldg_stats <- building_stats_by_zone(
  buildings = buildings,
  zones = parcels_sf,
  id_field = "PARCEL_ID"
)
# Returns: parcel_id, building_count, max_height_m, total_footprint_m2

# Detect structures in flood zones
flood_exposed <- structures_in_floodplain(
  buildings = buildings,
  floodplain = floodplain_sf,
  dtm = dtm_post
)
# Returns: sf with first_floor_elevation, flood_depth_at_100yr, etc.
```

### 7. Reclamation & Restoration Monitoring

Track reclamation, restoration, or construction progress over time using multi-temporal LiDAR — applicable to mine reclamation, stream restoration, brownfield redevelopment, or any grading project.

```r
# Load time series of DTMs
epochs <- list(
  "2018" = rast("dtm_2018.tif"),
  "2020" = rast("dtm_2020.tif"),
  "2022" = rast("dtm_2022.tif"),
  "2024" = rast("dtm_2024.tif")
)

# Track progress toward a target surface
progress <- restoration_progress(
  epochs = epochs,
  original_surface = rast("dtm_pre_disturbance.tif"),
  target_surface = rast("design_grade.tif"),  # AOC, design plans, etc.
  boundary = project_boundary_sf
)
# Returns per epoch: pct_area_at_grade, volume_remaining_m3, avg_grade_diff_m
plot(progress)

# Compare against design plans (for any grading project)
conformance <- grade_conformance(
  as_built_dtm = dtm_post,
  design_dtm = rast("design_surface.tif"),
  tolerance_m = 0.3
)
# Returns: SpatRaster of conformance (within tolerance / above / below)
```

### 8. KyFromAbove Integration

Kentucky-specific utilities for working with KyFromAbove LiDAR data. Phase 1 (5ft) covers the entire state; Phase 2 (2ft) is higher resolution. Data is hosted on AWS S3 at `s3://kyfromabove/` with a STAC catalog maintained by Ian Horn at `github.com/ianhorn/kyfromabove-stac`.

```r
# Discover available KyFromAbove tiles for an area of interest
tiles <- kfa_find_tiles(
  boundary = aoi_sf,     # Any sf polygon — doesn't have to be a permit
  phase = 2,
  product = "laz"        # or "dem", "dsm", "contours"
)
print(tiles)  # Tile names, S3 URLs, point density, acquisition dates

# Download and merge tiles
las <- kfa_download_merge(tiles, output = tempfile(fileext = ".laz"))

# Also works with derived products
dem <- kfa_download_dem(tiles, output = tempfile(fileext = ".tif"))

# Standardize classification (KyFromAbove uses custom classes)
las <- kfa_reclassify(las)  # Maps to ASPRS standard

# Search KyFromAbove STAC catalog for temporal coverage
coverage <- kfa_stac_search(
  boundary = aoi_sf,
  datetime = "2018-01-01/2024-12-31"
)
# Returns: tibble of available acquisitions with dates, density, products

# Compare Phase 1 vs Phase 2 for an area
comparison <- kfa_compare_phases(boundary = aoi_sf)
# Returns: resolution, point density, acquisition dates for each phase
```

-----

## Module Structure

```
R/
├── change.R           # terrain_change(), change_by_zone(), classify_change()
├── volume.R           # estimate_volume(), impoundment_curve(), stockpile_volume()
├── profile.R          # terrain_profile(), boundary_terrain_profile(),
│                      #   channel_cross_sections(), corridor_profile()
├── slope.R            # classify_slopes(), detect_steep_features(),
│                      #   landslide_indicators(), surface_roughness()
├── hydro.R            # delineate_floodplain(), channel_morphology(),
│                      #   sedimentation_volume(), detect_gullies()
├── buildings.R        # extract_buildings(), building_stats_by_zone(),
│                      #   structures_in_floodplain()
├── restoration.R      # restoration_progress(), grade_conformance()
├── kyfromabove.R      # kfa_find_tiles(), kfa_download_merge(), kfa_download_dem(),
│                      #   kfa_reclassify(), kfa_stac_search(), kfa_compare_phases()
├── classify.R         # Point classification helpers (ground, buildings, veg, noise)
├── visualize.R        # plot methods, color ramps for change/slope/conformance maps
└── utils.R            # Shared utilities (bbox handling, resolution matching, etc.)
```

-----

## Application Domains

aboveR serves multiple user communities, not just mining:

|Domain                             |Key Functions                                                               |Users                                                                      |
|-----------------------------------|----------------------------------------------------------------------------|---------------------------------------------------------------------------|
|**Mining & Reclamation**           |terrain_change, estimate_volume, restoration_progress, detect_steep_features|State mining regulatory agencies, mine operators, environmental consultants|
|**Floodplain Management**          |delineate_floodplain, channel_morphology, structures_in_floodplain          |FEMA, state floodplain managers, county planning offices                   |
|**Transportation & Infrastructure**|corridor_profile, terrain_profile, estimate_volume                          |DOT, pipeline companies, utility corridor planners                         |
|**Erosion & Stormwater**           |detect_gullies, sedimentation_volume, impoundment_curve                     |Soil conservation districts, stormwater utilities, MS4 programs            |
|**Natural Hazards**                |landslide_indicators, classify_slopes, surface_roughness                    |Emergency management, geological surveys, FEMA hazard mitigation           |
|**Urban & Tax Assessment**         |extract_buildings, building_stats_by_zone                                   |PVA offices, county assessors, urban planners                              |
|**Stream Restoration**             |channel_cross_sections, channel_morphology, restoration_progress            |Stream restoration practitioners, 404 permit reviewers, watershed groups   |
|**Construction & Grading**         |grade_conformance, estimate_volume, terrain_change                          |Civil engineers, site contractors, construction QA                         |
|**Agriculture**                    |terrain_profile, delineate_floodplain, detect_gullies                       |Farm planners, NRCS, conservation districts                                |

-----

## Architectural Decisions

### AD-001: Build on lidR, don’t replace it

lidR handles LAS I/O, point cloud indexing, ground classification, and DTM generation. aboveR adds terrain-focused domain logic on top.

### AD-002: Terra for rasters, sf for vectors

Follow the modern R spatial stack. No legacy sp/raster dependencies.

### AD-003: Functions return terra/sf objects

All raster outputs are SpatRaster (terra), all vector outputs are sf. This ensures interoperability with the entire R spatial ecosystem.

### AD-004: KyFromAbove utilities are optional

Kentucky-specific functions are in a separate module. The core analysis functions work with any LiDAR data from any source.

### AD-005: Domain-neutral function names

Functions are named for what they DO (terrain_change, estimate_volume, detect_steep_features), not for a specific industry. Mining users and floodplain managers call the same functions.

### AD-006: STAC integration for KyFromAbove

Use Ian Horn’s KyFromAbove STAC catalog (`github.com/ianhorn/kyfromabove-stac`) and the S3 bucket (`s3://kyfromabove/`) for tile discovery and download. This makes KyFromAbove data accessible via the same patterns as any STAC catalog.

-----

## Hex Sticker Design

**Tier:** 4 (Processing — Violet #8B5CF6)
**Icon:** Terrain cross-section with parallel LiDAR scan lines from above. One scan line highlighted in violet. Rolling terrain profile (not just a mine bench — more Kentucky hills).
**Background:** #13101E (dark violet-tinted)
**Tagline:** “terrain · volume · change”
**Package name:** aboveR (JetBrains Mono Bold, white)

-----

## Comparison with Related Packages

|Package         |Focus                                                                                |Overlap with aboveR                                                                                                                            |
|----------------|-------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
|**lidR**        |Forestry LiDAR (canopy, trees, biomass)                                              |I/O and base operations (aboveR depends on lidR)                                                                                               |
|**lasR**        |High-performance LiDAR processing pipeline                                           |Production processing (complementary)                                                                                                          |
|**terra**       |General raster operations                                                            |DTM manipulation (aboveR depends on terra)                                                                                                     |
|**whitebox**    |Hydrologic/geomorphic analysis                                                       |Some overlap in channel detection, flow accumulation, HAND                                                                                     |
|**RSAGA**       |Geomorphometry                                                                       |Terrain derivatives (slope, aspect, curvature)                                                                                                 |
|**FLOODeminger**|Flood inundation mapping                                                             |Some overlap in floodplain delineation                                                                                                         |
|**aboveR**      |Terrain analysis for environmental, infrastructure, and natural resource applications|**Unique:** multi-domain terrain change, volume estimation, KyFromAbove integration, grade conformance, building extraction, channel morphology|

-----

## Build Priority

This is a Phase 3-4 R package. It requires:

1. Familiarity with lidR API (reading the lidR book)
1. KyFromAbove sample data for testing (Phase 1 + Phase 2)
1. Multi-temporal DEM pairs for change detection testing

**Milestones:**

|Milestone               |Functions                                                                          |Value                                                  |
|------------------------|-----------------------------------------------------------------------------------|-------------------------------------------------------|
|**M1: Core terrain**    |`terrain_change()`, `estimate_volume()`, `terrain_profile()`                       |Immediately useful for any terrain comparison work     |
|**M2: KyFromAbove**     |`kfa_find_tiles()`, `kfa_download_merge()`, `kfa_reclassify()`, `kfa_stac_search()`|Makes KyFromAbove data accessible from R               |
|**M3: Hydro-terrain**   |`delineate_floodplain()`, `channel_morphology()`, `detect_gullies()`               |Floodplain and stream analysis users                   |
|**M4: Slopes & hazards**|`classify_slopes()`, `detect_steep_features()`, `landslide_indicators()`           |Natural hazards and slope stability                    |
|**M5: Buildings**       |`extract_buildings()`, `building_stats_by_zone()`, `structures_in_floodplain()`    |Urban analysis, flood risk, tax assessment             |
|**M6: Restoration**     |`restoration_progress()`, `grade_conformance()`                                    |Mining reclamation, stream restoration, construction QA|