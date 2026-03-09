# Shared Test Fixtures

Language-agnostic test data consumed by packages across all 6 ecosystems.

## Directories

| Directory | Contents | Consumers |
|-----------|----------|-----------|
| `geojson/` | Valid, invalid, edge-case GeoJSON files | geo-validate (all ecosystems) |
| `schemas/` | YAML schema contract files | schema-enforcer (all ecosystems) |
| `palettes/` | kartocolors palette definitions (JSON) | kartocolors (R, Rust, npm, NuGet) |
| `coordinates/` | Coordinate parsing test vectors (JSON) | geoai-parse (all ecosystems) |
| `rasters/` | Small GeoTIFFs for change detection | geochange (R, Rust) |

## How to Consume

### Git Submodule (recommended)
```bash
git submodule add https://github.com/chrislyonsKY/.github shared/test-fixtures
```

### CI Download
```yaml
- uses: actions/checkout@v4
  with:
    repository: chrislyonsKY/.github
    path: test-fixtures
    sparse-checkout: shared/test-fixtures
```
