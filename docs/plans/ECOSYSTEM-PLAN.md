# Chris Lyons — Multi-Ecosystem Package Plan

> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)  
> Geospatial packages across PyPI, CRAN, QGIS, crates.io, NuGet, and npm  
> Same concepts, idiomatic APIs, shared schema contracts

---

## Philosophy

Chris Lyons fills specific gaps in the geospatial tooling landscape — data quality, ETL auditing, cloud-native workflows, accessibility, change detection, and LLM integration. The core insight is that these problems are **language-agnostic**: a mining regulator needs schema enforcement whether they work in R, Python, C#, or TypeScript.

Each ecosystem gets packages that are **idiomatic to that community**, not mechanical ports. A Rust crate uses `Result<T, E>` and traits, not Python-style exceptions. A NuGet package follows ArcGIS Pro SDK patterns, not console-app conventions. An npm package ships ESM with TypeScript declarations, not CommonJS.

**What's shared across ecosystems:**
- YAML schema contracts (schemaenforcer) — define once, validate anywhere
- Accessibility algorithms (WCAG 2.1 AA contrast, CVD simulation) — identical math
- Cartographic palettes (kartocolors) — same hex values, different API wrappers
- Coordinate parsing grammars — same regex/parser logic
- Tier color system and branding — unified visual identity

**What's ecosystem-specific:**
- API surface (pipes in R, decorators in Python, builder patterns in C#, traits in Rust)
- Dependency graphs (sf vs GeoPandas vs NetTopologySuite vs geo crate vs Turf.js)
- Distribution channels (CRAN, PyPI, QGIS plugin repo, crates.io, NuGet Gallery, npm)
- Testing frameworks and CI patterns

---

## Ecosystem Summary

| Ecosystem | Registry | Count | Primary Audience |
|-----------|----------|-------|-----------------|
| **PyPI** | pypi.org | 20 | GIS analysts, data scientists, ArcPy users |
| **CRAN** | cran.r-project.org | 19 | R spatial community, academic researchers, environmental agencies |
| **QGIS** | plugins.qgis.org | 8 | Desktop GIS users, field analysts, non-coders |
| **crates.io** | crates.io | 10 | Performance-critical pipelines, CLI tools, WASM targets |
| **NuGet** | nuget.org | 8 | ArcGIS Pro add-in developers, .NET enterprise GIS |
| **npm** | npmjs.com | 12 | Web map developers, serverless geospatial, full-stack apps |
| **Total** | | **77** | |

---

## Tier System (Shared Across All Ecosystems)

| Tier | Name | Accent Color | Background | Domain |
|------|------|-------------|------------|--------|
| 1 | Discovery | `#14B8A6` Teal | `#0F172A` | Scanning, inventory, documentation |
| 2 | ETL | `#F59E0B` Amber | `#1A1708` | Pipelines, transformation, movement |
| 3 | Quality | `#F43F5E` Rose | `#1A0F12` | Validation, compliance, accessibility |
| 4 | Processing | `#8B5CF6` Violet | `#13101E` | Analysis, computation, spatial ops |
| 5 | Tools | `#0EA5E9` Sky | `#0C1929` | AI, monitoring, developer experience |

---

## 1. PyPI — Python Packages (20)

> **Status:** Scaffolded  
> **Conventions:** PEP 8, type hints, dataclass models, Click CLIs, pytest  
> **CI:** GitHub Actions (ruff, mypy, pytest, build, publish)

### Tier 1: Discovery (Teal)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 1 | `geoinventory` | Scan, document, and diff spatial datasets across formats | geopandas, pyarrow, fiona, pystac-client |
| 2 | `geodata-health` | Data staleness and health monitoring for geodatabases | fiona, psycopg2, Jinja2 |
| 3 | `sde-inspector` | ArcSDE/enterprise geodatabase diagnostics (versions, compress, locks) | arcpy, cx_Oracle/oracledb |
| 4 | `lyrx-tools` | Parse, compare, and manage ArcGIS Pro .lyrx layer files | json, lxml |

### Tier 2: ETL (Amber)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 5 | `spatial-etl` | Declarative spatial ETL pipelines with audit trails | geopandas, pyarrow, sqlalchemy |
| 6 | `oracle-spatial-etl` | Oracle Spatial → GeoPackage/GeoParquet ETL | cx_Oracle/oracledb, geopandas, fiona |
| 7 | `arcpy-etl-patterns` | Reusable ArcPy ETL patterns (truncate-load, upsert, delta) | arcpy |
| 8 | `format-bridge` | One-function format conversion with encoding detection | fiona, geopandas, pyarrow |

### Tier 3: Quality (Rose)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 9 | `spatial-validate` | Composable geometry + attribute validation | shapely, geopandas |
| 10 | `fgdc-metadata` | Create, validate, and score FGDC/ISO 19115 metadata XML | lxml, jinja2 |
| 11 | `map-accessibility` | WCAG 2.1 AA contrast ratios + CVD simulation for palettes | numpy, matplotlib (optional) |
| 12 | `schema-enforcer` | YAML schema contracts for spatial datasets | geopandas, pyyaml, pytest (plugin) |

### Tier 4: Processing (Violet)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 13 | `georeg` | Georeference scanned maps with GCPs + affine/polynomial transforms | rasterio, opencv-python, shapely |
| 14 | `permit-lifecycle` | State machine for regulatory permit tracking | sqlalchemy, geopandas |
| 15 | `watershed-tools` | HUC validation, tracing, permit-watershed overlay | geopandas, pynhd, pygeohydro |
| 16 | `terrain-extract` | Profile, zonal stats, slope from DEMs | rasterio, numpy, shapely |

### Tier 5: Tools (Sky)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 17 | `geoai-toolkit` | Coordinate parsing + dataset description for LLM context | shapely (optional) |
| 18 | `arcpy-notify` | ArcGIS Pro geoprocessing monitoring + alerts | arcpy, smtplib/requests |
| 19 | `spatial-diff` | Diff two spatial datasets by key field (added/deleted/changed) | geopandas, shapely |
| 20 | `gis-project-scaffold` | Scaffold GIS projects with CLAUDE.md, ai-dev/, specs | jinja2, click |

### Python Project Structure

```
package-name/
├── pyproject.toml          # PEP 621 metadata, build config
├── src/
│   └── package_name/
│       ├── __init__.py
│       ├── module.py
│       └── py.typed         # PEP 561 marker
├── tests/
│   ├── conftest.py
│   └── test_module.py
├── docs/                    # MkDocs or Sphinx
├── assets/
│   ├── banner.svg           # 1280×320 README banner
│   ├── logo.svg             # 512×512 PyPI logo
│   └── social-preview.png   # 1280×640 GitHub social
├── .github/workflows/
│   ├── ci.yml               # Lint + test + type-check
│   └── publish.yml          # PyPI publish on tag
├── CLAUDE.md
├── ai-dev/
│   ├── architecture.md
│   ├── decisions.md
│   └── agents/
└── README.md
```

---

## 2. CRAN — R Packages (19)

> **Status:** Scaffolded (708 files, 127 exported functions)  
> **Conventions:** roxygen2, testthat v3, tidyverse pipes, `R CMD check --as-cran`  
> **CI:** GitHub Actions (macOS + Windows + Ubuntu × R-devel/release/oldrel)

### Tier 1: Discovery (Teal)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 1 | `geoinventoryr` | Scan, document, and diff spatial datasets | sf, arrow, geoarrow, rstac, terra |
| 2 | `spatialvalidate` | Composable validation checks for sf objects | sf, dplyr |
| 3 | `schemaenforcer` | YAML schema contracts with testthat assertions | sf, yaml |
| 4 | `geodatahealth` | Data staleness and health monitoring | sf, DBI, gt |

### Tier 2: ETL (Amber)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 5 | `spatialetl` | Pipe-based spatial ETL pipelines with audit trails | sf, arrow, geoarrow, DBI, dplyr |
| 6 | `formatbridge` | One-function format conversion | sf, arrow |

### Tier 3: Quality (Rose)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 7 | `mapaccessibility` | WCAG 2.1 contrast ratios + CVD simulation | grDevices |
| 8 | `kartocolors` | Curated accessible cartographic palettes + ggplot2 scales | grDevices |

### Tier 4: Processing (Violet)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 9 | `watershedtools` | Hydrologic analysis on NHDPlus | sf, nhdplusTools, dplyr |
| 10 | `aboveR` | LiDAR terrain analysis + KyFromAbove integration | lidR, terra, sf |
| 11 | `geochange` | Raster + vector change detection | terra, sf |

### Tier 5: Tools & Cloud-Native (Sky/Mixed)

| # | Package | Description | Key Dependencies |
|---|---------|-------------|-----------------|
| 12 | `geoaitools` | Coordinate parsing + sf description for LLMs | stringr |
| 13 | `cloudgeo` | Universal cloud-native I/O | sf, terra |
| 14 | `overturer` | Overture Maps via DuckDB | duckdb, DBI, sf, cli |
| 15 | `stacr` | Tidy STAC workflows | rstac, tibble, cli |
| 16 | `duckgeo` | DuckDB spatial convenience functions | duckdb, DBI, sf |
| 17 | `pmtilesr` | PMTiles reader/writer | httr2, sf, protolite |
| 18 | `envdatar` | EPA, USGS, NOAA/NWS, KY environmental APIs | httr2, jsonlite, tibble, sf, cli |
| 19 | `georender` | Publication-ready static maps | ggplot2, sf, ggspatial |

### R Project Structure

```
packagename/
├── DESCRIPTION             # CRAN-formatted metadata
├── NAMESPACE               # roxygen2-generated exports
├── LICENSE / LICENSE.md
├── R/
│   ├── package.R           # Package-level docs
│   └── *.R                 # roxygen2-documented source
├── man/                    # roxygen2-generated .Rd files
│   └── figures/
│       ├── logo.svg        # Hex sticker (web)
│       └── logo.png
├── tests/
│   ├── testthat.R
│   └── testthat/
├── vignettes/
├── inst/
│   ├── extdata/            # Bundled sample data
│   └── figures/
│       └── sticker.svg     # Hex sticker (print, hexb.in)
├── .github/workflows/
│   └── R-CMD-check.yaml    # Full platform matrix
├── .github/copilot-instructions.md
├── CLAUDE.md
├── ai-dev/
│   ├── architecture.md
│   ├── decisions.md
│   ├── agents/
│   └── guardrails/
├── cran-comments.md
└── README.md
```

---

## 3. QGIS — Processing Plugins (8)

> **Registry:** plugins.qgis.org  
> **Conventions:** QGIS Processing framework, `QgsProcessingAlgorithm` subclasses, `metadata.txt`  
> **Language:** Python (PyQGIS API)  
> **CI:** GitHub Actions (QGIS Docker images for testing)

### Design Approach

QGIS plugins expose the same logic as the Python packages but through the QGIS Processing toolbox GUI. Users get parameter dialogs, progress bars, and output layers automatically. Each plugin registers one or more `QgsProcessingAlgorithm` subclasses grouped under a "Chris Lyons" provider.

Where possible, plugins **import the corresponding PyPI package** as a dependency rather than duplicating logic. When the PyPI package isn't installed, the plugin falls back to pure-PyQGIS implementations.

### Plugin Catalog

| # | Plugin | Provider Group | Algorithms | Wraps (PyPI) |
|---|--------|---------------|------------|--------------|
| 1 | `nil-validate` | Chris Lyons › Quality | Validate Geometry, Validate Attributes, Validate Schema, Fix Geometry | `spatial-validate`, `schema-enforcer` |
| 2 | `nil-inventory` | Chris Lyons › Discovery | Scan Data Source, Diff Inventories, Health Report | `geoinventory`, `geodata-health` |
| 3 | `nil-accessibility` | Chris Lyons › Quality | Check Palette Accessibility, Simulate CVD, Score Layer Symbology | `map-accessibility` |
| 4 | `nil-convert` | Chris Lyons › ETL | Format Bridge (any→any with validation) | `format-bridge` |
| 5 | `nil-change` | Chris Lyons › Processing | Raster Change Detection, Vector Change Detection, Change to Polygons | `spatial-diff`, raster diff logic |
| 6 | `nil-watershed` | Chris Lyons › Processing | Validate HUC, Delineate Watershed, Trace Network, Permit Overlay | `watershed-tools` |
| 7 | `nil-terrain` | Chris Lyons › Processing | Terrain Profile, Zonal Extract, Volume Estimation | `terrain-extract`, aboveR concepts |
| 8 | `nil-overture` | Chris Lyons › Data | Download Overture Buildings/Places/Roads/Divisions by Extent | DuckDB via subprocess |

### QGIS Plugin Structure

```
nil-validate/
├── metadata.txt              # Plugin metadata (name, version, QGIS min, author, etc.)
├── __init__.py               # Plugin entry point (classFactory)
├── plugin.py                 # QgsProcessingProvider registration
├── algorithms/
│   ├── validate_geometry.py  # QgsProcessingAlgorithm subclass
│   ├── validate_attributes.py
│   ├── validate_schema.py
│   └── fix_geometry.py
├── ui/                       # Optional: custom dialogs beyond Processing params
├── icons/
│   └── icon.svg              # Plugin icon (tier-colored)
├── test/
│   └── test_algorithms.py    # pytest + qgis.testing
├── CLAUDE.md
└── README.md
```

### QGIS-Specific Conventions

- **Provider name:** `chrislyonsKY` (registered once, shared if multiple plugins installed)
- **Algorithm IDs:** `nil:{plugin}:{algorithm}` (e.g., `nil:validate:check_geometry`)
- **Parameters:** Use `QgsProcessingParameterFeatureSource`, `QgsProcessingParameterRasterLayer`, etc. for native QGIS type integration
- **Output:** Always register output as `QgsProcessingOutputVectorLayer` or `QgsProcessingOutputRasterLayer` so results appear in the layer panel
- **Progress:** Use `feedback.setProgress()` and `feedback.pushInfo()` for progress reporting
- **Cancellation:** Check `feedback.isCanceled()` in loops
- **Dependencies:** Declare in `metadata.txt` `plugin_dependencies=` field; use `pip_dependencies=` for PyPI packages (QGIS 3.36+ supports this)
- **Min QGIS version:** Target 3.34 LTS (current long-term release)
- **Testing:** Use `qgis.testing` module with QGIS Docker (`qgis/qgis:latest`) in CI

### metadata.txt Template

```ini
[general]
name=Chris Lyons - Validate
qgisMinimumVersion=3.34
description=Geometry and attribute validation for vector layers with structured reporting
version=0.1.0
author=Chris Lyons / Chris Lyons
email=chris@github.com/chrislyonsKY
about=Composable validation checks for vector layers. Checks geometry validity, null attributes, CRS conformance, domain constraints, and topology. Returns a validation report layer with findings. Part of the Chris Lyons geospatial toolkit.
tracker=https://github.com/chrislyonsKY/nil-validate/issues
repository=https://github.com/chrislyonsKY/nil-validate
tags=validation,geometry,quality,qa,qc,attributes,topology
homepage=https://github.com/chrislyonsKY
category=Analysis
icon=icons/icon.svg
experimental=True
pip_dependencies=spatial-validate>=0.1.0
```

---

## 4. crates.io — Rust Crates (10)

> **Registry:** crates.io  
> **Conventions:** Rust 2021 edition, `#[derive]` macros, `Result<T, E>`, serde, thiserror  
> **CI:** GitHub Actions (cargo clippy, cargo test, cargo fmt, cargo publish)  
> **Key advantage:** WASM compilation targets, zero-copy performance, CLI tools via clap

### Design Approach

Rust crates serve two audiences: (1) developers building **high-performance geospatial pipelines** and CLI tools, and (2) **WASM targets** for browser-side spatial computation. The crates prioritize correctness (strong typing, exhaustive error handling) and performance (zero-copy where possible, streaming I/O).

Each crate ships with an optional `cli` feature that builds a command-line binary via `clap`. This means `cargo install geo-validate` gives you a CLI tool immediately.

The geo ecosystem in Rust is anchored by the `geo` crate (geometry types + algorithms), `geozero` (zero-copy format translation), `proj` (CRS transforms), and `gdal` (bindings). Our crates build on these.

### Crate Catalog

| # | Crate | Description | Key Dependencies | WASM? |
|---|-------|-------------|-----------------|-------|
| 1 | `geo-validate` | Geometry + attribute validation with structured error reports | geo, geozero, serde, thiserror | Yes |
| 2 | `geo-schema` | YAML schema contracts for geospatial datasets | serde, serde_yaml, geo | Yes |
| 3 | `geo-inventory` | Scan and profile spatial datasets (GeoPackage, GeoParquet, Shapefile) | gdal, arrow, geo, serde | No |
| 4 | `geo-health` | Data staleness and health classification | chrono, serde, walkdir | Yes |
| 5 | `geo-convert` | Format conversion with validation (GPKG ↔ GeoParquet ↔ Shapefile ↔ GeoJSON) | gdal, geozero, arrow | No |
| 6 | `geo-accessibility` | WCAG 2.1 contrast ratios + CVD simulation | (no deps — pure math) | Yes |
| 7 | `kartocolors` | Accessible cartographic color palettes | serde | Yes |
| 8 | `geo-diff` | Diff two spatial datasets by key field | geo, geozero, serde | Yes |
| 9 | `geo-change` | Raster change detection (difference, ratio, classification) | gdal, ndarray | No |
| 10 | `geoai-parse` | Coordinate parsing (DMS, UTM, MGRS, DD) + WKT/GeoJSON generation | geo, regex | Yes |

### Rust Crate Structure

```
geo-validate/
├── Cargo.toml
├── src/
│   ├── lib.rs              # Public API re-exports
│   ├── checks/
│   │   ├── mod.rs
│   │   ├── geometry.rs     # is_valid, no_null_geom, min_vertices
│   │   ├── attributes.rs   # no_nulls, unique, domain
│   │   └── topology.rs     # no_overlaps, no_gaps
│   ├── report.rs           # ValidationReport struct + Display impl
│   ├── error.rs            # thiserror error types
│   └── bin/
│       └── main.rs         # CLI (behind "cli" feature flag)
├── tests/
│   ├── integration_test.rs
│   └── fixtures/
│       ├── valid.geojson
│       └── invalid.geojson
├── benches/
│   └── validation_bench.rs # criterion benchmarks
├── CLAUDE.md
├── ai-dev/
│   └── architecture.md
└── README.md
```

### Cargo.toml Template

```toml
[package]
name = "geo-validate"
version = "0.1.0"
edition = "2021"
authors = ["Chris Lyons <chris@github.com/chrislyonsKY>"]
license = "MIT OR Apache-2.0"
description = "Geometry and attribute validation for geospatial datasets with structured error reports"
repository = "https://github.com/chrislyonsKY/geo-validate"
homepage = "https://github.com/chrislyonsKY"
keywords = ["geospatial", "validation", "gis", "geometry", "quality"]
categories = ["science::geo", "command-line-utilities"]
readme = "README.md"

[dependencies]
geo = "0.28"
geozero = { version = "0.13", optional = true }
serde = { version = "1", features = ["derive"] }
thiserror = "2"
clap = { version = "4", features = ["derive"], optional = true }

[features]
default = []
cli = ["clap", "geozero"]
wasm = []  # Excludes GDAL-dependent code paths

[dev-dependencies]
criterion = "0.5"
geojson = "0.24"

[[bench]]
name = "validation_bench"
harness = false

[[bin]]
name = "geo-validate"
required-features = ["cli"]
```

### Rust-Specific Conventions

- **Dual license:** MIT OR Apache-2.0 (Rust ecosystem standard)
- **Error handling:** `thiserror` for library errors, `anyhow` in CLI binaries only
- **Serialization:** `serde` derive on all public types for JSON/YAML/TOML interop
- **WASM:** Crates marked "Yes" compile to `wasm32-unknown-unknown` with `wasm-bindgen` exports behind a `wasm` feature flag. GDAL-dependent code excluded via `#[cfg(not(target_arch = "wasm32"))]`
- **CLI:** Behind `cli` feature flag, built with `clap` derive. `cargo install geo-validate` installs the CLI. `cargo add geo-validate` gets the library only.
- **Benchmarks:** `criterion` for performance-critical crates (validation, change detection, coordinate parsing)
- **Documentation:** `cargo doc` with `#![doc = include_str!("../README.md")]` for crate-level docs

---

## 5. NuGet — .NET Packages (8)

> **Registry:** nuget.org  
> **Conventions:** .NET 8, C# 12, nullable reference types, ArcGIS Pro SDK 3.x patterns  
> **CI:** GitHub Actions (dotnet build, dotnet test, dotnet pack, dotnet nuget push)  
> **Key advantage:** Direct ArcGIS Pro add-in integration, enterprise geodatabase access

### Design Approach

NuGet packages fall into two categories: (1) **standalone .NET libraries** that work in any .NET project (console, ASP.NET, MAUI), and (2) **ArcGIS Pro SDK companion libraries** that provide reusable components for add-in development. Both follow modern C# idioms: nullable reference types, `IAsyncEnumerable`, `System.Text.Json` serialization, `Microsoft.Extensions.Logging` integration.

The ArcGIS Pro packages reference `ArcGIS.Desktop.*` NuGet packages and provide abstract base classes, utilities, and patterns that accelerate add-in development. They don't ship full add-ins — they ship the reusable building blocks.

### Package Catalog

| # | Package | Description | Target | Key Dependencies |
|---|---------|-------------|--------|-----------------|
| 1 | `ChrisLyons.GeoValidate` | Geometry + attribute validation for NetTopologySuite geometries | .NET 8 (standalone) | NetTopologySuite, YamlDotNet |
| 2 | `ChrisLyons.SchemaEnforce` | YAML schema contracts for spatial datasets | .NET 8 (standalone) | NetTopologySuite, YamlDotNet |
| 3 | `ChrisLyons.GeoHealth` | Data staleness and health monitoring | .NET 8 (standalone) | — |
| 4 | `ChrisLyons.MapAccessibility` | WCAG 2.1 contrast ratios + CVD simulation | .NET 8 (standalone) | — |
| 5 | `ChrisLyons.KartoColors` | Accessible cartographic color palettes | .NET 8 (standalone) | — |
| 6 | `ChrisLyons.GeoAI` | Coordinate parsing + geometry description for LLM context | .NET 8 (standalone) | NetTopologySuite |
| 7 | `ChrisLyons.ArcPro.Patterns` | Reusable ArcGIS Pro add-in patterns (MVVM base classes, GP wrapper, edit operations) | .NET 8 + ArcGIS Pro SDK 3.x | ArcGIS.Desktop.* |
| 8 | `ChrisLyons.ArcPro.Accessibility` | ArcGIS Pro symbology accessibility checker (WCAG layer audit) | .NET 8 + ArcGIS Pro SDK 3.x | ArcGIS.Desktop.*, ChrisLyons.MapAccessibility |

### NuGet Project Structure

```
ChrisLyons.GeoValidate/
├── ChrisLyons.GeoValidate.sln
├── src/
│   └── ChrisLyons.GeoValidate/
│       ├── ChrisLyons.GeoValidate.csproj
│       ├── Checks/
│       │   ├── IValidationCheck.cs       # Check interface
│       │   ├── GeometryValidCheck.cs
│       │   ├── NullGeometryCheck.cs
│       │   ├── AttributeNullCheck.cs
│       │   ├── DomainCheck.cs
│       │   └── CrsCheck.cs
│       ├── Models/
│       │   ├── ValidationResult.cs        # Result record
│       │   ├── ValidationReport.cs        # Aggregated report
│       │   └── SchemaContract.cs          # YAML deserialization target
│       ├── ValidationRunner.cs            # Orchestrator
│       └── Extensions/
│           └── GeometryExtensions.cs      # NTS extension methods
├── tests/
│   └── ChrisLyons.GeoValidate.Tests/
│       ├── ChrisLyons.GeoValidate.Tests.csproj
│       ├── Checks/
│       │   ├── GeometryValidCheckTests.cs
│       │   └── AttributeNullCheckTests.cs
│       └── Fixtures/
│           ├── valid.geojson
│           └── invalid.geojson
├── CLAUDE.md
├── ai-dev/
│   └── architecture.md
└── README.md
```

### .csproj Template

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <LangVersion>12.0</LangVersion>

    <PackageId>ChrisLyons.GeoValidate</PackageId>
    <Version>0.1.0</Version>
    <Authors>Chris Lyons</Authors>
    <Company>Chris Lyons</Company>
    <Description>Geometry and attribute validation for geospatial datasets using NetTopologySuite</Description>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageProjectUrl>https://github.com/chrislyonsKY/geo-validate-dotnet</PackageProjectUrl>
    <RepositoryUrl>https://github.com/chrislyonsKY/geo-validate-dotnet</RepositoryUrl>
    <PackageTags>geospatial;validation;gis;geometry;quality;nts</PackageTags>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <PackageIcon>icon.png</PackageIcon>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="NetTopologySuite" Version="2.*" />
    <PackageReference Include="NetTopologySuite.IO.GeoJSON" Version="4.*" />
    <PackageReference Include="YamlDotNet" Version="16.*" />
  </ItemGroup>

  <ItemGroup>
    <None Include="../../README.md" Pack="true" PackagePath="" />
    <None Include="../../assets/icon.png" Pack="true" PackagePath="" />
  </ItemGroup>
</Project>
```

### NuGet-Specific Conventions

- **Namespace:** `ChrisLyons.{PackageName}` (e.g., `ChrisLyons.GeoValidate.Checks`)
- **Nullable reference types:** Enabled globally, all public APIs annotated
- **Records:** Use `record` types for immutable result objects
- **Interfaces:** All validation checks implement `IValidationCheck` for composability
- **Async:** I/O-bound operations use `async Task<T>` and `IAsyncEnumerable<T>`
- **Logging:** `Microsoft.Extensions.Logging.ILogger<T>` injection (not Console.WriteLine)
- **ArcGIS Pro packages:** Target `net8.0-windows` (Pro is Windows-only), reference SDK via NuGet
- **Source Link:** Enabled for debugging into NuGet package source
- **Strong naming:** Not required (only Microsoft-ecosystem libraries typically need this)

---

## 6. npm — JavaScript/TypeScript Packages (12)

> **Registry:** npmjs.com  
> **Conventions:** TypeScript strict mode, ESM exports, Vitest, tree-shakeable  
> **CI:** GitHub Actions (tsc, vitest, npm publish)  
> **Scope:** `@chrislyonsKY/` (npm org scope)  
> **Key advantage:** Browser + Node.js + serverless, Turf.js ecosystem, MapLibre GL JS integration

### Design Approach

npm packages target three runtimes: (1) **browser** (MapLibre GL JS, Leaflet, deck.gl), (2) **Node.js** (serverless functions, CLI tools, ETL scripts), and (3) **edge/serverless** (Cloudflare Workers, Vercel Edge Functions). All packages ship ESM with full TypeScript declarations. Browser-targeted packages are tree-shakeable and bundle-friendly.

The JavaScript geospatial ecosystem revolves around GeoJSON as the interchange format — every function accepts and returns GeoJSON `Feature` / `FeatureCollection` types. For raster work, packages integrate with `geotiff.js` and `loaders.gl`.

### Package Catalog

| # | Package | Description | Runtime | Key Dependencies |
|---|---------|-------------|---------|-----------------|
| 1 | `@chrislyonsKY/geo-validate` | GeoJSON geometry + property validation | Browser + Node | `@turf/helpers`, `@turf/boolean-valid` |
| 2 | `@chrislyonsKY/schema-enforce` | YAML/JSON schema contracts for GeoJSON | Browser + Node | `yaml`, `ajv` |
| 3 | `@chrislyonsKY/map-accessibility` | WCAG 2.1 contrast + CVD simulation | Browser + Node | (zero deps) |
| 4 | `@chrislyonsKY/kartocolors` | Accessible carto palettes (CSS, JS, MapLibre GL styles) | Browser + Node | (zero deps) |
| 5 | `@chrislyonsKY/geo-diff` | Diff two GeoJSON FeatureCollections by key property | Browser + Node | `@turf/helpers` |
| 6 | `@chrislyonsKY/geoai-parse` | Coordinate parsing (DMS, UTM, MGRS → GeoJSON Point) | Browser + Node | (zero deps) |
| 7 | `@chrislyonsKY/pmtiles-utils` | PMTiles header reading, tile extraction, bbox queries | Browser + Node | `pmtiles` |
| 8 | `@chrislyonsKY/overture-fetch` | Fetch Overture Maps data for a bbox (via PMTiles or DuckDB-WASM) | Browser + Node | `pmtiles`, `@duckdb/duckdb-wasm` (optional) |
| 9 | `@chrislyonsKY/stac-client` | Tidy STAC search with typed results | Node (+ browser fetch) | (zero deps — native fetch) |
| 10 | `@chrislyonsKY/maplibre-accessibility` | MapLibre GL JS plugin: live palette audit overlay | Browser | `maplibre-gl`, `@chrislyonsKY/map-accessibility` |
| 11 | `@chrislyonsKY/geo-render` | Server-side static map rendering (PNG/SVG) for Node.js | Node | `sharp`, `@turf/helpers`, `d3-geo` |
| 12 | `@chrislyonsKY/format-bridge` | Convert between GeoJSON, GeoPackage, Shapefile, CSV (Node.js) | Node | `gdal-async`, `papaparse` |

### npm Package Structure

```
geo-validate/
├── package.json
├── tsconfig.json
├── tsconfig.build.json
├── vitest.config.ts
├── src/
│   ├── index.ts            # Public API re-exports
│   ├── checks/
│   │   ├── geometry.ts     # isValid, noNullGeometry, minVertices
│   │   ├── attributes.ts   # noNulls, unique, domain
│   │   └── topology.ts     # noOverlaps
│   ├── report.ts           # ValidationReport type + builder
│   └── types.ts            # Shared TypeScript interfaces
├── dist/                   # Built output (ESM + CJS + .d.ts)
│   ├── index.js
│   ├── index.cjs
│   └── index.d.ts
├── tests/
│   ├── geometry.test.ts
│   ├── attributes.test.ts
│   └── fixtures/
│       ├── valid.geojson
│       └── invalid.geojson
├── CLAUDE.md
├── ai-dev/
│   └── architecture.md
└── README.md
```

### package.json Template

```json
{
  "name": "@chrislyonsKY/geo-validate",
  "version": "0.1.0",
  "description": "GeoJSON geometry and property validation with structured error reports",
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "test": "vitest run",
    "lint": "eslint src/ --ext .ts",
    "typecheck": "tsc --noEmit"
  },
  "keywords": ["geospatial", "validation", "geojson", "gis", "quality"],
  "author": "Chris Lyons <chris@github.com/chrislyonsKY>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/chrislyonsKY/geo-validate-js"
  },
  "homepage": "https://github.com/chrislyonsKY"
}
```

### npm-Specific Conventions

- **Scope:** All packages under `@chrislyonsKY/` org scope
- **Module format:** Dual ESM + CJS via `tsup`, TypeScript declarations included
- **Zero-dep preference:** Browser-targeted packages aim for zero dependencies where possible
- **GeoJSON types:** Use `@types/geojson` for `Feature`, `FeatureCollection`, `Geometry` types
- **Tree-shaking:** Named exports only, no default exports, no side effects
- **Bundle size:** Track via `size-limit` in CI; browser packages must stay under 10KB gzipped (excluding Turf)
- **Testing:** Vitest (compatible with Jest API, native ESM support)
- **Docs:** TypeDoc for API reference, auto-published to GitHub Pages
- **MapLibre plugins:** Follow MapLibre plugin conventions (`IControl` interface, `map.addControl()`)

---

## Cross-Ecosystem Concept Matrix

This matrix shows which concepts ship in which ecosystems. Not every concept needs every language — we only build where there's genuine demand and ecosystem fit.

| Concept | PyPI | CRAN | QGIS | Rust | NuGet | npm |
|---------|------|------|------|------|-------|-----|
| **Geo Validation** | `spatial-validate` | `spatialvalidate` | `nil-validate` | `geo-validate` | `ChrisLyons.GeoValidate` | `@chrislyonsKY/geo-validate` |
| **Schema Contracts** | `schema-enforcer` | `schemaenforcer` | `nil-validate` | `geo-schema` | `ChrisLyons.SchemaEnforce` | `@chrislyonsKY/schema-enforce` |
| **Data Inventory** | `geoinventory` | `geoinventoryr` | `nil-inventory` | `geo-inventory` | — | — |
| **Data Health** | `geodata-health` | `geodatahealth` | `nil-inventory` | `geo-health` | `ChrisLyons.GeoHealth` | — |
| **Format Conversion** | `format-bridge` | `formatbridge` | `nil-convert` | `geo-convert` | — | `@chrislyonsKY/format-bridge` |
| **Spatial ETL** | `spatial-etl` | `spatialetl` | — | — | — | — |
| **WCAG Accessibility** | `map-accessibility` | `mapaccessibility` | `nil-accessibility` | `geo-accessibility` | `ChrisLyons.MapAccessibility` | `@chrislyonsKY/map-accessibility` |
| **Carto Palettes** | — | `kartocolors` | — | `kartocolors` | `ChrisLyons.KartoColors` | `@chrislyonsKY/kartocolors` |
| **Spatial Diff** | `spatial-diff` | `geochange` | `nil-change` | `geo-diff` | — | `@chrislyonsKY/geo-diff` |
| **Change Detection** | — | `geochange` | `nil-change` | `geo-change` | — | — |
| **Coordinate Parsing** | `geoai-toolkit` | `geoaitools` | — | `geoai-parse` | `ChrisLyons.GeoAI` | `@chrislyonsKY/geoai-parse` |
| **LLM Geo Context** | `geoai-toolkit` | `geoaitools` | — | — | `ChrisLyons.GeoAI` | — |
| **Watershed Tools** | `watershed-tools` | `watershedtools` | `nil-watershed` | — | — | — |
| **Terrain Analysis** | `terrain-extract` | `aboveR` | `nil-terrain` | — | — | — |
| **Cloud-Native I/O** | — | `cloudgeo` | — | — | — | — |
| **Overture Maps** | — | `overturer` | `nil-overture` | — | — | `@chrislyonsKY/overture-fetch` |
| **STAC Workflows** | — | `stacr` | — | — | — | `@chrislyonsKY/stac-client` |
| **DuckDB Spatial** | — | `duckgeo` | — | — | — | — |
| **PMTiles** | — | `pmtilesr` | — | — | — | `@chrislyonsKY/pmtiles-utils` |
| **Env Data APIs** | — | `envdatar` | — | — | — | — |
| **Map Rendering** | — | `georender` | — | — | — | `@chrislyonsKY/geo-render` |
| **ArcGIS Pro Patterns** | `arcpy-etl-patterns` | — | — | — | `ChrisLyons.ArcPro.Patterns` | — |
| **ArcGIS Pro A11y** | — | — | — | — | `ChrisLyons.ArcPro.Accessibility` | — |
| **MapLibre A11y Plugin** | — | — | — | — | — | `@chrislyonsKY/maplibre-accessibility` |
| **FGDC Metadata** | `fgdc-metadata` | — | — | — | — | — |
| **ArcPy Notify** | `arcpy-notify` | — | — | — | — | — |
| **SDE Inspector** | `sde-inspector` | — | — | — | — | — |
| **LYRX Tools** | `lyrx-tools` | — | — | — | — | — |
| **Permit Lifecycle** | `permit-lifecycle` | — | — | — | — | — |
| **Georeferencing** | `georeg` | — | — | — | — | — |
| **Project Scaffold** | `gis-project-scaffold` | — | — | — | — | — |

### Legend

- Filled cells = package planned or in development
- `—` = Not applicable or no ecosystem demand for this concept

---

## Build Priority (Cross-Ecosystem)

### Phase 1: Foundation (Pure-math, zero/low deps, ship to all ecosystems fast)

| Priority | Concept | Ship To | Rationale |
|----------|---------|---------|-----------|
| 1a | **WCAG Accessibility** | PyPI, CRAN, Rust, NuGet, npm | Pure math, zero deps in some ecosystems, shared test vectors, Section 508 credibility |
| 1b | **Kartocolors Palettes** | CRAN, Rust, npm, NuGet | Palette hex arrays are trivial to port, high-visibility demos |
| 1c | **Coordinate Parsing** | PyPI, CRAN, Rust, NuGet, npm | Regex/grammar-based, easy to validate, immediately useful |

### Phase 2: Validation (Core value prop, builds on Phase 1)

| Priority | Concept | Ship To | Rationale |
|----------|---------|---------|-----------|
| 2a | **Geo Validation** | PyPI, CRAN, QGIS, Rust, NuGet, npm | The "flagship" concept — ships everywhere |
| 2b | **Schema Contracts** | PyPI, CRAN, QGIS, Rust, NuGet, npm | Shared YAML contracts are the cross-ecosystem superpower |
| 2c | **Spatial Diff** | PyPI, CRAN, Rust, npm | Broadly useful, moderate complexity |

### Phase 3: Cloud-Native & Data (R-first, then expand)

| Priority | Concept | Ship To | Rationale |
|----------|---------|---------|-----------|
| 3a | **Overturer** | CRAN, npm | First-mover advantage, massive dataset |
| 3b | **Cloud-Native I/O** | CRAN | R-specific gap |
| 3c | **PMTiles** | CRAN, npm | Unoccupied niches in both ecosystems |
| 3d | **STAC Client** | CRAN, npm | Tidy wrappers over existing lower-level libraries |
| 3e | **Env Data APIs** | CRAN | Domain-specific, your community |

### Phase 4: Processing & Domain (Requires sample data, domain expertise)

| Priority | Concept | Ship To | Rationale |
|----------|---------|---------|-----------|
| 4a | **Terrain / aboveR** | CRAN, QGIS | KyFromAbove integration, mining regulatory niche |
| 4b | **Watershed Tools** | PyPI, CRAN, QGIS | NHDPlus workflows |
| 4c | **Change Detection** | CRAN, Rust, QGIS | Raster analysis, multi-temporal |
| 4d | **Map Rendering** | CRAN, npm | Publication maps (R), server-side render (Node) |

### Phase 5: Enterprise & Platform-Specific

| Priority | Concept | Ship To | Rationale |
|----------|---------|---------|-----------|
| 5a | **ArcGIS Pro Patterns** | NuGet | Directly serves your add-in development |
| 5b | **ArcGIS Pro A11y** | NuGet | Builds on AccessibilityAuditor work |
| 5c | **MapLibre A11y Plugin** | npm | Browser-side live accessibility audit |
| 5d | **ArcPy-specific packages** | PyPI | `arcpy-etl-patterns`, `sde-inspector`, `arcpy-notify` — ArcGIS Pro only |

---

## Shared Infrastructure

### Shared Test Fixtures

A `chrislyonsKY/test-fixtures` repo containing:
- GeoJSON files (valid, invalid, edge cases) used by validation packages across all languages
- YAML schema contract files used by schema-enforcer packages
- Color palette definitions (JSON) used by kartocolors packages
- Coordinate strings for parser test vectors
- Small sample rasters (GeoTIFF) for change detection tests

### Shared YAML Contract Spec

The schema contract YAML format is the same across all ecosystems:

```yaml
# schema-contract.yml — enforced by schemaenforcer (R), schema-enforcer (Python),
# geo-schema (Rust), ChrisLyons.SchemaEnforce (C#), @chrislyonsKY/schema-enforce (TS)
name: mining_permits
version: 1.0
geometry:
  type: [Polygon, MultiPolygon]
  crs: "EPSG:4326"
  valid: true
  no_null: true
fields:
  PERMIT_ID:
    type: character
    required: true
    unique: true
  STATUS:
    type: character
    required: true
    domain: [Active, Inactive, Revoked, Pending]
  ISSUE_DATE:
    type: Date
    required: true
  ACREAGE:
    type: numeric
    min: 0
    max: 100000
```

### Branding Per Ecosystem

| Ecosystem | Visual Asset | Location | Format |
|-----------|-------------|----------|--------|
| PyPI | README banner + PyPI logo | `assets/banner.png`, `assets/logo.png` | 1280×320, 512×512 |
| CRAN | Hex sticker | `man/figures/logo.png` | hexb.in spec |
| QGIS | Plugin icon | `icons/icon.svg` | SVG, 64×64 min |
| crates.io | README banner | `assets/banner.png` | 1280×320 |
| NuGet | Package icon | `assets/icon.png` | 128×128 |
| npm | README banner | `assets/banner.png` | 1280×320 |

All use the shared tier color system, JetBrains Mono for package names, line-art icons.

---

## Repository Naming Convention

| Ecosystem | Pattern | Example |
|-----------|---------|---------|
| PyPI | `{package-name}` (kebab-case) | `chrislyonsKY/spatial-validate` |
| CRAN | `{packagename}` (no separators) | `chrislyonsKY/spatialvalidate` |
| QGIS | `nil-{plugin}` | `chrislyonsKY/nil-validate` |
| Rust | `{crate-name}` (kebab-case) | `chrislyonsKY/geo-validate` |
| NuGet | `{Package.Name}-dotnet` | `chrislyonsKY/geo-validate-dotnet` |
| npm | `{package-name}-js` | `chrislyonsKY/geo-validate-js` |

---

## Total Package Count: 77

| Ecosystem | Count |
|-----------|-------|
| PyPI | 20 |
| CRAN | 19 |
| QGIS | 8 |
| crates.io | 10 |
| NuGet | 8 |
| npm | 12 |
| **Total** | **77** |
