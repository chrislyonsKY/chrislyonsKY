# Chris Lyons — Cross-Registry Publishing Compliance & Tooling

> Registry-specific compliance rules, CI/CD workflows, and release checklists  
> Covers: CRAN, PyPI, QGIS, crates.io, NuGet, npm  
> Companion to: chrislyonsKY-ECOSYSTEM-PLAN.md

---

## Quick Reference: Key Docs Per Registry

| Registry | Primary Guide | Policy / Rules | Submission Tool |
|----------|--------------|---------------|-----------------|
| **CRAN** | [r-pkgs.org/release.html](https://r-pkgs.org/release.html) | [CRAN Policies](https://cran.r-project.org/web/packages/policies.html) | `devtools::submit_cran()` or [web form](https://cran.r-project.org/submit.html) |
| **PyPI** | [packaging.python.org](https://packaging.python.org/en/latest/tutorials/packaging-projects/) | [PyPI Help](https://pypi.org/help/) | `pypa/gh-action-pypi-publish` (trusted publisher) |
| **QGIS** | [PyQGIS Cookbook: Plugins](https://docs.qgis.org/3.40/en/docs/pyqgis_developer_cookbook/plugins/plugins.html) | [Approval Docs](https://plugins.qgis.org/docs/approval) | `qgis-plugin-ci` or [web upload](https://plugins.qgis.org/plugins/add/) |
| **crates.io** | [Cargo: Publishing](https://doc.rust-lang.org/cargo/reference/publishing.html) | [crates.io Policies](https://crates.io/policies) | `cargo publish` + trusted publisher |
| **NuGet** | [Publish a Package](https://learn.microsoft.com/en-us/nuget/nuget-org/publish-a-package) | [Best Practices](https://learn.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices) | `dotnet nuget push` |
| **npm** | [npm Docs: Contributing](https://docs.npmjs.com/packages-and-modules/contributing-packages-to-the-registry) | [npm Policies](https://docs.npmjs.com/policies) | `npm publish --provenance` + trusted publisher |

---

## Critical Differences That Affect Our Workflow

### Human Review

Only two registries require human review before packages go live:

| Registry | Review | Turnaround | Implication |
|----------|--------|-----------|-------------|
| **CRAN** | Volunteer team, every submission | Days (sometimes weeks) | Plan 2-3 week buffer; never rush. Increment version on resubmit. |
| **QGIS** | Staff review, daily M-F | ~1 business day | Less risky, but still plan for rejection on first attempt. |
| PyPI, crates.io, NuGet, npm | None (automated validation only) | Instant to 15 min | Ship fast, but version numbers are permanent. |

### Version Immutability

Once published, versions cannot be reused on **any** registry:

| Registry | Can Delete? | Can Overwrite? | Recovery |
|----------|-------------|---------------|----------|
| CRAN | Only via CRAN team (rare) | No | Submit new version |
| PyPI | Yes (but name+version burned forever) | No | Bump version |
| QGIS | Via staff request | No | Bump version |
| crates.io | Never (yank hides, doesn't delete) | No | Bump version |
| NuGet | Never (unlist hides, doesn't delete) | No | Bump version |
| npm | Within 72 hours only | No | Bump version |

**Rule: Always dry-run/preview before publishing. Always.**

### Size Limits

| Registry | Limit | Our Risk |
|----------|-------|----------|
| CRAN | 5 MB soft / 10 MB hard | `aboveR` sample LiDAR data, `envdatar` cached responses |
| PyPI | 100 MiB per file, 10 GiB per project | No risk |
| QGIS | 25 MB | `nil-overture` bundled DuckDB could be tricky |
| crates.io | 10 MB per .crate | Fine for our pure-logic crates |
| NuGet | ~250 MB | No risk |
| npm | No hard limit (practical ~50 MB) | No risk |

---

## 1. CRAN — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `devtools` | Build, check, submit | `install.packages("devtools")` |
| `usethis` | Release issues, version bumping, GitHub Actions setup | `install.packages("usethis")` |
| `roxygen2` | Documentation generation | `install.packages("roxygen2")` |
| `testthat` v3 | Testing framework | `install.packages("testthat")` |
| `urlchecker` | Validate all URLs in package | `install.packages("urlchecker")` |
| `rhub` | Cross-platform check via R-hub | `install.packages("rhub")` |
| `revdepcheck` | Reverse dependency checking (for updates) | `install.packages("revdepcheck")` |
| `available` | Check name availability | `install.packages("available")` |
| `rcmdcheck` | Programmatic R CMD check | `install.packages("rcmdcheck")` |
| `goodpractice` | Additional code quality checks | `install.packages("goodpractice")` |
| `spelling` | Spell-check documentation | `install.packages("spelling")` |

### Pre-Submission Commands (Run for Every Package)

```r
# 1. Documentation
devtools::document()
urlchecker::url_check()
spelling::spell_check_package()

# 2. Local check
devtools::check(remote = TRUE, manual = TRUE)
# Must produce: 0 errors, 0 warnings, 0 notes (1 NOTE for new submission OK)

# 3. Cross-platform checks
devtools::check_win_devel()        # Windows R-devel
devtools::check_mac_release()      # macOS
rhub::rhub_check()                 # Linux flavors

# 4. Build README
devtools::build_readme()

# 5. Reverse dependency check (updates only)
revdepcheck::revdep_check(num_workers = 4)

# 6. Final submission
usethis::use_version("minor")      # or "patch" / "major"
devtools::submit_cran()            # → confirm email → track at CRAN/incoming
```

### GitHub Actions: R-CMD-check.yaml

Already defined in CRAN-COMPLIANCE-GUARDRAILS.md. Key addition — add `pkgdown` site deployment:

```yaml
# .github/workflows/pkgdown.yaml
name: pkgdown
on:
  push:
    branches: [main]
  release:
    types: [published]
permissions:
  contents: write
jobs:
  pkgdown:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: r-lib/actions/setup-r@v2
      - uses: r-lib/actions/setup-r-dependencies@v2
        with:
          extra-packages: any::pkgdown
          needs: website
      - name: Build site
        run: pkgdown::build_site_github_pages(new_process = FALSE)
        shell: Rscript {0}
      - uses: actions/upload-pages-artifact@v3
        with:
          path: docs
      - uses: actions/deploy-pages@v4
```

### CRAN Release Checklist (All 19 R Packages)

```
[ ] `available::available("pkgname")` confirms name is free
[ ] DESCRIPTION: Title in Title Case, no "A package..." or "in R"
[ ] DESCRIPTION: Description is multi-sentence, informative
[ ] DESCRIPTION: Package refs in single quotes ('sf', 'terra')
[ ] DESCRIPTION: Version has no .9000 suffix
[ ] All exported functions have @return tag
[ ] All exported functions have @examples or @examplesIf
[ ] No \dontrun{} (use \donttest{} or @examplesIf)
[ ] No commented-out example code
[ ] Examples run in < 5 sec each
[ ] All file writes use tempdir()
[ ] Network tests use skip_on_cran()
[ ] Suggested packages checked with requireNamespace()
[ ] `devtools::document()` — clean
[ ] `urlchecker::url_check()` — no broken URLs
[ ] `spelling::spell_check_package()` — clean or WORDLIST updated
[ ] `devtools::check(remote = TRUE, manual = TRUE)` — 0/0/0
[ ] `devtools::check_win_devel()` — clean
[ ] `devtools::check_mac_release()` — clean
[ ] `rhub::rhub_check()` — clean
[ ] NEWS.md updated
[ ] cran-comments.md updated
[ ] `devtools::build_readme()` — renders cleanly
[ ] Vignettes build without error
[ ] GitHub Actions R-CMD-check passes on all platforms
[ ] `devtools::submit_cran()` — submitted
[ ] Confirmation email clicked
[ ] Post-acceptance: `usethis::use_github_release()` + bump to .9000
```

---

## 2. PyPI — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `build` | PEP 517 build frontend | `pip install build` |
| `twine` | Upload + README validation | `pip install twine` |
| `ruff` | Linting + formatting (replaces flake8+black+isort) | `pip install ruff` |
| `mypy` | Static type checking | `pip install mypy` |
| `pytest` | Testing | `pip install pytest` |
| `pytest-cov` | Coverage reporting | `pip install pytest-cov` |
| `hatch` / `hatchling` | Build backend (our choice) | `pip install hatch` |
| `check-wheel-contents` | Validate wheel packaging | `pip install check-wheel-contents` |

### Pre-Publish Commands

```bash
# 1. Lint + type check
ruff check src/
ruff format --check src/
mypy src/

# 2. Test
pytest --cov=src --cov-report=term-missing

# 3. Build
python -m build

# 4. Validate
twine check dist/*                      # README rendering check
check-wheel-contents dist/*.whl          # packaging sanity

# 5. Test upload (first time)
twine upload --repository testpypi dist/*

# 6. Production upload (manual fallback — prefer trusted publisher)
twine upload dist/*
```

### GitHub Actions: Trusted Publisher Workflow

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI
on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install build
      - run: python -m build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  publish:
    needs: build
    runs-on: ubuntu-latest
    environment: pypi                    # GitHub environment with protection rules
    permissions:
      id-token: write                    # Required for OIDC
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist/ }
      - uses: pypa/gh-action-pypi-publish@release/v1
```

**Setup:** Register each package as a trusted publisher at pypi.org → Manage → Publishing. Specify `chrislyonsKY` org, repo name, workflow file `publish.yml`, and environment `pypi`.

### GitHub Actions: CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: ${{ matrix.python-version }} }
      - run: pip install -e ".[dev]"
      - run: ruff check src/
      - run: ruff format --check src/
      - run: mypy src/
      - run: pytest --cov=src --cov-report=xml
      - uses: codecov/codecov-action@v4
        if: matrix.os == 'ubuntu-latest' && matrix.python-version == '3.12'
```

### PyPI Release Checklist (All 20 Python Packages)

```
[ ] Package name available (check pypi.org search with normalized name)
[ ] pyproject.toml: name, version, description, readme, license, requires-python, authors
[ ] pyproject.toml: classifiers include Python version + license + topic
[ ] pyproject.toml: [project.urls] has Repository, Documentation, Issues
[ ] README.md renders correctly (twine check dist/*)
[ ] No Private :: classifier (would block upload)
[ ] py.typed marker present for typed packages
[ ] Version follows PEP 440 (no +local suffixes)
[ ] ruff check + ruff format — clean
[ ] mypy — clean (or configured ignores documented)
[ ] pytest — all passing, coverage > 80%
[ ] python -m build — succeeds
[ ] check-wheel-contents — no issues
[ ] Trusted publisher configured on pypi.org
[ ] GitHub environment "pypi" created with required reviewers
[ ] Create GitHub Release → workflow publishes automatically
[ ] Verify package page renders at pypi.org/project/{name}
```

---

## 3. QGIS — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `qgis-plugin-ci` | Build, package, upload to plugins.qgis.org | `pip install qgis-plugin-ci` |
| `pb_tool` | Alternative plugin build tool | `pip install pb_tool` |
| `pytest` + `qgis.testing` | Testing within QGIS environment | Docker: `qgis/qgis:latest` |
| `pylint` + `flake8` | Linting (QGIS plugins use older conventions) | `pip install pylint flake8` |

### qgis-plugin-ci Configuration

```yaml
# .qgis-plugin-ci
plugin_path: nil_validate
github_organization_slug: chrislyonsKY
project_slug: nil-validate
```

### GitHub Actions: Release + Test

```yaml
# .github/workflows/release.yml
name: Release QGIS Plugin
on:
  release:
    types: [published]

jobs:
  package:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install qgis-plugin-ci
      - name: Package and release
        run: >-
          qgis-plugin-ci release ${GITHUB_REF/refs\/tags\//}
          --github-token ${{ secrets.GITHUB_TOKEN }}
          --osgeo-username ${{ secrets.OSGEO_USER }}
          --osgeo-password ${{ secrets.OSGEO_PASSWORD }}
          --create-plugin-repo
```

```yaml
# .github/workflows/test.yml
name: Test QGIS Plugin
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: qgis/qgis:latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: |
          export QT_QPA_PLATFORM=offscreen
          cd /github/workspace
          python -m pytest tests/ -v
```

### QGIS Release Checklist (All 8 Plugins)

```
[ ] OSGeo ID registered at osgeo.org
[ ] metadata.txt: all 8 required fields present
[ ] metadata.txt: version incremented (not reused)
[ ] metadata.txt: qgisMinimumVersion = 3.34 (current LTS)
[ ] metadata.txt: tracker and repository URLs valid and public
[ ] metadata.txt: tags relevant (comma-separated)
[ ] LICENSE file present (no extension, not LICENSE.txt)
[ ] __init__.py with classFactory() function
[ ] No __pycache__, .git, __MACOSX in ZIP
[ ] No binary files in ZIP
[ ] ZIP has single top-level directory matching plugin name
[ ] ZIP under 25 MB
[ ] Plugin installs without crash in QGIS 3.34
[ ] All Processing algorithms register correctly
[ ] Icon present (icons/icon.svg or .png)
[ ] OSGEO_USER and OSGEO_PASSWORD in GitHub Secrets
[ ] qgis-plugin-ci release succeeds
[ ] Approval received (check plugins.qgis.org dashboard)
```

---

## 4. crates.io — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `cargo` | Build, test, publish (built-in) | Rust toolchain |
| `cargo-release` | Automated version bumping, tagging, publishing | `cargo install cargo-release` |
| `cargo-semver-checks` | SemVer violation detection | `cargo install cargo-semver-checks` |
| `cargo-deny` | Dependency license + advisory audit | `cargo install cargo-deny` |
| `cargo-tarpaulin` | Code coverage | `cargo install cargo-tarpaulin` |
| `cargo-audit` | Security advisory checking | `cargo install cargo-audit` |

### Pre-Publish Commands

```bash
# 1. Format + lint
cargo fmt --check
cargo clippy --all-features -- -D warnings

# 2. Test
cargo test --all-features
cargo test --no-default-features       # WASM-compatible subset

# 3. SemVer check (updates only)
cargo semver-checks check-release

# 4. Dependency audit
cargo deny check licenses
cargo audit

# 5. Dry run
cargo publish --dry-run
cargo package --list                   # verify included files

# 6. Publish
cargo publish
```

### GitHub Actions: CI + Trusted Publisher Release

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with: { components: clippy, rustfmt }
      - run: cargo fmt --check
      - run: cargo clippy --all-features -- -D warnings
      - run: cargo test --all-features
      - run: cargo doc --all-features --no-deps

  # WASM compatibility check for applicable crates
  wasm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with: { targets: wasm32-unknown-unknown }
      - run: cargo build --target wasm32-unknown-unknown --no-default-features --features wasm
```

```yaml
# .github/workflows/release.yml
name: Publish to crates.io
on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: rust-lang/crates-io-auth-action@v1
        id: auth
      - run: cargo publish
        env:
          CARGO_REGISTRY_TOKEN: ${{ steps.auth.outputs.token }}
```

**Setup:** Configure trusted publisher at crates.io → Settings → Trusted Publishers. Specify GitHub repo and workflow file.

### crates.io Release Checklist (All 10 Crates)

```
[ ] Cargo.toml: name, version, edition, description, license all present
[ ] Cargo.toml: repository, readme, keywords (max 5), categories (max 5)
[ ] Cargo.toml: categories match crates.io slugs exactly
[ ] License: MIT OR Apache-2.0 (dual license, Rust convention)
[ ] cargo fmt --check — clean
[ ] cargo clippy --all-features -- -D warnings — clean
[ ] cargo test --all-features — all passing
[ ] cargo semver-checks — no breaking changes (for non-major bumps)
[ ] cargo deny check licenses — no GPL contamination in deps
[ ] cargo audit — no known vulnerabilities
[ ] cargo publish --dry-run — succeeds
[ ] cargo package --list — no unintended files (.env, secrets)
[ ] .crate file under 10 MB
[ ] README.md includes crates.io badge, docs.rs badge
[ ] Trusted publisher configured on crates.io
[ ] Create git tag (v*) → workflow publishes automatically
[ ] Verify at crates.io/crates/{name} and docs.rs/{name}
```

---

## 5. NuGet — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `dotnet pack` | Create .nupkg from .csproj | .NET SDK |
| `dotnet nuget push` | Upload to NuGet.org | .NET SDK |
| `dotnet test` | Run tests | .NET SDK |
| `nuget-trends.com` | Check name popularity/conflicts | Web |
| `snupkg` | Symbol package for debugging | .NET SDK (built-in) |

### .csproj Packaging Properties (Standard for All NuGet Packages)

```xml
<PropertyGroup>
  <!-- Required -->
  <PackageId>ChrisLyons.GeoValidate</PackageId>
  <Version>1.0.0</Version>
  <Authors>Chris Lyons</Authors>
  <Description>Geometry and attribute validation for geospatial datasets</Description>
  
  <!-- Required for quality -->
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <PackageReadmeFile>README.md</PackageReadmeFile>
  <PackageIcon>icon.png</PackageIcon>
  <RepositoryUrl>https://github.com/chrislyonsKY/geo-validate-dotnet</RepositoryUrl>
  <RepositoryType>git</RepositoryType>
  <PackageTags>geospatial;gis;validation;geometry;quality</PackageTags>
  
  <!-- Best practices -->
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  <EmbedUntrackedSources>true</EmbedUntrackedSources>
  <PublishRepositoryUrl>true</PublishRepositoryUrl>
  
  <!-- Deterministic builds for Source Link -->
  <ContinuousIntegrationBuild Condition="'$(CI)' == 'true'">true</ContinuousIntegrationBuild>
</PropertyGroup>

<ItemGroup>
  <PackageReference Include="Microsoft.SourceLink.GitHub" Version="8.*" PrivateAssets="All" />
</ItemGroup>

<ItemGroup>
  <None Include="../../README.md" Pack="true" PackagePath="" />
  <None Include="../../assets/icon.png" Pack="true" PackagePath="" />
</ItemGroup>
```

### GitHub Actions: CI + Publish

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  build-test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '8.x' }
      - run: dotnet restore
      - run: dotnet build --configuration Release --no-restore
      - run: dotnet test --configuration Release --no-restore --verbosity normal
```

```yaml
# .github/workflows/publish.yml
name: Publish NuGet
on:
  push:
    tags: ['v*.*.*']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }           # Full history for Source Link
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '8.x' }
      - run: dotnet restore
      - run: dotnet build --configuration Release --no-restore
      - run: dotnet test --configuration Release --no-restore
      - name: Pack
        run: dotnet pack --configuration Release --no-restore -p:PackageVersion=${GITHUB_REF#refs/tags/v}
      - name: Push
        run: dotnet nuget push "**/*.nupkg" --source https://api.nuget.org/v3/index.json --api-key ${{ secrets.NUGET_API_KEY }} --skip-duplicate
```

**Setup:** Create API key at nuget.org → Account → API Keys. Scope to `ChrisLyons.*` glob pattern. Set 365-day expiration and rotate annually.

### NuGet Release Checklist (All 8 Packages)

```
[ ] PackageId follows ChrisLyons.{Name} convention
[ ] Version is SemVer 2.0.0 compliant
[ ] PackageLicenseExpression set (not deprecated LicenseUrl)
[ ] PackageIcon embedded (not deprecated IconUrl)
[ ] PackageReadmeFile points to README.md
[ ] RepositoryUrl set with RepositoryType=git
[ ] GenerateDocumentationFile=true (XML docs)
[ ] IncludeSymbols=true, SymbolPackageFormat=snupkg
[ ] Source Link configured (Microsoft.SourceLink.GitHub)
[ ] Nullable reference types enabled
[ ] dotnet build --configuration Release — no warnings
[ ] dotnet test — all passing
[ ] dotnet pack — produces .nupkg + .snupkg
[ ] Package icon is 128×128 PNG, under 1 MB
[ ] NUGET_API_KEY in GitHub Secrets (scoped to ChrisLyons.*)
[ ] Create git tag (v*.*.*) → workflow publishes
[ ] Verify at nuget.org/packages/ChrisLyons.{Name}
[ ] Consider prefix reservation: email account@nuget.org for ChrisLyons.*
```

---

## 6. npm — Release Tooling & Compliance

### Toolchain

| Tool | Purpose | Install |
|------|---------|---------|
| `tsup` | TypeScript → ESM + CJS + .d.ts bundler | `npm install -D tsup` |
| `vitest` | Testing (ESM-native, Jest-compatible API) | `npm install -D vitest` |
| `eslint` | Linting | `npm install -D eslint` |
| `typescript` | Type checking | `npm install -D typescript` |
| `size-limit` | Bundle size tracking | `npm install -D size-limit @size-limit/preset-small-lib` |
| `@arethetypeswrong/cli` | Validate TypeScript export maps | `npx @arethetypeswrong/cli` |
| `typedoc` | API documentation generation | `npm install -D typedoc` |
| `npm-pack-all` | Preview published contents | `npm pack --dry-run` |

### package.json Scripts (Standard for All npm Packages)

```json
{
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts --clean",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src/ --ext .ts",
    "typecheck": "tsc --noEmit",
    "size": "size-limit",
    "attw": "attw --pack .",
    "prepublishOnly": "npm run build && npm run test && npm run typecheck"
  }
}
```

### GitHub Actions: CI + Trusted Publisher

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: ${{ matrix.node-version }} }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run build
      - run: npm test
      - run: npx @arethetypeswrong/cli --pack .
      - run: npx size-limit
        if: matrix.node-version == 22
```

```yaml
# .github/workflows/publish.yml
name: Publish to npm
on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write                     # Required for provenance + trusted publishing
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm run build
      - run: npm test
      - run: npm publish --provenance --access public
```

**Setup:** Configure trusted publisher at npmjs.com → Package Settings → Trusted Publishing. Requires npm CLI 11.5.1+ and Node 22.14.0+. First publish of a scoped package needs `--access public`.

### npm Release Checklist (All 12 Packages)

```
[ ] @chrislyonsKY npm org created and 2FA enabled
[ ] package.json: name (@chrislyonsKY/{name}), version, description
[ ] package.json: "type": "module"
[ ] package.json: exports map with import/require/types conditions
[ ] package.json: "files" whitelist (prefer over .npmignore)
[ ] package.json: "sideEffects": false (for tree-shaking)
[ ] package.json: license, repository, homepage, engines
[ ] package.json: publishConfig.access = "public" (scoped packages)
[ ] TypeScript: types conditions listed FIRST in exports
[ ] npx @arethetypeswrong/cli --pack . — no issues
[ ] npm pack --dry-run — only intended files included
[ ] No .env, .npmrc, secrets in packed files
[ ] npm run lint — clean
[ ] npm run typecheck — clean
[ ] npm test — all passing
[ ] npx size-limit — within budget (< 10KB gzipped for browser pkgs)
[ ] ESM + CJS dual build verified
[ ] Trusted publisher configured on npmjs.com
[ ] Create GitHub Release → workflow publishes with provenance
[ ] Verify green provenance checkmark on npmjs.com/package/@chrislyonsKY/{name}
```

---

## Cross-Ecosystem CI/CD Pattern

Every package across all 6 registries follows the same trigger model:

```
Push to main/PR  →  CI (lint, test, build, check)
Create GitHub Release (tag: v*)  →  Publish to registry
```

| Event | CRAN | PyPI | QGIS | crates.io | NuGet | npm |
|-------|------|------|------|-----------|-------|-----|
| Push/PR | R-CMD-check | ruff + mypy + pytest | pytest + QGIS Docker | clippy + test + fmt | build + test | lint + typecheck + vitest |
| Release tag | Manual submission | Trusted publisher OIDC | qgis-plugin-ci | Trusted publisher OIDC | API key push | Trusted publisher OIDC |

**CRAN is the exception** — automated publishing is not recommended. The release workflow for R packages is: CI passes → manual `devtools::submit_cran()` → confirm email → wait for approval.

---

## Shared Infrastructure Repos

### `chrislyonsKY/test-fixtures`

Shared test data consumed by packages across all ecosystems via git submodule or CI download:

```
test-fixtures/
├── geojson/
│   ├── valid_polygons.geojson
│   ├── invalid_self_intersecting.geojson
│   ├── null_geometry.geojson
│   └── mixed_types.geojson
├── schemas/
│   ├── mining_permits.yml
│   ├── building_footprints.yml
│   └── stream_segments.yml
├── palettes/
│   └── kartocolors.json          # All palette definitions
├── coordinates/
│   └── parse_test_vectors.json   # DMS, UTM, MGRS, DD test cases
├── rasters/
│   ├── small_dem_before.tif      # 100x100 px change detection pair
│   └── small_dem_after.tif
└── README.md
```

### `chrislyonsKY/release-tools`

Shared GitHub Actions workflows consumed via `uses: chrislyonsKY/release-tools/.github/workflows/...@v1`:

```
release-tools/
├── .github/workflows/
│   ├── pypi-publish.yml          # Reusable trusted publisher workflow
│   ├── npm-publish.yml           # Reusable trusted publisher workflow
│   ├── crates-publish.yml        # Reusable trusted publisher workflow
│   ├── nuget-publish.yml         # Reusable API key workflow
│   └── qgis-publish.yml          # Reusable qgis-plugin-ci workflow
├── templates/
│   ├── pyproject.toml.j2         # Standard pyproject.toml template
│   ├── Cargo.toml.j2
│   ├── package.json.j2
│   └── metadata.txt.j2
└── README.md
```

---

## Version Synchronization Strategy

Packages that implement the same concept across ecosystems do NOT need synchronized version numbers. Each ecosystem has its own release cadence. However, the **shared schema contract spec** (`schemas/*.yml` in test-fixtures) is versioned, and all schema-enforcer packages across ecosystems must support the current spec version.

| Shared Artifact | Versioned? | Sync Required? |
|----------------|-----------|---------------|
| Schema contract YAML spec | Yes (1.0, 1.1, etc.) | All schema-enforcer packages must support latest |
| Palette definitions (JSON) | Yes | All kartocolors packages use same hex values |
| Coordinate parse test vectors | Yes | All geoai-parse packages must pass all vectors |
| WCAG algorithms | Standardized (WCAG 2.1) | Implementations must produce identical results |

---

## Name Registration Priority

Some registries (PyPI, npm, crates.io) allow instant name registration without publishing code. **Register names early** to prevent squatting:

| Registry | How to Reserve | Priority Names |
|----------|---------------|----------------|
| **PyPI** | Upload empty 0.0.1 to TestPyPI first, then reserve on PyPI | All 20 package names |
| **crates.io** | Cannot reserve without publishing | Publish 0.1.0 stubs for `geo-validate`, `geo-schema`, `kartocolors`, `geoai-parse` first |
| **npm** | `npm init --scope=@chrislyonsKY` → `npm publish --access public` with stub | Register `@chrislyonsKY` org, publish stubs for all 12 |
| **NuGet** | Publish 0.1.0 stub or request prefix reservation | Email account@nuget.org to reserve `ChrisLyons.*` prefix |
| **CRAN** | Cannot reserve — names claimed on acceptance only | No action needed |
| **QGIS** | Cannot reserve — names claimed on upload | No action needed |
