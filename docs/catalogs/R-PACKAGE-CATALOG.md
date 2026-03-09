# Chris Lyons — R Package Catalog

> Companion R packages to the Chris Lyons Python ecosystem  
> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)

---

## Philosophy

The R spatial ecosystem (sf, terra, stars, rstac, geoarrow) is mature and excellent. These packages don't try to replace it — they fill specific gaps where practitioners need tooling that doesn't exist, particularly around **data quality, ETL auditing, cloud-native workflows, accessibility, and the bridge between R and ArcGIS/enterprise GIS**. Several are direct R ports of the Python Chris Lyons packages, enabling the same workflows in both languages.

All packages follow tidyverse conventions: pipe-friendly, tibble outputs, tidy evaluation where appropriate.

---

## R Packages (10)

### Tier 1: Data Discovery & Quality

---

#### 1. `geoinventoryr` 📦

**Scan, document, and diff any spatial dataset from R.**

The R companion to `geoinventory` (Python). Scans GeoPackage, Shapefile, GeoParquet, PostGIS, STAC catalogs, COG — produces tidy tibble inventories with schema, CRS, extent, and field profiling. Markdown/HTML reports via RMarkdown templates.

**Gap it fills:** R users have `sf::st_layers()` and `terra::describe()` but nothing that inventories an entire data estate across formats with profiling, diffing, and reporting in one package.

**Key functions:**
- `scan_source(path)` → tibble inventory
- `scan_geoparquet(path)` → GeoParquet metadata without full read (via arrow)
- `scan_stac(url)` → STAC collection inventory
- `diff_inventory(old, new)` → schema diff tibble
- `report_inventory(inv, format = "html")` → RMarkdown report

**Depends:** sf, arrow, geoarrow, rstac, terra, DBI  
**CRAN:** Yes (planned)

---

#### 2. `spatialvalidate` 📦

**Geometry and attribute validation for sf objects.**

R companion to `spatial-validate` (Python). Composable validation checks that return tidy tibbles of findings. Pipe-friendly for integration with dplyr workflows.

**Gap it fills:** `sf::st_is_valid()` exists but there's no unified validation framework that checks geometry + attributes + CRS + domains + topology in one pass with structured output.

**Key functions:**
- `validate(sf_obj, checks)` → validation report tibble
- `check_valid_geometry()`, `check_no_null_geometry()`, `check_crs_matches(epsg)`
- `check_no_nulls(fields)`, `check_unique(field)`, `check_domain(field, values)`
- `check_topology(type = "no_overlaps")`
- `fix_geometry(sf_obj)` → repaired sf with make_valid

**Depends:** sf, dplyr  
**CRAN:** Yes (planned)

---

#### 3. `schemaenforcer` 📦

**Schema contract enforcement for spatial datasets in R.**

Define expected schemas as YAML, validate sf/terra objects against them. CI-friendly with `testthat`-compatible assertions.

**Gap it fills:** No R package provides declarative schema contracts for spatial data. `pointblank` does tabular validation but doesn't understand CRS, geometry types, or spatial domains.

**Key functions:**
- `load_contract("schema.yml")` → schema object
- `check_schema(sf_obj, contract)` → pass/fail with violations
- `generate_contract(sf_obj)` → auto-generate YAML from existing data
- `expect_schema(sf_obj, contract)` → testthat assertion

**Depends:** sf, yaml, testthat (Suggests)  
**CRAN:** Yes (planned)

---

#### 4. `geodatahealth` 📦

**Data staleness and health monitoring for geodatabases.**

Scans file geodatabases, GeoPackages, and PostGIS schemas for last-modified dates, record counts, and metadata completeness. Produces "health card" tibbles with Active/Stale/Critical/Unknown classification.

**Gap it fills:** Nobody in R-land has a data staleness tracker. Enterprise GIS shops using R for analysis still manually track which datasets are current.

**Key functions:**
- `scan_health(path, thresholds)` → health tibble
- `classify_staleness(days, thresholds)` → status label
- `health_report(health_tbl, format = "html")` → RMarkdown health report
- `health_badge(health_tbl)` → summary badge (Active: 12, Stale: 3, Critical: 1)

**Depends:** sf, DBI, gt (for reports)  
**CRAN:** Yes (planned)

---

### Tier 2: ETL & Transformation

---

#### 5. `spatialetl` 📦

**Declarative spatial ETL pipelines with tidyverse syntax.**

R companion to `spatial-etl` (Python). Build reproducible spatial data pipelines using a pipe-based API. Tracks row counts, timing, and validation results through every step.

**Gap it fills:** R has great spatial tools but no pipeline framework that provides structured audit trails for ETL operations. People write ad-hoc scripts; this makes them reproducible and auditable.

**Key functions:**
- `pipeline("name") |> extract(read_sf(...)) |> transform(reproject(...), validate_geometry(...)) |> load(write_geoparquet(...))`
- `run_pipeline(pipeline)` → PipelineResult with per-step metrics
- Steps: `reproject()`, `clip()`, `buffer()`, `spatial_join()`, `drop_nulls()`, `validate_geometry()`, `rename_fields()`
- Readers: `read_sf()`, `read_geoparquet()`, `read_postgis()`
- Writers: `write_sf()`, `write_geoparquet()`, `write_postgis()`

**Depends:** sf, arrow, geoarrow, DBI, dplyr  
**CRAN:** Yes (planned)

---

#### 6. `formatbridge` 📦

**One-function format conversion for spatial data.**

Convert between any sf-readable format with CRS reprojection, encoding detection, and geometry validation. The `ogr2ogr` of R, but with R-native error handling and reporting.

**Gap it fills:** `sf::st_write()` works but doesn't handle encoding detection, validation, or format-specific quirks (like shapefile field name truncation warnings). This wraps the whole conversion workflow cleanly.

**Key functions:**
- `convert(input, output, crs = NULL, validate = TRUE)`
- `detect_encoding(shapefile_path)` → encoding string
- `validate_conversion(input, output)` → comparison tibble

**Depends:** sf, arrow (for GeoParquet)  
**CRAN:** Yes (planned)

---

### Tier 3: Visualization & Accessibility

---

#### 7. `mapaccessibility` 📦

**WCAG 2.1 AA color accessibility for cartographic palettes.**

R companion to `map-accessibility` (Python). Check color palettes for contrast ratios, simulate colorblind vision, and score palette accessibility. Works with ggplot2 scales and tmap palettes.

**Gap it fills:** `colorspace` has HCL-based tools and `colorblindr` simulates colorblindness on ggplot2 plots, but neither provides WCAG 2.1 contrast ratio checking or palette accessibility scoring specifically for cartographic use.

**Key functions:**
- `contrast_ratio(fg, bg)` → numeric ratio
- `check_wcag(fg, bg, level = "AA", size = "normal")` → pass/fail
- `simulate_cvd(colors, type = "deuteranopia")` → transformed hex colors
- `score_palette(colors)` → accessibility score (0-1)
- `pal_accessible(n)` → pre-built accessible color palette
- `scale_fill_accessible()`, `scale_color_accessible()` → ggplot2 scales

**Depends:** grDevices  
**Suggests:** ggplot2, colorspace  
**CRAN:** Yes (planned)

---

#### 8. `kartocolors` 📦

**Curated cartographic color palettes with built-in accessibility.**

A palette package (like `viridis` or `RColorBrewer`) but specifically designed for cartographic use. Every palette is pre-tested for WCAG 2.1 AA compliance and colorblind safety. Includes sequential, diverging, and qualitative schemes for common map types.

**Gap it fills:** `viridis` is great for continuous data but limited for qualitative maps. `RColorBrewer` palettes aren't all colorblind-safe. No existing package provides cartography-specific palettes with built-in accessibility guarantees and ggplot2/tmap integration.

**Key functions:**
- `karto_pal(n, name = "terrain")` → hex color vector
- `scale_fill_karto(palette = "terrain")` → ggplot2 scale
- `scale_color_karto(palette = "landuse")` → ggplot2 scale
- `karto_palettes()` → tibble of all palettes with accessibility scores
- Palette categories: terrain, landuse, hydro, population, environmental, diverging, sequential

**Depends:** grDevices  
**Suggests:** ggplot2, tmap, scales  
**CRAN:** Yes (planned)

---

### Tier 4: Domain-Specific

---

#### 9. `watershedtools` 📦

**Hydrologic analysis utilities built on NHDPlus.**

R companion to `watershed-tools` (Python). HUC validation, watershed boundary retrieval, upstream/downstream tracing, and permit-watershed overlay. Built on sf + NHDPlus data model.

**Gap it fills:** `nhdplusTools` exists and is excellent for NHDPlus access, but it doesn't provide the permit-overlay, staleness-aware watershed analysis, and CHIA-style workflow patterns that environmental agencies need. This builds on top of `nhdplusTools` rather than replacing it.

**Key functions:**
- `validate_huc(code)` → logical (validates format and hierarchy)
- `get_watershed(huc12, wbd_path)` → sf boundary
- `trace_upstream(point, flowlines)` → sf of upstream reaches
- `trace_downstream(point, flowlines)` → sf of downstream reaches
- `overlay_permits(permits_sf, watersheds_sf)` → intersection tibble
- `watershed_summary(huc12, permits, flowlines)` → CHIA-style summary

**Depends:** sf, nhdplusTools, dplyr  
**CRAN:** Yes (planned)

---

#### 10. `geoaitools` 📦

**LLM utility functions for geospatial workflows in R.**

R companion to `geoai-toolkit` (Python). Coordinate parsing, sf object description for LLM context windows, and prompt template management.

**Gap it fills:** Zero R packages exist for bridging LLMs with spatial data. This provides the same coordinate parsing and dataset description capabilities as the Python version.

**Key functions:**
- `parse_coordinates("38°02'53\"N 84°30'04\"W")` → tibble with lat/lon
- `parse_coordinates("16S 722714 4213456")` → tibble (UTM)
- `describe_sf(sf_obj)` → character string for LLM context
- `dms_to_dd(38, 2, 53, "N")` → numeric
- `dd_to_dms(38.048)` → list(deg, min, sec, dir)
- `geo_prompt(template, ...)` → rendered prompt string

**Depends:** sf (Suggests), stringr  
**CRAN:** Yes (planned)

---

## Comparison: R vs Python Packages

| Domain | Python Package | R Package | Shared? |
|--------|---------------|-----------|---------|
| Data inventory | `geoinventory` | `geoinventoryr` | Same models, different APIs |
| Validation | `spatial-validate` | `spatialvalidate` | Same checks, tidyverse output |
| Schema enforcement | `schema-enforcer` | `schemaenforcer` | Same YAML contracts |
| Data health | `geodata-health` | `geodatahealth` | Same thresholds/classification |
| ETL pipelines | `spatial-etl` | `spatialetl` | Same concepts, pipe syntax |
| Format conversion | `format-bridge` | `formatbridge` | Same logic |
| Color accessibility | `map-accessibility` | `mapaccessibility` | Same WCAG algorithms |
| Carto palettes | — | `kartocolors` | R-only (ggplot2 native) |
| Hydrology | `watershed-tools` | `watershedtools` | Same analysis, nhdplusTools integration |
| GeoAI/LLM | `geoai-toolkit` | `geoaitools` | Same coordinate parsing |

## R Package Structure Template

Every R package follows this structure:

```
packagename/
├── DESCRIPTION            # Package metadata, dependencies, version
├── NAMESPACE              # Exports (generated by roxygen2)
├── LICENSE                # MIT
├── README.md              # Badges, install, examples
├── .github/
│   └── workflows/
│       └── R-CMD-check.yml  # GitHub Actions CI
├── R/                     # Source code
│   ├── package.R          # Package-level documentation
│   ├── module1.R
│   └── module2.R
├── man/                   # Documentation (generated by roxygen2)
├── tests/
│   ├── testthat.R
│   └── testthat/
│       ├── test-module1.R
│       └── test-module2.R
├── vignettes/             # Long-form documentation
│   └── getting-started.Rmd
├── inst/
│   └── extdata/           # Sample data for examples
└── CLAUDE.md              # AI-dev context for Claude Code
```

## Build Priority

| Phase | Packages | Rationale |
|-------|----------|-----------|
| **1** | `mapaccessibility`, `kartocolors` | Low dependency count, high newsletter/NACIS value, pure R |
| **2** | `geoaitools`, `spatialvalidate` | Unique in R ecosystem, direct Python companions |
| **3** | `geoinventoryr`, `schemaenforcer` | Data documentation tooling |
| **4** | `spatialetl`, `formatbridge` | ETL infrastructure |
| **5** | `geodatahealth`, `watershedtools` | Domain-specific, builds on earlier packages |
