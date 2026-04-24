# OSDI Template Repository

Standard project template for OSDI-managed pipelines. Clone this repo to get started.

## Structure
- `.snowflake/` — Cortex Code Skills (auto-loaded)
- `pipelines/` — SQL worksheets and notebooks
- `manifests/` — Registration artifacts (dataset/pipeline YAML)
- `reports/` — Output reports and validation results
- `scratch/` — Working area (not promoted to production)

## Getting Started
1. Create a new repo from this template
2. Connect it to a Snowflake Workspace
3. Cortex Code Skills are available immediately — try `@osdi-readiness`

## Included Skills
- **OSDI Readiness Check** — Validates code against OSDI standards before commit
- **OSDI Artifacts** — Generates registration YAML for datasets and pipelines
- **OSDI Performance** — Reviews SQL for optimization opportunities
- **OSDI Troubleshoot** — Assists with pipeline debugging
- **OSDI Unit Test** — Scaffolds test templates with sample data
