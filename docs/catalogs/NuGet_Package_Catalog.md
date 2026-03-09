# Chris Lyons — NuGet Package Catalog

> .NET companion packages to the Chris Lyons Python & R ecosystems  
> **Owner:** Chris Lyons | **Org:** [chrislyonsKY](https://github.com/chrislyonsKY)

-----

## Philosophy

The .NET packages provide the same data quality, validation, and tooling capabilities available in the Python and R catalogs — but with C# idioms, strong typing, and NuGet distribution. No ArcGIS dependency. Usable in console apps, web APIs, MAUI, Blazor, or anywhere .NET runs.

Package names are short and match their Python/R counterparts: `SpatialValidate`, `GeoInventory`, `KartoColors`. PascalCase, no org prefix, no hyphens.

All packages target **.NET 8+** and follow the same tier system (Discovery, ETL, Quality, Processing, Tools) and visual identity as the Python/R ecosystems.

-----

## NuGet Packages (8)

### Tier 1: Data Discovery & Quality

-----

#### 1. `SpatialValidate`

**Geometry and attribute validation for NetTopologySuite geometries and GeoJSON.**

The C# companion to `spatial-validate` (Python) and `spatialvalidate` (R). Composable validation rules that return structured `ValidationResult` objects. Fluent API for building validation pipelines. Integrates with `FluentValidation` conventions.

**Gap it fills:** NTS has `IsValid` on geometries, but .NET has no unified spatial validation framework that checks geometry + attributes + CRS + domains + topology with structured output. Enterprise GIS shops building .NET services do ad-hoc validation everywhere.

**Key types & methods:**

```csharp
// Fluent validation pipeline
var result = new SpatialValidator()
    .CheckValidGeometry()
    .CheckNoNullGeometry()
    .CheckCrs(4326)
    .CheckNoNulls("PERMIT_ID", "STATUS")
    .CheckUnique("PERMIT_ID")
    .CheckDomain("STATUS", "Active", "Inactive", "Pending")
    .CheckTopology(TopologyRule.NoOverlaps)
    .Validate(featureCollection);

// result.IsValid, result.Findings (list of ValidationFinding)
// Each finding: Severity, Rule, Message, FeatureIndex, FieldName

// Fix geometries
var repaired = GeometryFixer.Fix(featureCollection);

// Assertion-style for xUnit/NUnit
features.ShouldPassValidation(v => v
    .CheckValidGeometry()
    .CheckCrs(4326));
```

**Depends:** NetTopologySuite, NetTopologySuite.IO.GeoJSON  
**Suggests:** FluentValidation (optional bridge)  
**NuGet:** Yes (planned)

-----

#### 2. `SchemaEnforce`

**Schema contract enforcement for spatial datasets in .NET.**

Define expected schemas as YAML or JSON, validate feature collections against them. CI-friendly with xUnit/NUnit assertion helpers. Same YAML contract format as the Python and R versions — write once, enforce in any language.

**Gap it fills:** No .NET package provides declarative schema contracts for spatial data with CRS, geometry type, and field domain awareness. `FluentValidation` doesn’t understand spatial schemas.

**Key types & methods:**

```csharp
// Load a shared contract (same YAML as Python/R)
var contract = SchemaContract.FromYaml("schema.yml");

// Validate
var result = contract.Check(featureCollection);
// result.Pass, result.Violations (list of SchemaViolation)
// SchemaViolation: Field, Expected, Actual, ViolationType

// Auto-generate contract from existing data
var generated = SchemaContract.Generate(featureCollection);
generated.ToYaml("schema.yml");

// xUnit assertion
[Fact]
public void Permits_match_schema()
{
    var permits = LoadPermits();
    var contract = SchemaContract.FromYaml("permits_schema.yml");
    contract.ShouldPass(permits);  // throws with violation details
}
```

**Depends:** YamlDotNet, NetTopologySuite  
**NuGet:** Yes (planned)

-----

#### 3. `GeoInventory`

**Scan, document, and diff any spatial data source from .NET.**

C# companion to `geoinventory` (Python) and `geoinventoryr` (R). Scans GeoPackage, Shapefile, GeoJSON, GeoParquet, PostGIS, SQL Server Spatial — produces `InventoryResult` objects with schema, CRS, extent, row counts, and field profiling. Outputs to JSON, Markdown, or HTML.

**Gap it fills:** .NET has no spatial data estate scanner. Enterprise shops managing Oracle/SQL Server spatial databases alongside file-based data have zero tooling for automated inventory.

**Key types & methods:**

```csharp
// Scan a data source
var inventory = await GeoScanner.ScanAsync("data/permits.gpkg");
// inventory.Layers → List<LayerInfo> (name, geomType, crs, rowCount, extent, fields)
// inventory.Fields → per-field profiling (nulls, unique count, min/max, domain values)

// Scan a SQL Server spatial database
var inventory = await GeoScanner.ScanAsync(
    "Server=gis-db;Database=SDE;...",
    provider: DataProvider.SqlServer);

// Diff two inventories
var diff = InventoryDiff.Compare(oldInventory, newInventory);
// diff.Added, diff.Removed, diff.SchemaChanged (per-field changes)

// Export
inventory.ToJson("inventory.json");
inventory.ToMarkdown("inventory.md");
```

**Depends:** NetTopologySuite, GDAL (via MaxRev.Gdal.Core or similar), Npgsql (optional), Microsoft.Data.SqlClient (optional)  
**NuGet:** Yes (planned)

-----

#### 4. `GeoDataHealth`

**Data staleness and health monitoring for geodatabases.**

C# companion to `geodata-health` (Python) and `geodatahealth` (R). Scans spatial databases and file collections for last-modified dates, record counts, and metadata completeness. Produces health reports with Active/Stale/Critical/Unknown classification. Same thresholds as Python/R versions.

**Gap it fills:** Enterprise GIS shops using .NET for backend services have no automated way to monitor whether their spatial datasets are current. They build one-off scripts; this standardizes the pattern.

**Key types & methods:**

```csharp
// Scan a geodatabase for health
var health = await HealthScanner.ScanAsync("data/enterprise.gdb",
    thresholds: new StalenessThresholds(
        activeDays: 30,
        staleDays: 90,
        criticalDays: 180));

// health.Layers → List<LayerHealth>
// LayerHealth: Name, LastModified, RowCount, Status (Active/Stale/Critical/Unknown)

// Summary
health.Summary(); // Active: 12, Stale: 3, Critical: 1, Unknown: 0

// Export
health.ToJson("health.json");
health.ToMarkdown("health_report.md");

// ASP.NET health check endpoint
app.MapGet("/health/spatial", () => HealthScanner.ScanAsync("..."));
```

**Depends:** NetTopologySuite, GDAL (optional), Npgsql (optional)  
**NuGet:** Yes (planned)

-----

### Tier 2: ETL & Transformation

-----

#### 5. `FormatBridge`

**One-method format conversion for spatial data in .NET.**

C# companion to `format-bridge` (Python) and `formatbridge` (R). Convert between any GDAL-readable format with CRS reprojection, encoding detection, and geometry validation. The ogr2ogr of .NET with proper error handling and structured results.

**Gap it fills:** GDAL’s C# bindings exist but are painful. OGR’s conversion pipeline wrapped in a clean .NET API with validation and reporting fills a real need for .NET backend services that process spatial data uploads.

**Key types & methods:**

```csharp
// One-liner conversion
var result = await FormatBridge.ConvertAsync(
    input: "permits.shp",
    output: "permits.gpkg",
    options: new ConvertOptions
    {
        TargetCrs = 4326,
        ValidateGeometry = true,
        FixInvalid = true
    });
// result.InputRows, result.OutputRows, result.Warnings, result.Duration

// Validate a completed conversion
var comparison = await FormatBridge.ValidateConversionAsync("input.shp", "output.gpkg");
// comparison.RowCountMatch, comparison.SchemaMatch, comparison.ExtentMatch

// Encoding detection for legacy shapefiles
var encoding = EncodingDetector.Detect("old_data.dbf");
```

**Depends:** MaxRev.Gdal.Core (or GDAL bindings), NetTopologySuite  
**NuGet:** Yes (planned)

-----

### Tier 3: Visualization & Accessibility

-----

#### 6. `MapAccessibility`

**WCAG 2.1 AA color accessibility for cartographic palettes in .NET.**

C# companion to `map-accessibility` (Python) and `mapaccessibility` (R). Same WCAG algorithms, same scoring. Useful in Blazor map apps, WPF/MAUI GIS tools, backend map tile generation, and anywhere .NET renders color for cartographic output.

**Gap it fills:** No .NET package provides WCAG contrast ratio checking + CVD simulation specifically for cartographic use. Generic accessibility libraries exist but don’t understand map-specific requirements like distinguishing adjacent fill colors.

**Key types & methods:**

```csharp
// Contrast ratio
double ratio = AccessibilityChecker.ContrastRatio("#1B4F72", "#FFFFFF"); // 10.5

// WCAG check
bool passes = AccessibilityChecker.MeetsWcag("#1B4F72", "#FFFFFF",
    level: WcagLevel.AA, textSize: TextSize.Normal); // true

// Simulate color vision deficiency
string simulated = CvdSimulator.Simulate("#FF0000", CvdType.Deuteranopia);

// Score an entire palette
var score = PaletteScorer.Score(new[] { "#1B4F72", "#2E86C1", "#AED6F1", "#FDEBD0", "#E74C3C" });
// score.OverallScore (0.0–1.0), score.MinContrastRatio, score.CvdSafeCount

// Pre-built accessible palettes
var colors = AccessiblePalettes.Qualitative(5);
var sequential = AccessiblePalettes.Sequential("blues", 7);
```

**Depends:** (none — pure .NET, System.Drawing.Color only)  
**NuGet:** Yes (planned)

-----

#### 7. `KartoColors`

**Curated cartographic color palettes with built-in accessibility for .NET.**

C# companion to `kartocolors` (R). Same palette definitions, same accessibility guarantees. Provides palette lookup, interpolation, and integration points for Blazor, MAUI, SkiaSharp, and any .NET rendering pipeline.

**Gap it fills:** .NET has no cartography-specific palette library. Developers hard-code hex colors or copy ColorBrewer values without accessibility testing. This provides the same curated, pre-tested palettes available in the R version.

**Key types & methods:**

```csharp
// Get a palette
var colors = KartoPalette.Get("terrain", 7);  // string[] of hex colors
var hydro = KartoPalette.Get("hydro", 5);

// List all palettes with metadata
var palettes = KartoPalette.All();
// Each: Name, Category, MinN, MaxN, AccessibilityScore, CvdSafe

// Interpolate for continuous data
var color = KartoPalette.Interpolate("sequential_blues", 0.65); // value 0.0–1.0

// SkiaSharp integration
SKColor[] skColors = KartoPalette.GetSk("terrain", 7);

// Blazor CSS helper
string css = KartoPalette.ToCssVariables("terrain", 7);
// --karto-0: #1B4F72; --karto-1: #2E86C1; ...
```

**Depends:** (none — pure .NET)  
**Suggests:** SkiaSharp (optional bridge)  
**NuGet:** Yes (planned)

-----

### Tier 5: Tools

-----

#### 8. `GeoAI`

**LLM utility functions for geospatial workflows in .NET.**

C# companion to `geoai-toolkit` (Python) and `geoaitools` (R). Coordinate parsing, feature collection description for LLM context windows, and prompt template management. Essential for .NET services that bridge LLMs with spatial data (Semantic Kernel, Azure OpenAI).

**Gap it fills:** Zero .NET packages exist for bridging LLMs with spatial data. The same coordinate parsing and dataset description capabilities as the Python version, but with strong typing and Semantic Kernel plugin integration.

**Key types & methods:**

```csharp
// Parse any coordinate format
var coord = CoordinateParser.Parse("38°02'53\"N 84°30'04\"W");
// coord.Latitude, coord.Longitude, coord.Format (DMS)

var coord2 = CoordinateParser.Parse("16S 722714 4213456");
// coord2.Latitude, coord2.Longitude, coord2.Format (UTM)

// Describe a feature collection for LLM context
string description = GeoDescriber.Describe(featureCollection);
// "FeatureCollection: 1,247 Polygon features, EPSG:4326,
//  Extent: [-84.50, 38.00, -84.30, 38.20],
//  Fields: PERMIT_ID (string, 1247 unique), STATUS (string, 3 unique: Active, Inactive, Pending)..."

// Coordinate conversion
var dd = CoordinateConverter.DmsToDecimal(38, 2, 53, CardinalDirection.North);
var dms = CoordinateConverter.DecimalToDms(38.048);

// Semantic Kernel plugin (optional)
kernel.Plugins.Add(new GeoAIPlugin());
// Exposes: ParseCoordinates, DescribeDataset, ConvertCoordinates as SK functions
```

**Depends:** NetTopologySuite (optional, for Describe)  
**Suggests:** Microsoft.SemanticKernel (optional plugin)  
**NuGet:** Yes (planned)

-----

## Cross-Language Companion Matrix

|Domain             |Python             |R                 |NuGet             |
|-------------------|-------------------|------------------|------------------|
|Data inventory     |`geoinventory`     |`geoinventoryr`   |`GeoInventory`    |
|Validation         |`spatial-validate` |`spatialvalidate` |`SpatialValidate` |
|Schema enforcement |`schema-enforcer`  |`schemaenforcer`  |`SchemaEnforce`   |
|Data health        |`geodata-health`   |`geodatahealth`   |`GeoDataHealth`   |
|ETL pipelines      |`spatial-etl`      |`spatialetl`      |—                 |
|Format conversion  |`format-bridge`    |`formatbridge`    |`FormatBridge`    |
|Color accessibility|`map-accessibility`|`mapaccessibility`|`MapAccessibility`|
|Carto palettes     |—                  |`kartocolors`     |`KartoColors`     |
|GeoAI/LLM          |`geoai-toolkit`    |`geoaitools`      |`GeoAI`           |

### Shared Contracts

The schema enforcement YAML format is identical across all three languages. A `permits_schema.yml` written for the Python version validates the same dataset in R or .NET. This is a deliberate design choice — schema contracts are infrastructure, not language-specific.

Similarly, `KartoColors` palette definitions are stored as JSON and shared across R and .NET. One palette definition, three language integrations.

-----

## Dependency Graph

```
SpatialValidate ──── NetTopologySuite
        │
        ├── SchemaEnforce ──── YamlDotNet
        │
        └── GeoInventory ──── GDAL, Npgsql (optional)
                │
                └── GeoDataHealth

FormatBridge ──── GDAL, NetTopologySuite

MapAccessibility ──── (none, pure .NET)
        │
        └── KartoColors ──── (none, pure .NET)

GeoAI ──── NetTopologySuite (optional)
```

-----

## Visual Identity — NuGet Icons

NuGet packages use a **128 × 128px square icon** (NuGet gallery spec). Follow the same brand system:

|Asset                    |Size                     |Where Used                                      |
|-------------------------|-------------------------|------------------------------------------------|
|**NuGet icon**           |128 × 128px (square, PNG)|NuGet.org gallery, Visual Studio package manager|
|**README banner**        |1280 × 320px             |Top of README.md (same format as Python)        |
|**GitHub social preview**|1280 × 640px             |Link previews                                   |

**NuGet icon design:** Dark background (tier color), centered line-art icon (same icons as Python/R companions), package shortname below icon in JetBrains Mono. No text below 8px — NuGet thumbnails are tiny.

|#|Package         |Tier     |Icon (shared with)       |Border/Accent  |
|-|----------------|---------|-------------------------|---------------|
|1|SpatialValidate |Quality  |Polygon + checkmark      |Rose `#F43F5E` |
|2|SchemaEnforce   |Quality  |Shield + YAML + lock     |Rose `#F43F5E` |
|3|GeoInventory    |Discovery|Magnifying glass + layers|Teal `#14B8A6` |
|4|GeoDataHealth   |Discovery|Heartbeat EKG line       |Teal `#14B8A6` |
|5|FormatBridge    |ETL      |Files + bridge arch      |Amber `#F59E0B`|
|6|MapAccessibility|Quality  |Eye + contrast iris      |Rose `#F43F5E` |
|7|KartoColors     |Quality  |Palette + color dots     |Rose `#F43F5E` |
|8|GeoAI           |Tools    |Brain + map pin          |Sky `#0EA5E9`  |

-----

## .NET Package Structure Template

Every NuGet package follows this structure:

```
PackageName/
├── CLAUDE.md                        # AI-dev context
├── README.md                        # NuGet readme (rendered on NuGet.org)
├── LICENSE                          # MIT
├── PackageName.sln
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                   # Build + test on push/PR
│   │   └── publish.yml              # Publish to NuGet on tag
│   └── copilot-instructions.md
├── ai-dev/
│   ├── architecture.md
│   ├── spec.md
│   ├── patterns.md
│   ├── agents/
│   │   ├── architect.md
│   │   └── csharp_expert.md
│   ├── decisions/
│   │   └── DL-001-*.md
│   └── guardrails/
│       ├── coding-standards.md
│       └── nuget-compliance.md
├── src/
│   └── PackageName/
│       ├── PackageName.csproj
│       ├── PackageName.cs           # Public API surface
│       └── Internal/                # Internal implementation
├── tests/
│   └── PackageName.Tests/
│       ├── PackageName.Tests.csproj
│       └── PackageNameTests.cs
├── samples/                         # Sample console app or usage examples
│   └── PackageName.Samples/
├── docs/
│   └── api.md                       # API reference
├── assets/
│   ├── icon.png                     # 128×128 NuGet icon
│   ├── banner.png                   # 1280×320 README banner
│   └── social-preview.png           # 1280×640 GitHub social
└── CHANGELOG.md
```

### .csproj Template

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>

    <!-- NuGet metadata -->
    <PackageId>PackageName</PackageId>
    <Version>0.1.0</Version>
    <Authors>Chris Lyons</Authors>
    <Company>Chris Lyons</Company>
    <Description>Package description here.</Description>
    <PackageTags>geospatial;gis;spatial;validation</PackageTags>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageProjectUrl>https://github.com/chrislyonsKY/PackageName</PackageProjectUrl>
    <RepositoryUrl>https://github.com/chrislyonsKY/PackageName</RepositoryUrl>
    <PackageIcon>icon.png</PackageIcon>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>

  <ItemGroup>
    <None Include="../../README.md" Pack="true" PackagePath="" />
    <None Include="../../assets/icon.png" Pack="true" PackagePath="" />
  </ItemGroup>
</Project>
```

### GitHub Actions CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - run: dotnet restore
      - run: dotnet build --no-restore --configuration Release
      - run: dotnet test --no-build --configuration Release --verbosity normal
```

-----

## NuGet Publishing Guardrails

> Equivalent to the CRAN compliance guardrails for R packages.

### Pre-Publish Checklist

```
[ ] dotnet build --configuration Release — 0 errors, 0 warnings (TreatWarningsAsErrors)
[ ] dotnet test — all tests pass on Linux, macOS, Windows
[ ] All public types and members have XML doc comments
[ ] README.md is informative (rendered on NuGet.org)
[ ] CHANGELOG.md is current
[ ] icon.png is 128×128, under 1MB
[ ] Version follows SemVer (x.y.z, no pre-release suffix for stable)
[ ] PackageTags include relevant discovery terms
[ ] License is MIT
[ ] No hardcoded file paths, connection strings, or credentials
[ ] All file I/O uses caller-provided paths or system temp
[ ] Nullable reference types enabled, no suppressions without justification
[ ] SourceLink configured for debuggable NuGet packages
[ ] Strong naming configured (if targeting enterprise consumers)
```

### Common Issue Patterns

|Issue                                 |Prevention                                                                         |
|--------------------------------------|-----------------------------------------------------------------------------------|
|Missing XML docs on public API        |`<GenerateDocumentationFile>true</GenerateDocumentationFile>` + CI warning-as-error|
|Platform-specific code breaks on Linux|CI matrix includes ubuntu, windows, macos                                          |
|GDAL native dependency fails          |Document SystemRequirements, provide `MaxRev.Gdal.Core` fallback                   |
|Large package size                    |Exclude test data from NuGet pack, use `.nuspec` exclude patterns                  |
|Name collision on NuGet.org           |Check availability before committing to a name                                     |

-----

## Architectural Decisions (Cross-Cutting)

### AD-001: NetTopologySuite as the geometry foundation

All packages that handle geometry use NTS. NTS is the .NET port of JTS (the same lineage as GEOS, which underlies PostGIS and GDAL). It provides a consistent geometry model without requiring GDAL or ArcGIS runtime dependencies.

**Rationale:** NTS is the only production-quality .NET geometry library. It’s used by Entity Framework Core Spatial, NpgsqlNetTopologySuite, and every serious .NET GIS project.

### AD-002: Shared contract formats across languages

YAML schema contracts (`SchemaEnforce`) and palette definitions (`KartoColors`) use identical file formats across Python, R, and .NET. A schema.yml created by the Python version validates in .NET. Palette JSON is the same across R and .NET.

**Rationale:** Enterprise shops use multiple languages. Schema contracts are data infrastructure — they shouldn’t be language-locked.

### AD-003: Pure .NET for accessibility packages

`MapAccessibility` and `KartoColors` have zero external dependencies. They use only `System.Drawing.Color` for color representation. This makes them embeddable in any .NET context — Blazor WASM, MAUI, Unity, console apps — without pulling in NTS or GDAL.

**Rationale:** Accessibility checking is a pure algorithm. No reason to add dependencies. Maximum portability.

### AD-004: No standalone ETL package

The .NET catalog doesn’t include an ETL pipeline package. Generic .NET ETL is well-served by existing libraries (MediatR pipelines, Dataflow blocks, channels, etc.). The Python/R ETL packages fill a real gap in those ecosystems; in .NET the gap doesn’t exist at the same scale.

**Rationale:** Don’t duplicate what the ecosystem already provides.

### AD-005: Short package names, no org prefix

Packages use short names (`SpatialValidate`, not `ChrisLyons.SpatialValidate`). The org is already established via the NuGet author field, GitHub org, and README branding. The short names match their Python/R counterparts and keep `<PackageReference>` lines clean.

**Rationale:** Brevity in `.csproj` files, consistency with the kebab-case (Python) and lowercase (R) naming in sibling ecosystems. The tradeoff is potential NuGet namespace collisions — mitigated by checking availability before committing to names.

-----

## Build Priority

|Phase|Packages                         |Rationale                                                                                                                      |
|-----|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
|**1**|`MapAccessibility`, `KartoColors`|Zero dependencies, pure .NET, high cross-platform value. Direct companions to R Phase 1. Shared palette JSON across R and .NET.|
|**2**|`GeoAI`, `SpatialValidate`       |Unique in .NET ecosystem, direct companions to Python/R Phase 2. GeoAI has Semantic Kernel plugin angle for newsletter content.|
|**3**|`SchemaEnforce`, `GeoInventory`  |Data documentation tooling. SchemaEnforce shares YAML contracts with Python/R — strong cross-language story.                   |
|**4**|`GeoDataHealth`, `FormatBridge`  |Enterprise-focused. GDAL dependency makes these heavier to build and test.                                                     |

### Phase 1 Rationale (Start Here)

`MapAccessibility` and `KartoColors` are the best first NuGet packages for the same reasons they’re Phase 1 in the R catalog:

1. **Zero dependencies** — pure algorithm implementations, no GDAL/NTS complications
1. **Fast to build** — well-defined scope, clear algorithms (WCAG formulas, palette lookup)
1. **Cross-language story** — “same palettes in R, Python, and .NET” is a compelling narrative
1. **sletter content** — accessibility tooling resonates with the Chris Lyons Dispatch audience
1. **Prove the pipeline** — establishes the NuGet publishing workflow, CI templates, and visual identity before tackling complex packages

-----

## Updated Chris Lyons Package Count

|Ecosystem    |Count                           |
|-------------|--------------------------------|
|Python (PyPI)|20                              |
|R (CRAN)     |18 (10 catalog + 8 cloud-native)|
|.NET (NuGet) |**8**                           |
|**Total**    |**46**                          |