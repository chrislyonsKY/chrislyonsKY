# Reusable Workflow Templates

Standardized GitHub Actions workflows for publishing to each registry.

## Usage

```yaml
# In a package repo
jobs:
  publish:
    uses: chrislyonsKY/.github/.github/workflows/pypi-publish.yml@main
    secrets: inherit
```

## Available Templates

| Template | Registry | Auth Method |
|----------|----------|-------------|
| `r-cmd-check.yml` | CRAN (CI only) | N/A |
| `pypi-publish.yml` | PyPI | Trusted publisher (OIDC) |
| `npm-publish.yml` | npm | Trusted publisher (OIDC) |
| `crates-publish.yml` | crates.io | Trusted publisher (OIDC) |
| `nuget-publish.yml` | NuGet | Scoped API key |
| `qgis-publish.yml` | plugins.qgis.org | OSGeo password |
