# Copilot Instructions — Chris Lyons Ecosystem Hub

This is the `.github` repo for the chrislyonsKY GitHub organization.
It contains documentation, shared test fixtures, reusable CI/CD workflows,
and AI-development infrastructure for 77 packages across 6 registries.

## What This Repo Contains

- `docs/` — Ecosystem plans, registry compliance guides, branding specs, package catalogs
- `shared/test-fixtures/` — Language-agnostic test data (GeoJSON, YAML schemas, palettes, rasters)
- `shared/workflow-templates/` — Reusable GitHub Actions workflows for publishing
- `ai-dev/` — Agent prompts, guardrails, decisions, skills for AI-assisted development
- `profile/README.md` — The org profile shown on github.com/chrislyonsKY

## Key Rules

- This repo contains NO package source code
- All Markdown docs must be well-structured with clear headings
- Workflow templates must use `workflow_call` trigger for reusability
- Test fixtures must be language-agnostic (GeoJSON, YAML, JSON, GeoTIFF)
- Never hardcode credentials in workflow templates

## Registries We Target

CRAN (R), PyPI (Python), plugins.qgis.org (QGIS), crates.io (Rust), NuGet (.NET), npm (JS/TS)
