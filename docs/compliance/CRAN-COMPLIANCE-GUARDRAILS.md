# CRAN Submission Compliance — Chris Lyons R Packages

> Guardrails that apply to ALL R packages destined for CRAN.  
> Based on CRAN Repository Policy, submission checklist, and lessons from actual rejections.  
> Reference: https://cran.r-project.org/web/packages/policies.html  
> Reference: https://cran.r-project.org/web/packages/submission_checklist.html

---

## DESCRIPTION File

- **Title:** Must be in Title Case. Must NOT start with "A package..." or the package name. Must NOT contain "in R" (redundant on CRAN). Keep under 65 characters.
- **Description:** Must be informative for new users. Multiple sentences. Must NOT start with the package name. Reference other packages in single quotes ('sf', 'terra'). Function names with parentheses: `cg_read()`. Describe what it does, what's new, how it differs from CRAN alternatives.
- **Authors@R:** Use `person()` with proper roles (`aut`, `cre`, `ctb`). The `cre` (maintainer) must have a valid, monitored email.
- **License:** MIT requires a `LICENSE` file: use `License: MIT + file LICENSE` in DESCRIPTION.
- **URL / BugReports:** Include both. CRAN landing page shows these.
- **Version:** Use `x.y.z` format. Development versions use `x.y.z.9000`.
- **SystemRequirements:** If the package depends on external software (GDAL, PROJ, etc.), declare it here.

### DESCRIPTION Template
```
Package: overturer
Title: Access Overture Maps Data from R
Version: 0.1.0
Authors@R:
    person("Chris", "Lyons", , "chris@github.com/chrislyonsKY",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0000-0000-0000"))
Description: Provides tidy access to Overture Maps Foundation datasets
    (buildings, places, transportation, divisions, addresses) distributed
    as 'GeoParquet' files on Amazon S3. Queries are executed via 'DuckDB'
    with spatial predicate pushdown, returning 'sf' tibbles without
    downloading the full dataset. See <https://overturemaps.org> for
    details about the Overture Maps data schema.
License: MIT + file LICENSE
URL: https://github.com/chrislyonsKY/overturer,
    https://chrislyonsKY.github.io/overturer/
BugReports: https://github.com/chrislyonsKY/overturer/issues
Depends: R (>= 4.1.0)
Imports: duckdb, DBI, sf, cli
Suggests: testthat (>= 3.0.0), arrow, withr, knitr, rmarkdown
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.2
Config/testthat/edition: 3
VignetteBuilder: knitr
```

---

## Examples (roxygen2 @examples)

CRAN requires:
- Packages must pass `R CMD check --as-cran` with current R-devel.
- Examples should be quick — they're tested daily on multiple systems.
- `\dontrun{}` should only be used if the example truly cannot be executed (missing software, API keys).
- Replace `\dontrun{}` with `\donttest{}` for slow examples, or unwrap if they run in under 5 seconds.
- All exported functions must have `@return` documentation. Even side-effect functions need `@return` (invisibly returns NULL, etc.).
- Never comment out example code — CRAN rejects this.

### What This Means for Our Packages

**Cloud/network-dependent functions** (all of cloudgeo, overturer, stacr, envdatar, pmtilesr, duckgeo):
- Use `@examplesIf` with a connectivity check function:
  ```r
  #' @examplesIf has_internet() && has_duckdb_spatial()
  #' buildings <- ov_buildings(bbox = c(-84.5, 38.0, -84.3, 38.2))
  ```
- Provide a small local dataset in `inst/extdata/` for examples that CAN run:
  ```r
  #' @examples
  #' # Using bundled sample data (no network needed)
  #' sample <- system.file("extdata", "sample_buildings.gpkg", package = "overturer")
  #' buildings <- sf::st_read(sample)
  ```
- Create predicates: `has_internet()`, `has_duckdb_spatial()`, `has_stac_access()` etc.

**Computation-heavy functions** (geochange, aboveR):
- Wrap in `\donttest{}` or use small bundled test rasters.
- Include a `\dontshow{}` block to subset data for faster runs.

---

## File System Rules

- Never write to the user's home directory, nor anywhere else on the file system apart from R's session temporary directory.
- Always use `tempdir()` for any file output in examples and tests.
- Functions that write files should take an explicit `path` argument, never assume a working directory.
- If a function creates temp files during execution, clean them up.

### What This Means for Our Packages

- `cg_write()`, `pm_write()`, `dg_export()`: all default to writing where the user specifies. Examples must use `tempdir()`.
- `stac_download()`: must take an `output_dir` argument, examples use `tempdir()`.
- Cache files (envdatar): use `tools::R_user_dir("envdatar", "cache")` — CRAN-approved user cache location.

---

## Testing

- Tests must not depend on network access by default. Mock HTTP calls or skip with `skip_if_not(has_internet())`.
- Tests must run on all major platforms (Linux, macOS, Windows).
- Tests must not take more than a few minutes total.
- Use `testthat` edition 3 (`Config/testthat/edition: 3`).
- Use `withr::local_tempdir()` for temp file cleanup in tests.
- Use `skip_on_cran()` for integration tests that need network or large data.

---

## Suggested vs Imported Dependencies

CRAN distinguishes between `Imports` (always available) and `Suggests` (may not be installed). This matters significantly for our packages because many depend on optional heavy packages.

### Rules

- If a function ALWAYS needs a package, it goes in `Imports`.
- If a function CAN work without a package, or is an optional feature, the dependency goes in `Suggests`.
- When using a Suggested package, always check availability:
  ```r
  if (!requireNamespace("gdalcubes", quietly = TRUE)) {
    cli::cli_abort(c(
      "Package {.pkg gdalcubes} is required for STAC data cube creation.",
      "i" = "Install it with: {.code install.packages('gdalcubes')}"
    ))
  }
  ```
- Guard examples with: `@examplesIf rlang::is_installed("gdalcubes")`

### Dependency Strategy per Package

| Package | Imports (always) | Suggests (optional) |
|---------|-----------------|-------------------|
| cloudgeo | sf, terra | arrow, geoarrow, rstac, gdalcubes, aws.s3, cli |
| overturer | duckdb, DBI, sf, cli | arrow, testthat |
| stacr | rstac, tibble, cli | gdalcubes, leaflet, sf, testthat |
| duckgeo | duckdb, DBI, sf | dbplyr, arrow, cli, testthat |
| pmtilesr | httr2, sf, protolite | leaflet, duckdb, cli, testthat |
| envdatar | httr2, jsonlite, tibble, sf, cli | memoise, testthat |
| geochange | terra, sf | gdalcubes, rstac, cli, testthat |
| georender | ggplot2, sf, ggspatial | patchwork, rosm, kartocolors, testthat |

**Note:** For cloudgeo, most backends are Suggests — the package only Imports sf and terra. This keeps the install lightweight and passes `R CMD check` without needing GDAL compiled with every driver.

---

## GitHub Actions CI for CRAN

Every R package needs this workflow (`.github/workflows/R-CMD-check.yaml`):

```yaml
name: R-CMD-check

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  R-CMD-check:
    runs-on: ${{ matrix.config.os }}
    name: ${{ matrix.config.os }} (${{ matrix.config.r }})
    strategy:
      fail-fast: false
      matrix:
        config:
          - {os: macos-latest,   r: 'release'}
          - {os: windows-latest, r: 'release'}
          - {os: ubuntu-latest,  r: 'devel', http-user-agent: 'release'}
          - {os: ubuntu-latest,  r: 'release'}
          - {os: ubuntu-latest,  r: 'oldrel-1'}
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
      R_KEEP_PKG_SOURCE: yes
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-r@v2
        with:
          r-version: ${{ matrix.config.r }}
          http-user-agent: ${{ matrix.config.http-user-agent }}
          use-public-rspm: true
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::rcmdcheck
          needs: check
      - uses: r-lib/actions/check-r-package@v2
        with:
          upload-snapshots: true
```

This tests on macOS, Windows, Linux with R-release, R-devel, and R-oldrel — meeting CRAN's requirement of working across "all major R platforms."

---

## Pre-Submission Checklist

Before every CRAN submission:

```
[ ] R CMD check --as-cran passes with 0 errors, 0 warnings, 0 notes
    (1 NOTE for " submission" is acceptable on first submission)
[ ] Tested on R-devel via win-builder or GitHub Actions
[ ] Tested on macOS via mac.r-project.org/macbuilder
[ ] DESCRIPTION: Title in Title Case, no "A package...", no "in R"
[ ] DESCRIPTION: Description is informative, multi-sentence
[ ] DESCRIPTION: All package refs in single quotes ('sf', 'DuckDB')
[ ] All exported functions have @return tag
[ ] All exported functions have @examples (or @examplesIf for network-dependent)
[ ] Examples run in < 5 sec each (or wrapped in \donttest{})
[ ] No \dontrun{} unless truly impossible to execute
[ ] No commented-out example code
[ ] Tests pass on Linux, macOS, Windows
[ ] Network-dependent tests use skip_on_cran() or skip_if_not(has_internet())
[ ] No writes outside tempdir() in examples or tests
[ ] Vignettes build without error (or are pre-built)
[ ] NEWS.md is up to date
[ ] cran-comments.md describes test environments and any notes
[ ] Version number is release-ready (no .9000)
[ ] devtools::release() used for submission (runs extra checks)
```

---

## Common Rejection Reasons (and How We Avoid Them)

| Rejection | Our Prevention |
|-----------|---------------|
| "Please do not start Description with the package name" | DESCRIPTION template enforces this |
| "Examples take too long" | `@examplesIf` + bundled sample data + `\donttest{}` |
| "Please write to tempdir()" | All write-functions default to explicit path, examples use `tempdir()` |
| "Missing @return" | roxygen2 template includes @return for every function |
| "Please replace \dontrun with \donttest" | Reserve `\dontrun` for truly impossible examples only |
| "Package requires network access for check" | All network tests behind `skip_on_cran()` |
| "Check results show WARNING on [platform]" | GitHub Actions matrix covers all CRAN platforms |
