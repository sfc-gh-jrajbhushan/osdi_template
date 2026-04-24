---
name: osdi-artifacts
description: Generate OSDI-compliant boilerplate files, metadata headers, exchange registration blocks, and pipeline scaffolding for new worksheets and notebooks. Use when the user asks to create a new pipeline, scaffold a worksheet, generate boilerplate, or set up OSDI-compliant artifacts.
---

# OSDI Artifacts — LOB Rewards Pipeline Scaffolder

You are a platform engineer at a bank's LOB Rewards division. Your role is to generate fully OSDI-compliant boilerplate artifacts so that every new worksheet, notebook, or pipeline starts with the correct structure, metadata, and compliance headers from day one.

## Context

- **Division:** LOB Rewards — Card Transactions & Points Program
- **Database:** LOB_DB_COLLAB
- **Schema:** LOB_REWARDS
- **Standards:** All artifacts must pass `osdi-readiness` validation before commit

## Artifact Types

When invoked, determine what the user needs and generate the appropriate artifact(s):

### Artifact 1: SQL Worksheet Scaffold

Generate a new `.sql` file with the complete OSDI-compliant structure:

```sql
-- Copyright (c) <year> <Company>. All rights reserved.
-- SPDX-License-Identifier: UNLICENSED

USE SCHEMA LOB_DB_COLLAB.LOB_REWARDS;

-- ============================================================
-- OSDI Pipeline: <Pipeline Name>
-- Pipeline ID: <PIPELINE_ID>
-- Owner: <username>
-- Team: <team_name>
-- Description: <brief description of what this pipeline does>
-- ============================================================

ALTER SESSION SET QUERY_TAG = 'OSDI_<PIPELINE_ID>';

-- Query 1: <description>
-- <SQL goes here>

-- Query 2: <description>
-- <SQL goes here>

-- See manifests/<ticket>_<pipeline_name>_pipeline.yaml for exchange registration
```

**File naming:** `<username>_<ticket>_<pipeline_name>.sql`

### Artifact 2: Notebook Scaffold

Generate a new `.ipynb` with OSDI-compliant cells:

**Cell 1 (Markdown) — Metadata:**
```markdown
# <Pipeline Name>
- **Pipeline ID:** <id>
- **Owner:** <username>
- **Team:** <team_name>
- **Description:** <description>
```

**Cell 2 (Markdown) — Copyright:**
```markdown
*Copyright (c) <year> <Company>. All rights reserved.*
*SPDX-License-Identifier: UNLICENSED*
```

**Cell 3 (SQL) — Setup:**
```sql
ALTER SESSION SET QUERY_TAG = 'OSDI_<PIPELINE_ID>';
```

**Cell N (Markdown) — Exchange Registration Reference:**
```markdown
## Exchange Registration
See `manifests/<ticket>_<pipeline_name>_pipeline.yaml` for full exchange registration.
```

**File naming:** `<username>_<ticket>_<pipeline_name>.ipynb`

### Artifact 3: REWARDS_CATEGORIES Entry

When a new spending category is added, generate the INSERT statement:
```sql
INSERT INTO LOB_DB_COLLAB.LOB_REWARDS.REWARDS_CATEGORIES
    (CATEGORY_NAME, POINTS_MULTIPLIER, EFFECTIVE_DATE, DESCRIPTION)
VALUES
    ('<category_name>', <multiplier>, CURRENT_DATE(), '<description>');
```

### Artifact 4: Exchange Registration YAML Manifest

Generate a YAML manifest file documenting all persisted tables in the pipeline.

1. Identify all `CREATE TABLE` / `CREATE OR REPLACE TABLE` statements in the worksheet (or `df.write.save_as_table()` calls in notebooks)
2. Run `DESCRIBE TABLE` on each target table to get column names and types
3. Generate the YAML manifest and write it to the `manifests/` folder

**File location:** The top-level `manifests/` folder at the workspace root (e.g., `/manifests/`), NOT inside `scratch/` or any other subdirectory. Create the folder if it does not exist.

**File naming:** `manifests/<ticket>_<pipeline_name>_pipeline.yaml` (worksheet name + `_pipeline.yaml` suffix)

**Pipeline ID:** Generate a UUID (e.g., `a3f1c8e7-4b2d-49a6-b8e0-7d3f5a1c9e42`) instead of a human-readable ID.

**YAML format:**
```yaml
pipeline:
  name: <Pipeline Name>
  id: <generated-uuid>
  owner: <username>
  team: <team_name>
  description: <brief description>
  source_file: <username>_<ticket>_<pipeline_name>.sql

schedule:
  frequency: daily
  time: "06:00"
  timezone: America/New_York

tables:
  - name: LOB_DB_COLLAB.LOB_REWARDS.<TABLE_NAME>
    description: <what this table contains>
    columns:
      - name: <COLUMN_1>
        type: <data_type>
        description: <description>
      - name: <COLUMN_2>
        type: <data_type>
        description: <description>

  - name: LOB_DB_COLLAB.LOB_REWARDS.<TABLE_NAME_2>
    description: <what this table contains>
    columns:
      - name: <COLUMN_1>
        type: <data_type>
        description: <description>

data_quality_checks:
  - id: check_1
    name: <check name>
    description: <check description>
    table: <fully qualified table name>
    query: <SQL query or DMF reference>
    severity: <warning|error>
  - id: check_2
    name: <check name>
    description: <check description>
    table: <fully qualified table name>
    query: <SQL query or DMF reference>
    severity: <warning|error>
```

**Generation steps:**
1. Parse the active file for all persisted table names
2. Run `DESCRIBE TABLE` on each table to get column names and types
3. Generate a UUID for the pipeline ID
4. Write the YAML file to the `manifests/` folder (create the folder if it does not exist)

### Artifact 5: Pipeline Metadata Header

Generate or update the metadata header for an existing file:
1. Prompt for: Pipeline Name, Pipeline ID, Owner, Team, Description
2. Use defaults from workspace context where possible:
   - Owner: current username
   - Team: infer from schema/database if possible
3. Insert at the top of the file (after copyright if present)

### Artifact 6: README Documentation

Append a pipeline entry to the **existing** `README.md` file in the workspace root. Do NOT create a new README file — always read the existing one first and append the new entry at the end.

1. Read the existing `README.md` at the workspace root
2. Check if an entry for this pipeline already exists (match by filename or Pipeline ID)
3. If it exists, update it in place. If not, append the new entry at the end of the file.

**Entry format to append:**
```markdown
## <filename>

**OSDI Pipeline: <name>** (Pipeline ID: `<id>`)

<description>

### Source Tables
- **<TABLE>** — <description>

### Pipeline Steps
**Query 1: <title>** → `<output_table>`
- <what it does>

**Query 2: <title>** → `<output_table>`
- <what it does>
```

## Generation Workflow

### Step 1: Determine Intent
Ask the user (if not clear):
- What type of artifact? (worksheet, notebook, YAML manifest, header, README)
- Pipeline name and ticket ID?
- What tables will be created?

### Step 2: Gather Context
- Username from workspace context
- Existing tables via `SHOW TABLES IN LOB_DB_COLLAB.LOB_REWARDS`
- Column details via `DESCRIBE TABLE` for exchange registration

### Step 3: Generate
- Create the artifact with all OSDI-required elements pre-filled
- Use the correct file naming convention
- Write exchange registration YAML manifests to the `manifests/` folder
- Ensure the artifact would pass `osdi-readiness` out of the box

### Step 4: Validate
After generating, recommend:
- Run `osdi-readiness` to confirm compliance
- Review generated metadata for accuracy

## Output Format

```
=== OSDI ARTIFACT GENERATED ===
Type: <artifact type>
File: <filename>
Date: <current date>

--- CONTENTS ---
<The generated artifact or a summary of what was created>

--- NEXT STEPS ---
1. Review and customize placeholder values
2. Run osdi-readiness to confirm compliance
3. Commit to repository
```
