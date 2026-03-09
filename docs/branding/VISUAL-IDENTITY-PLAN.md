# Chris Lyons — Visual Identity & Package Branding Plan

> 20 Python packages (banners + logos) + 10 R packages (hex stickers) + 1 org identity  
> Cohesive design system across both ecosystems

---

## Format Standards

### Python Packages — Banners & Logos

Python packages use three visual assets:

| Asset | Size | Where Used |
|-------|------|-----------|
| **README banner** | 1280 × 320px | Top of README.md (`<img>` tag, full width) |
| **PyPI logo** | 512 × 512px (square) | PyPI project page, favicon |
| **GitHub social preview** | 1280 × 640px | Link previews on social media, Slack, etc. |

**Format:** SVG source → export as PNG. README banners also work as SVG directly.

**README usage:**
```markdown
<p align="center">
  <img src="https://raw.githubusercontent.com/chrislyonsKY/{pkg}/main/assets/banner.png" alt="{pkg} banner" width="100%">
</p>
```

### R Packages — Hex Stickers

R packages use one hex sticker asset:

| Asset | Size | Where Used |
|-------|------|-----------|
| **Hex sticker** | 181 × 209mm (hexb.in spec) | README, pkgdown site, conference stickers, CRAN |

**Format:** SVG source → export as PNG at 300 DPI for print, 72 DPI for web.

**Standard R location:** `man/figures/logo.png` (pkgdown convention)

**README usage:**
```markdown
# packagename <img src="man/figures/logo.png" align="right" height="139" alt="" />
```

---

## Brand Foundation

### Color System

5 tier colors shared across Python and R. Dark backgrounds for all assets.

| Tier | Name | Accent | Background | Usage |
|------|------|--------|------------|-------|
| 1 | **Discovery** | `#14B8A6` (Teal) | `#0F172A` | Scanning, inventory, documentation |
| 2 | **ETL** | `#F59E0B` (Amber) | `#1A1708` | Pipelines, transformation, movement |
| 3 | **Quality** | `#F43F5E` (Rose) | `#1A0F12` | Validation, compliance, enforcement |
| 4 | **Processing** | `#8B5CF6` (Violet) | `#13101E` | Analysis, computation, spatial ops |
| 5 | **Tools** | `#0EA5E9` (Sky) | `#0C1929` | AI, monitoring, developer experience |

### Typography

| Use | Font | Weight | Notes |
|-----|------|--------|-------|
| Package name (Python banners) | **JetBrains Mono** | Bold | Monospace signals developer tool |
| Tagline (Python banners) | **Source Sans 3** | Light | Clean, readable at small sizes |
| Package name (R hex) | **JetBrains Mono** | Bold | Same as Python for consistency |
| "chrislyonsKY" org text | **Source Sans 3** | Regular | Subtle, bottom of banners/hexes |

### Icon Style

Minimal **line-art** — 2-3 strokes max, white lines on dark background with one element in the tier accent color. Think architectural blueprint / technical drawing aesthetic. No gradients, no photorealism, no clipart.

---

## Org Identity

### Chris Lyons Master Logo

**Concept:** A small island silhouette (palm tree on a tiny atoll) centered at crosshairs (0°N, 0°E), with faint latitude/longitude grid lines behind it. "CHRIS LYONS" in JetBrains Mono below.

**Usage:** GitHub org profile picture (square crop), README banner for the org landing page, conference table banner.

**Color:** White icon elements on `#0B1426` (deepest navy). Grid lines at 8% opacity. Crosshairs in `#14B8A6` (teal).

---

## Python Package Banners (20)

Each banner: 1280 × 320px, dark background with tier accent on left edge (4px vertical stripe), icon left-aligned, package name + tagline right of icon.

### Tier 1: Discovery — Teal (#14B8A6)

| # | Package | Icon Description | Tagline |
|---|---------|-----------------|---------|
| 1 | `geoinventory` | Magnifying glass over stacked layers | scan · document · diff |
| 2 | `geodata-health` | Heartbeat/EKG line transitioning green→yellow→red | staleness · health · monitoring |
| 3 | `sde-inspector` | Wrench + database cylinder with eye symbol | versions · compress · locks |
| 4 | `lyrx-tools` | Page icon peeling up to reveal JSON `{}` brackets | parse · compare · manage |

### Tier 2: ETL — Amber (#F59E0B)

| # | Package | Icon Description | Tagline |
|---|---------|-----------------|---------|
| 5 | `spatial-etl` | Three arrows flowing through a funnel | extract · transform · load |
| 6 | `oracle-spatial-etl` | Database cylinder → arrow → map pin | Oracle → spatial |
| 7 | `arcpy-etl-patterns` | Interlocking puzzle pieces (composable blocks) | truncate · upsert · delta |
| 8 | `format-bridge` | Two file icons connected by an arch/bridge | any format → any format |

### Tier 3: Quality — Rose (#F43F5E)

| # | Package | Icon Description | Tagline |
|---|---------|-----------------|---------|
| 9 | `spatial-validate` | Polygon with checkmark overlay | geometry · attributes · CRS |
| 10 | `fgdc-metadata` | Document with `<xml>` tag + quality badge | create · validate · score |
| 11 | `map-accessibility` | Eye with contrast-split circle as iris | WCAG 2.1 · contrast · CVD |
| 12 | `schema-enforcer` | Shield with YAML `---` brackets + lock | contracts · YAML · CI |

### Tier 4: Processing — Violet (#8B5CF6)

| # | Package | Icon Description | Tagline |
|---|---------|-----------------|---------|
| 13 | `georeg` | Tilted rectangle with crosshair registration marks | georeference · transform · QA |
| 14 | `permit-lifecycle` | Circular state diagram with 3 nodes + arrows | states · transitions · audit |
| 15 | `watershed-tools` | Dendritic stream network with watershed boundary | HUC · trace · overlay |
| 16 | `terrain-extract` | Mountain profile silhouette with sampling point | profile · zonal · slope |

### Tier 5: Tools — Sky (#0EA5E9)

| # | Package | Icon Description | Tagline |
|---|---------|-----------------|---------|
| 17 | `geoai-toolkit` | Brain with map pin growing from it | LLM + spatial |
| 18 | `arcpy-notify` | Bell with lightning bolt | monitor · alert · audit |
| 19 | `spatial-diff` | Two overlapping polygons with diff area highlighted | added · deleted · changed |
| 20 | `gis-project-scaffold` | Directory tree with hex-shaped root node | CLAUDE.md · ai-dev · scaffold |

---

## R Package Hex Stickers (10)

Standard hexb.in dimensions. Dark fill, tier-colored border, JetBrains Mono package name, icon centered, subtle "R" watermark in bottom corner at 10% opacity.

### Tier 1: Discovery — Teal border

| # | Package | Icon | Tagline |
|---|---------|------|---------|
| 1 | `geoinventoryr` | Same as Python geoinventory (magnifying glass + layers) | scan · document · diff |
| 2 | `spatialvalidate` | Checkmark inside polygon shape | check · validate · report |
| 3 | `schemaenforcer` | Shield with YAML `---` + lock | contracts · enforce · CI |
| 4 | `geodatahealth` | Heartbeat EKG line (same concept as Python) | staleness · health · monitor |

### Tier 2: ETL — Amber border

| # | Package | Icon | Tagline |
|---|---------|------|---------|
| 5 | `spatialetl` | Three arrows through funnel | extract · transform · load |
| 6 | `formatbridge` | Two files + bridge arch | any → any |

### Tier 3: Quality — Rose border

| # | Package | Icon | Tagline |
|---|---------|------|---------|
| 7 | `mapaccessibility` | Eye with contrast-split iris | WCAG · contrast · CVD |
| 8 | `kartocolors` | Painter's palette with 5 accessible color dots as map legend | accessible palettes |

### Tier 4+5: Processing & Tools — Violet/Sky border

| # | Package | Icon | Tagline |
|---|---------|------|---------|
| 9 | `watershedtools` | Dendritic stream network | HUC · trace · overlay |
| 10 | `geoaitools` | Brain + map pin | LLM + spatial |

---

## Implementation Approach

### Phase 1: SVG Templates

Create two Jinja2 SVG templates:

**Python banner template** (`banner_template.svg`):
- 1280 × 320px canvas
- Left: 4px tier-accent vertical stripe
- Left-center: icon (120 × 120px area)
- Center-right: package name (JetBrains Mono Bold, 48px, white)
- Below name: tagline (Source Sans 3 Light, 24px, 60% white opacity)
- Bottom-right: "chrislyonsKY" (Source Sans 3, 14px, 30% opacity)
- Background: tier-specific dark color

**R hex template** (`hex_template.svg`):
- hexb.in spec dimensions
- Tier-colored border (2px stroke)
- Dark fill
- Icon centered (60 × 60px area)
- Package name at top (JetBrains Mono Bold, auto-sized)
- Tagline at bottom (Source Sans 3 Light, small)
- "R" watermark bottom-right (10% opacity)

### Phase 2: Icon SVGs

Create 20 unique icon SVGs (line art, stroke-only, single accent color). Many are shared between Python and R versions.

Unique icons needed: **16** (some R packages share icons with their Python companions)

| Icon | Used By |
|------|---------|
| Magnifying glass + layers | geoinventory, geoinventoryr |
| Heartbeat EKG | geodata-health, geodatahealth |
| Wrench + database + eye | sde-inspector |
| Page peeling to JSON | lyrx-tools |
| Arrows through funnel | spatial-etl, spatialetl |
| Database → arrow → pin | oracle-spatial-etl |
| Puzzle pieces | arcpy-etl-patterns |
| Files + bridge arch | format-bridge, formatbridge |
| Polygon + checkmark | spatial-validate, spatialvalidate |
| Document + XML + badge | fgdc-metadata |
| Eye + contrast iris | map-accessibility, mapaccessibility |
| Shield + YAML + lock | schema-enforcer, schemaenforcer |
| Rectangle + crosshairs | georeg |
| State diagram | permit-lifecycle |
| Stream network | watershed-tools, watershedtools |
| Mountain profile | terrain-extract |
| Brain + map pin | geoai-toolkit, geoaitools |
| Bell + lightning | arcpy-notify |
| Overlapping polygons | spatial-diff |
| Directory tree + hex root | gis-project-scaffold |
| Palette + color dots | kartocolors (R-only) |

### Phase 3: Batch Generation

```python
# Python script to generate all banners
from jinja2 import Template
import yaml

config = yaml.safe_load(open("packages.yml"))
template = Template(open("banner_template.svg").read())

for pkg in config["python_packages"]:
    svg = template.render(**pkg)
    Path(f"{pkg['name']}/assets/banner.svg").write_text(svg)
    # Convert SVG → PNG via cairosvg or inkscape CLI
```

```r
# R script to generate all hex stickers
library(hexSticker)
packages <- yaml::read_yaml("packages.yml")$r_packages

for (pkg in packages) {
  sticker(
    pkg$icon_path,
    package = pkg$name,
    p_size = 20, p_color = "#FFFFFF", p_family = "JetBrains Mono",
    h_fill = pkg$bg_color, h_color = pkg$border_color,
    url = "chrislyonsKY", u_color = "#64748B",
    filename = file.path(pkg$name, "man/figures/logo.png")
  )
}
```

### Phase 4: Print Production

For conference stickers:
- **R hex stickers:** Export at 300 DPI, die-cut hexagon, matte finish
- **Python stickers:** Use the square PyPI logo (512×512) on round or square stickers
- **Org sticker:** The Chris Lyons master logo on a rectangular sticker (3" × 1.5")

Recommended vendor: StickerMule (hex die-cut option) or Sticker Giant

---

## File Locations per Package

### Python
```
{package}/
├── assets/
│   ├── banner.svg          # Source SVG (1280×320)
│   ├── banner.png          # Exported PNG for README
│   ├── logo.svg            # Square logo source (512×512)
│   ├── logo.png            # PyPI logo
│   └── social-preview.png  # GitHub social preview (1280×640)
```

### R
```
{package}/
├── man/
│   └── figures/
│       ├── logo.svg        # Source SVG (hex)
│       └── logo.png        # Exported PNG for README + pkgdown
├── inst/
│   └── figures/
│       └── sticker.svg     # Print-ready hex sticker
```
