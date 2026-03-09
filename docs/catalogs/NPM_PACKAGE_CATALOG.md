# Chris Lyons — npm Package Catalog

> TypeScript packages for web developers building geospatial applications  
> **Owner:** Chris Lyons | **Org:** [@chrislyonsKY](https://www.npmjs.com/org/chrislyonsKY)

-----

## Philosophy

The JavaScript geospatial ecosystem has mature tools for rendering (MapLibre, deck.gl, Leaflet) and analysis (Turf.js). These packages fill the gaps in **validation, accessibility, LLM integration, and developer experience** — areas where web developers building mapping applications lack good tooling. They follow modern TypeScript conventions: tree-shakeable ESM exports, zero/minimal dependencies, and full type safety.

All packages use the `@chrislyonsKY/` npm scope.

-----

## Landscape Analysis

Before building, I surveyed what already exists:

|Domain                 |Existing Solutions                                                                           |Gap                                                                                                                 |
|-----------------------|---------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
|**GeoJSON validation** |`geojson-validation` (2018, basic), `@yaga/geojson-schema` (JSON Schema), scattered utilities|No unified validator with attribute checks, topology validation, and tidy error reports                             |
|**Color accessibility**|`wcag-contrast`, `apca-w3`, `accessible-colors`                                              |Generic web accessibility — nothing cartography-specific (palette scoring, CVD simulation for maps, legend contrast)|
|**LLM + geospatial**   |None in npm                                                                                  |Zero packages for coordinate parsing, GeoJSON description for context windows, spatial prompt templates             |
|**PMTiles**            |`pmtiles` (Protomaps official)                                                               |Excellent for rendering, but no GeoJSON extraction, metadata inspection, or bbox queries                            |
|**COG**                |`geotiff` (mature), `@cogeotiff/core`                                                        |Good readers exist — low priority                                                                                   |
|**Schema contracts**   |`ajv`, `zod`                                                                                 |Generic JSON validation; nothing understands CRS, geometry types, or spatial domains                                |

**Conclusion:** Biggest gaps are in **validation+schema**, **accessibility for maps**, and **LLM utilities**. PMTiles has good rendering support but could use analysis utilities.

-----

## npm Packages (8)

### Tier 1: Validation & Schema

-----

#### 1. `@chrislyonsKY/geojson-validate`

**Composable GeoJSON validation with structured error reports.**

TypeScript companion to `spatialvalidate` (R) and `spatial-validate` (Python). Runs geometry checks (valid, non-null, correct type), attribute checks (nulls, uniques, domains), CRS validation, and basic topology. Returns structured error arrays, not just boolean pass/fail.

**Gap it fills:** `geojson-validation` is unmaintained (2018) and returns only boolean + trace strings. No existing npm package provides composable validation with structured reports suitable for CI pipelines, form validation, or data dashboards.

```typescript
import { validate, checks } from '@chrislyonsKY/geojson-validate';

const result = validate(featureCollection, [
  checks.validGeometry(),
  checks.noNullGeometry(),
  checks.crs('EPSG:4326'),
  checks.noNulls(['name', 'id']),
  checks.unique('id'),
  checks.domain('status', ['active', 'inactive', 'pending']),
  checks.bbox([-180, -90, 180, 90]),
]);

console.log(result.valid); // boolean
console.log(result.errors); 
// [{ check: 'noNulls', field: 'name', featureIndex: 12, message: '...' }]
```

**Key functions:**

- `validate(geojson, checks[])` → `ValidationResult`
- `checks.validGeometry()` — RFC 7946 geometry validation
- `checks.noNullGeometry()` — no null/missing geometries
- `checks.crs(epsg)` — coordinate range validation for known CRS
- `checks.noNulls(fields[])` — required fields
- `checks.unique(field)` — unique constraint
- `checks.domain(field, values[])` — enum validation
- `checks.bbox(bounds)` — features within bounding box
- `checks.topology('no-self-intersect' | 'no-overlaps')` — basic topology
- `fix.geometry(geojson)` — attempt auto-repair via buffer(0)

**Dependencies:** None (pure TypeScript)  
**Size target:** <10KB minified

-----

#### 2. `@chrislyonsKY/geojson-schema`

**Declarative schema contracts for GeoJSON with TypeScript inference.**

Define expected schemas as TypeScript objects or JSON/YAML, validate GeoJSON against them, generate schemas from existing data. Integrates with Zod for runtime validation and TypeScript type generation.

**Gap it fills:** No npm package provides spatial-aware schema contracts. Zod and AJV don’t understand geometry types, CRS, or coordinate precision.

```typescript
import { defineSchema, validateSchema, inferSchema } from '@chrislyonsKY/geojson-schema';

// Define a schema
const permitSchema = defineSchema({
  geometryType: 'Polygon',
  crs: 'EPSG:4326',
  properties: {
    permit_id: { type: 'string', required: true, unique: true },
    status: { type: 'string', enum: ['active', 'expired', 'pending'] },
    area_sqm: { type: 'number', min: 0 },
    issued_date: { type: 'date' },
  },
  coordinatePrecision: 6,
});

// Validate
const result = validateSchema(featureCollection, permitSchema);

// Infer schema from existing data
const inferred = inferSchema(existingFeatureCollection);
console.log(inferred); // { geometryType: 'Polygon', properties: { ... } }

// TypeScript type inference
type Permit = InferFeature<typeof permitSchema>;
// { geometry: Polygon; properties: { permit_id: string; status: 'active' | ... } }
```

**Key functions:**

- `defineSchema(config)` → Schema object
- `validateSchema(geojson, schema)` → ValidationResult
- `inferSchema(geojson)` → inferred Schema
- `schemaToZod(schema)` → Zod schema for property validation
- `schemaToTypeScript(schema)` → TypeScript interface as string
- `loadSchema(path)` — load from JSON/YAML file

**Dependencies:** Optional `zod` peer dependency  
**Size target:** <15KB minified

-----

### Tier 2: Accessibility

-----

#### 3. `@chrislyonsKY/map-a11y`

**WCAG 2.1 AA accessibility for cartographic color palettes.**

TypeScript companion to `mapaccessibility` (R) and `map-accessibility` (Python). Checks contrast ratios, simulates color vision deficiency (CVD), scores palettes for map readability. Understands cartographic context: adjacent polygon contrast, legend readability, small symbol visibility.

**Gap it fills:** Generic WCAG checkers (`wcag-contrast`, `apca-w3`) don’t handle cartographic use cases. No npm package scores palette accessibility for maps or checks adjacent feature contrast.

```typescript
import { 
  contrastRatio, 
  checkWCAG, 
  simulateCVD, 
  scorePalette,
  checkLegendContrast,
  checkAdjacentContrast 
} from '@chrislyonsKY/map-a11y';

// Basic contrast
const ratio = contrastRatio('#1a73e8', '#ffffff'); // 4.51
const passes = checkWCAG('#1a73e8', '#ffffff', 'AA', 'normal'); // true

// CVD simulation
const simulated = simulateCVD(['#ff0000', '#00ff00'], 'deuteranopia');
// Returns how these colors appear to someone with deuteranopia

// Palette scoring (0-1, higher = more accessible)
const score = scorePalette(['#1a73e8', '#ea4335', '#34a853', '#fbbc04']);
// { overall: 0.82, contrastScore: 0.9, cvdScore: 0.75, distinctiveness: 0.8 }

// Map-specific: check legend text against background
const legendOk = checkLegendContrast({
  swatches: ['#1a73e8', '#ea4335'],
  labels: ['Water', 'Fire'],
  labelColor: '#333333',
  backgroundColor: '#ffffff'
});

// Map-specific: check that adjacent polygon classes are distinguishable
const adjacentOk = checkAdjacentContrast(palette, { minRatio: 2.5 });
```

**Key functions:**

- `contrastRatio(fg, bg)` → number (1-21)
- `checkWCAG(fg, bg, level, size)` → boolean
- `simulateCVD(colors, type)` → transformed hex array
- `scorePalette(colors)` → { overall, contrastScore, cvdScore, distinctiveness }
- `checkLegendContrast(config)` → { valid: boolean, issues: [] }
- `checkAdjacentContrast(palette, options)` → { valid: boolean, problematicPairs: [] }
- `suggestAlternative(color, background, level)` → nearest compliant color

**Dependencies:** None  
**Size target:** <12KB minified

-----

#### 4. `@chrislyonsKY/karto-palettes`

**Curated cartographic color palettes with built-in accessibility.**

TypeScript companion to `kartocolors` (R). Every palette is pre-tested for WCAG 2.1 AA and colorblind safety. Includes sequential, diverging, and qualitative schemes optimized for maps. Works with any rendering library.

**Gap it fills:** `d3-scale-chromatic` palettes aren’t all colorblind-safe. No npm package provides cartography-specific palettes with accessibility guarantees.

```typescript
import { 
  palette, 
  palettes, 
  scale,
  getPaletteInfo 
} from '@chrislyonsKY/karto-palettes';

// Get a palette by name
const terrain = palette('terrain', 5); // ['#264653', '#2a9d8f', ...]

// List all palettes with metadata
const all = palettes();
// [{ name: 'terrain', type: 'sequential', wcagScore: 0.95, cvdSafe: true }, ...]

// Create a D3-compatible scale function
const colorScale = scale('diverging-earth', { domain: [-100, 0, 100] });
colorScale(50); // returns hex color

// Get detailed info
const info = getPaletteInfo('terrain');
// { colors: [...], type: 'sequential', wcagScore: 0.95, cvdSafe: true, 
//   usage: 'Elevation, terrain, topographic maps' }

// Palette categories
import { terrain, landuse, hydro, population, environmental, heatmap } from '@chrislyonsKY/karto-palettes';
```

**Palette categories:**

- `terrain` — elevation, hillshade, topographic
- `landuse` — zoning, land cover classification
- `hydro` — water depth, flow, precipitation
- `population` — density, demographics
- `environmental` — pollution, temperature, NDVI
- `diverging` — change maps, above/below threshold
- `qualitative` — categorical distinctions (max 8 classes)

**Dependencies:** None  
**Size target:** <8KB minified (palettes as data)

-----

### Tier 3: LLM & AI Utilities

-----

#### 5. `@chrislyonsKY/geo-llm`

**LLM utility functions for geospatial workflows.**

TypeScript companion to `geoaitools` (R) and `geoai-toolkit` (Python). Parses coordinates from natural language, describes GeoJSON for LLM context windows, and provides prompt templates for spatial tasks.

**Gap it fills:** Zero npm packages bridge LLMs with spatial data. This provides coordinate parsing, feature description, and prompt utilities that the research community is building in Python but hasn’t ported to JS.

```typescript
import { 
  parseCoordinates, 
  describeGeoJSON, 
  spatialPrompt,
  dmsToDecimal,
  decimalToDms 
} from '@chrislyonsKY/geo-llm';

// Parse coordinates from natural language
const coords = parseCoordinates("38°02'53\"N 84°30'04\"W");
// { lat: 38.048, lng: -84.501, format: 'dms', confidence: 0.99 }

const utmCoords = parseCoordinates("16S 722714 4213456");
// { lat: 38.048, lng: -84.501, format: 'utm', zone: '16S', confidence: 0.95 }

// Describe GeoJSON for LLM context (token-efficient)
const description = describeGeoJSON(featureCollection, { maxTokens: 500 });
// "FeatureCollection with 47 Polygon features. Bounds: [-84.5, 38.0, -84.3, 38.2].
//  Properties: permit_id (string, unique), status (enum: active|expired|pending), 
//  area_sqm (number, range: 100-50000). CRS: EPSG:4326."

// Spatial prompt templates
const prompt = spatialPrompt('analyze-change', {
  before: describeGeoJSON(beforeFC),
  after: describeGeoJSON(afterFC),
  question: 'What parcels were subdivided?'
});

// Coordinate conversion utilities
const decimal = dmsToDecimal(38, 2, 53, 'N'); // 38.048
const dms = decimalToDms(38.048); // { deg: 38, min: 2, sec: 52.8, dir: 'N' }
```

**Key functions:**

- `parseCoordinates(text)` → { lat, lng, format, confidence } | null
- `describeGeoJSON(geojson, options)` → string (LLM-friendly description)
- `spatialPrompt(template, vars)` → string
- `dmsToDecimal(deg, min, sec, dir)` → number
- `decimalToDms(decimal)` → { deg, min, sec, dir }
- `estimateTokens(geojson)` → number (approximate token count)

**Prompt templates:**

- `analyze-change` — before/after comparison
- `extract-features` — find features matching criteria
- `spatial-query` — natural language to spatial predicate
- `describe-location` — generate location description

**Dependencies:** None  
**Size target:** <10KB minified

-----

### Tier 4: Cloud-Native Data

-----

#### 6. `@chrislyonsKY/pmtiles-utils`

**PMTiles analysis utilities beyond rendering.**

The official `pmtiles` package is excellent for MapLibre integration. This package adds analysis capabilities: extract GeoJSON from tiles, inspect metadata, query by bbox without rendering, and validate archives.

**Gap it fills:** `pmtiles` focuses on tile serving for MapLibre/Leaflet. No package extracts actual feature data or provides inspection utilities.

```typescript
import { 
  inspect, 
  extractFeatures, 
  queryBbox,
  validateArchive,
  getTileAsGeoJSON 
} from '@chrislyonsKY/pmtiles-utils';

// Inspect archive metadata
const info = await inspect('https://example.com/buildings.pmtiles');
// { 
//   format: 'mvt', 
//   minZoom: 0, maxZoom: 14, 
//   tileCount: 2300000,
//   bounds: [-180, -85, 180, 85],
//   layers: ['buildings', 'roads'],
//   size: '340MB',
//   compression: 'gzip'
// }

// Extract features from a bbox as GeoJSON
const features = await extractFeatures('buildings.pmtiles', {
  bbox: [-84.5, 38.0, -84.3, 38.2],
  zoom: 14,
  layers: ['buildings']
});
// Returns FeatureCollection

// Query without full extraction (returns tile indices)
const tiles = await queryBbox('buildings.pmtiles', {
  bbox: [-84.5, 38.0, -84.3, 38.2],
  zoom: 14
});
// [{ z: 14, x: 4523, y: 6234 }, ...]

// Validate archive structure
const validation = await validateArchive('buildings.pmtiles');
// { valid: true, issues: [], tileCount: 2300000 }

// Get single tile as GeoJSON
const tile = await getTileAsGeoJSON('buildings.pmtiles', { z: 14, x: 4523, y: 6234 });
```

**Key functions:**

- `inspect(url)` → metadata object
- `extractFeatures(url, options)` → FeatureCollection
- `queryBbox(url, options)` → tile index array
- `validateArchive(url)` → validation result
- `getTileAsGeoJSON(url, tileCoord)` → FeatureCollection
- `compareTilesets(url1, url2)` → diff report

**Dependencies:** `pmtiles` (peer), `@mapbox/vector-tile` for MVT decoding  
**Size target:** <20KB minified

-----

#### 7. `@chrislyonsKY/stac-client`

**Lightweight STAC API client with TypeScript types.**

Tidy wrapper around STAC API interactions. Returns typed responses, handles pagination, and provides search helpers. Designed for frontend apps that need to query STAC catalogs without server-side processing.

**Gap it fills:** `stac-js` exists but is heavy and designed for Node. No lightweight browser-first STAC client with good TypeScript types.

```typescript
import { 
  StacClient, 
  searchItems, 
  getCollection,
  catalogs 
} from '@chrislyonsKY/stac-client';

// Quick search (stateless)
const items = await searchItems({
  url: 'https://planetarycomputer.microsoft.com/api/stac/v1',
  collection: 'sentinel-2-l2a',
  bbox: [-84.5, 38.0, -84.0, 38.5],
  datetime: '2024-06-01/2024-08-31',
  query: { 'eo:cloud_cover': { lt: 20 } },
  limit: 10
});
// Returns typed ItemCollection

// Client instance for multiple requests
const client = new StacClient('https://planetarycomputer.microsoft.com/api/stac/v1');
const collections = await client.collections();
const collection = await client.collection('sentinel-2-l2a');

// Pagination helper
for await (const page of client.searchPaginated({ collection: 'naip', bbox })) {
  console.log(page.features.length);
}

// Known catalog registry
const pc = catalogs.planetaryComputer;
const earthSearch = catalogs.earthSearch;
```

**Key functions:**

- `searchItems(options)` → ItemCollection
- `getCollection(url, id)` → Collection
- `getCatalog(url)` → Catalog
- `StacClient` class with methods: `search()`, `collections()`, `collection()`, `item()`
- `catalogs` — registry of known STAC endpoints

**Dependencies:** None (uses fetch)  
**Size target:** <12KB minified

-----

#### 8. `@chrislyonsKY/overture-query`

**Query Overture Maps data from the browser via DuckDB-WASM.**

TypeScript companion to `overturer` (R). Runs DuckDB queries against Overture’s S3-hosted GeoParquet directly in the browser. Returns GeoJSON. Uses DuckDB-WASM for client-side execution with spatial predicate pushdown.

**Gap it fills:** No npm package provides browser-native Overture Maps access. Existing approaches require a backend or downloading entire files.

```typescript
import { 
  OvertureMaps, 
  buildings, 
  places, 
  transportation,
  divisions 
} from '@chrislyonsKY/overture-query';

// Quick query functions
const buildingsFC = await buildings({
  bbox: [-84.5, 38.0, -84.3, 38.2],
  limit: 1000
});
// Returns GeoJSON FeatureCollection

const restaurants = await places({
  bbox: [-84.5, 38.0, -84.3, 38.2],
  categories: ['restaurant', 'cafe']
});

// Full client for complex queries
const client = new OvertureMaps({ release: '2024-08-20' });

const roads = await client.query({
  theme: 'transportation',
  type: 'segment',
  bbox: [-84.5, 38.0, -84.3, 38.2],
  where: "subtype = 'road' AND class IN ('primary', 'secondary')"
});

// Available releases
const releases = await client.releases();
```

**Key functions:**

- `buildings(options)` → FeatureCollection
- `places(options)` → FeatureCollection
- `transportation(options)` → FeatureCollection
- `divisions(options)` → FeatureCollection
- `addresses(options)` → FeatureCollection
- `OvertureMaps` class for advanced queries
- `releases()` → available Overture releases

**Dependencies:** `@duckdb/duckdb-wasm` (peer dependency)  
**Size target:** <25KB minified (excluding DuckDB-WASM)

-----

## Package Summary

|#|Package                             |Tier         |Gap Filled                             |Dependencies      |
|-|------------------------------------|-------------|---------------------------------------|------------------|
|1|`@chrislyonsKY/geojson-validate`|Validation   |Composable GeoJSON validation          |None              |
|2|`@chrislyonsKY/geojson-schema`  |Validation   |Spatial-aware schema contracts         |Optional zod      |
|3|`@chrislyonsKY/map-a11y`        |Accessibility|Cartographic WCAG checking             |None              |
|4|`@chrislyonsKY/karto-palettes`  |Accessibility|Accessible map palettes                |None              |
|5|`@chrislyonsKY/geo-llm`         |LLM/AI       |Coordinate parsing, GeoJSON description|None              |
|6|`@chrislyonsKY/pmtiles-utils`   |Cloud-Native |PMTiles analysis utilities             |pmtiles (peer)    |
|7|`@chrislyonsKY/stac-client`     |Cloud-Native |Lightweight STAC client                |None              |
|8|`@chrislyonsKY/overture-query`  |Cloud-Native |Browser-native Overture access         |duckdb-wasm (peer)|

-----

## Build Priority

|Phase|Packages                      |Rationale                                                              |
|-----|------------------------------|-----------------------------------------------------------------------|
|**1**|`geojson-validate`, `map-a11y`|Zero dependencies, high utility, direct companions to Python/R packages|
|**2**|`karto-palettes`, `geo-llm`   |Zero dependencies, unique in npm ecosystem, newsletter content         |
|**3**|`geojson-schema`              |Builds on geojson-validate, adds TypeScript magic                      |
|**4**|`stac-client`, `pmtiles-utils`|Cloud-native utilities, pairs with R cloud-native packages             |
|**5**|`overture-query`              |Most complex (DuckDB-WASM), but high impact                            |

-----

## Cross-Ecosystem Alignment

|Domain             |Python             |R                 |npm (TypeScript)                    |
|-------------------|-------------------|------------------|------------------------------------|
|Validation         |`spatial-validate` |`spatialvalidate` |`@chrislyonsKY/geojson-validate`|
|Schema contracts   |`schema-enforcer`  |`schemaenforcer`  |`@chrislyonsKY/geojson-schema`  |
|Color accessibility|`map-accessibility`|`mapaccessibility`|`@chrislyonsKY/map-a11y`        |
|Carto palettes     |—                  |`kartocolors`     |`@chrislyonsKY/karto-palettes`  |
|LLM utilities      |`geoai-toolkit`    |`geoaitools`      |`@chrislyonsKY/geo-llm`         |
|STAC access        |(rstac, pystac)    |`stacr`           |`@chrislyonsKY/stac-client`     |
|PMTiles            |—                  |`pmtilesr`        |`@chrislyonsKY/pmtiles-utils`   |
|Overture Maps      |—                  |`overturer`       |`@chrislyonsKY/overture-query`  |

-----

## Technical Standards

### Package Structure

```
@chrislyonsKY/package-name/
├── src/
│   ├── index.ts           # Public exports
│   ├── types.ts           # TypeScript types
│   └── [modules].ts       # Implementation
├── dist/
│   ├── index.js           # ESM bundle
│   ├── index.cjs          # CommonJS bundle
│   ├── index.d.ts         # Type declarations
│   └── index.min.js       # Minified for CDN
├── package.json
├── tsconfig.json
├── README.md
├── CHANGELOG.md
└── LICENSE                # MIT
```

### package.json Conventions

```json
{
  "name": "@chrislyonsKY/package-name",
  "version": "0.1.0",
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
  "sideEffects": false,
  "engines": { "node": ">=18" },
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "test": "vitest",
    "lint": "eslint src"
  }
}
```

### Build Tooling

- **Bundler:** tsup (zero-config, fast)
- **Testing:** Vitest
- **Linting:** ESLint with `@typescript-eslint`
- **Formatting:** Prettier
- **CI:** GitHub Actions (test on Node 18, 20, 22)

-----

## Visual Identity

npm packages follow the same tier system as Python/R:

|Tier|Name   |Color         |Packages                                                          |
|----|-------|--------------|------------------------------------------------------------------|
|3   |Quality|Rose `#F43F5E`|`geojson-validate`, `geojson-schema`, `map-a11y`, `karto-palettes`|
|5   |Tools  |Sky `#0EA5E9` |`geo-llm`, `pmtiles-utils`, `stac-client`, `overture-query`       |

**npm package badges:** Square format (512×512), dark background with tier accent, package name in JetBrains Mono.

-----

## What I’m NOT Building

These already have good npm solutions:

|Don’t Build          |Use Instead                     |
|---------------------|--------------------------------|
|GeoJSON manipulation |`@turf/turf`                    |
|Map rendering        |MapLibre GL JS, Leaflet, deck.gl|
|Coordinate transforms|`proj4`                         |
|WKT/WKB parsing      |`wkx`, `wellknown`              |
|Spatial indexing     |`rbush`, `flatbush`             |
|GeoTIFF reading      |`geotiff`                       |
|PMTiles rendering    |`pmtiles` (official)            |
|General validation   |`zod`, `ajv`                    |
