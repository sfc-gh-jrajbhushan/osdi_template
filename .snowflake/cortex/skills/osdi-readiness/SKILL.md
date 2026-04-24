---
name: osdi-readiness
description: Validate worksheet or file content against OSDI git rule enforcement policies. Use when the user asks to check OSDI readiness, validate git rules, check compliance, or run osdi_readiness.
---

# OSDI Readiness Validator

You are an OSDI compliance validator. When invoked, read the target file (default: the currently active file) and evaluate it against ALL of the rules below. For each rule, report PASS or FAIL with a brief explanation. At the end, provide a summary of violations and actionable remediation steps.

## File Type Detection

First, determine the file type:
- **SQL Worksheet** (`.sql`) — apply all rules as written below
- **Notebook** (`.ipynb`) — apply all rules with notebook-specific adaptations described in each rule

For notebooks, scan ALL cells (code cells, markdown cells, and raw cells) as the content surface. Cell numbers should be referenced instead of line numbers (e.g., "Cell 3 (code)").

## Rules to Validate

### Rule 1: No Secrets or Credentials
Scan for patterns that indicate hardcoded secrets, API keys, tokens, or passwords:
- Strings matching patterns like `password=`, `secret=`, `token=`, `api_key=`, `AWS_ACCESS_KEY`, `PRIVATE_KEY`
- Base64-encoded blobs that may be credentials
- Any `CREATE ... SECRET`, `ALTER ... SET PASSWORD`, or similar DDL with literal credential values
- **Notebooks:** Also check for credentials in `%pip install` commands (private index URLs), `snowflake.connector.connect()` calls with hardcoded credentials, and any cell outputs that may have leaked secrets
- **PASS** if none found. **FAIL** if any pattern detected — list each occurrence with line/cell number.

### Rule 2: No PII or Customer Data
Check for personally identifiable information in SQL, comments, or test fixtures:
- Real email addresses, phone numbers, SSNs, credit card numbers
- Names that appear to be real customer data (not generic placeholders like 'John Doe')
- Hardcoded values in WHERE clauses or INSERT statements that look like real PII
- **Notebooks:** Also check markdown cells for real names/emails, and cell outputs for PII data that should be cleared before commit
- **PASS** if none found. **FAIL** if any pattern detected — list each occurrence with line/cell number.

### Rule 3: No Hardcoded Connection Strings or Environment-Specific Configs
Look for:
- JDBC/ODBC connection strings (e.g., `jdbc:snowflake://`, `snowflake://`)
- Hardcoded hostnames, IP addresses, or account locators used as connection endpoints
- Environment-specific references like `dev`, `staging`, `prod` in hostnames or database names that should be parameterized
- **Notebooks:** Also check for hardcoded `account`, `host`, or `connection_params` dicts in Python cells
- **PASS** if none found. **FAIL** if any pattern detected — list each occurrence with line/cell number.

### Rule 4: Approved File Types Only
Check the file extension against the approved list:
- **Approved:** `.sql`, `.py`, `.yml`, `.yaml`, `.md`, `.txt`, `.json`, `.csv`, `.ipynb`, `.toml`
- **Not approved:** `.exe`, `.dll`, `.so`, `.class`, `.jar`, `.bin`, `.o`, `.pyc`, `.zip`, `.tar`, `.gz` and any other binary/compiled artifacts
- **PASS** if file type is approved. **FAIL** if not — state the file type and recommend removal.

### Rule 5: Mandatory Metadata
Check that the file contains a metadata header block with the following fields:
- **Pipeline owner** (e.g., `-- Owner:` or `-- Pipeline Owner:`)
- **Team** (e.g., `-- Team:`)
- **Description** (e.g., `-- Description:` or a comment block describing purpose)
- A structured header format (comment block with separators) is expected
- **Notebooks:** The first cell must be a markdown cell containing the metadata in this format:
  ```markdown
  # <Pipeline Name>
  - **Pipeline ID:** <id>
  - **Owner:** <owner>
  - **Team:** <team>
  - **Description:** <description>
  ```
- **PASS** if all three metadata fields are present. **FAIL** if any are missing — list which fields are absent.

### Rule 6: File Naming Convention
Check that the filename follows the required pattern based on file type:
- **SQL Worksheets:** `<username>_<ticket>_<pipeline>.sql` (e.g., `jrajbhushan_icd521_card_rewards.sql`)
- **Notebooks:** `<username>_<ticket>_<pipeline>.ipynb` (e.g., `jrajbhushan_icd521_card_rewards.ipynb`)
- Format: `{username}_{ticket_id}_{pipeline_name}.<ext>` (all lowercase, underscores as separators)
- Each segment must be present and non-empty
- **PASS** if the filename matches the convention. **FAIL** if not — show the current filename and the expected format.

### Rule 7: Branch Naming Conventions and PR Template
This rule applies to git context, not file content. If git metadata is available:
- Branch name should follow pattern: `feature/`, `bugfix/`, `hotfix/`, `release/`, or `chore/` prefix
- If no git context is available, mark as **SKIPPED** with a note to verify branch naming before committing.

### Rule 8: License/Copyright Headers
Check that the file begins with (or contains near the top) a license or copyright header:
- **SQL:** Look for `-- Copyright`, `-- License`, `-- (c)`, `-- SPDX-License-Identifier`, or equivalent
- **Notebooks:** The first cell (or second cell if first is metadata) should contain a copyright notice, either as a markdown cell or a comment in a code cell (e.g., `# Copyright (c) 2026 <Company>. All rights reserved.`)
- **PASS** if a header is found. **FAIL** if missing — recommend adding a standard copyright header.

### Rule 9: Exchange Registration — Persisted Table Documentation
Check that every `CREATE TABLE` or `CREATE OR REPLACE TABLE` statement (in SQL or in notebook code cells) has a corresponding exchange registration block at the end of the file. For notebooks, also check for `df.write.save_as_table()` or `session.sql("CREATE...")` calls that persist tables. The registration block must document each persisted table with:
- **Table name** (fully qualified)
- **Description** of the table's purpose
- **Columns** — list each column with its name and a brief description

Expected format at the end of the file:
```sql
-- ============================================================
-- EXCHANGE REGISTRATION
-- ============================================================
-- Table: DATABASE.SCHEMA.TABLE_NAME
-- Description: <what this table contains>
-- Columns:
--   COLUMN_1: <description>
--   COLUMN_2: <description>
--   ...
-- ============================================================
```
- **Notebooks:** The registration block should be in the last markdown cell of the notebook using the same structure (with markdown formatting instead of SQL comments)
- **PASS** if every persisted table has a matching registration block. **FAIL** if any table is missing — list each unregistered table and provide the recommended block to add.

### Rule 10: Code Scanning — Secret Detection Patterns
Perform a deeper scan simulating tools like TruffleHog:
- High-entropy strings (long alphanumeric strings 20+ chars that look like tokens/keys)
- AWS key patterns (`AKIA...`), Azure connection strings, GCP service account JSON fragments
- Private key blocks (`-----BEGIN ... PRIVATE KEY-----`)
- Generic secret patterns in variable assignments
- **Notebooks:** Also scan for secrets in `!pip install` commands, `os.environ` assignments with literal values, and any cell outputs containing tokens or keys
- **PASS** if none found. **FAIL** if any pattern detected — list each occurrence with line/cell number and pattern type.

## Report Persistence

After generating the readiness report, you MUST write it to a file in the same workspace/repo:

1. **File naming:** `<worksheet_name_without_extension>_OSDI_readiness_report_<timestamp>.md`
   - Timestamp format: `YYYYMMDD_HHMMSS` (use today's date and current time)
   - Example: For `icd521_card_rewards.sql` → `icd521_card_rewards_OSDI_readiness_report_20260423_143022.md`
   - Example: For `sample.ipynb` → `sample_OSDI_readiness_report_20260423_143022.md`

2. **File location:** The `reports/` folder relative to the workspace root (e.g., `/scratch/reports/`). If the `reports/` folder does not exist, create it first.

3. **Action:** Use the `write` tool to create this file with the full report content (in markdown format). This file serves as an audit trail of compliance checks and MUST be committed alongside the code.

## Output Format

After evaluating all rules, produce output in this format:

```
=== OSDI READINESS REPORT ===
File: <filename>
Date: <current date>

Rule 1 — No Secrets/Credentials:        [PASS/FAIL]
Rule 2 — No PII/Customer Data:          [PASS/FAIL]
Rule 3 — No Hardcoded Connections:       [PASS/FAIL]
Rule 4 — Approved File Types:            [PASS/FAIL]
Rule 5 — Mandatory Metadata:             [PASS/FAIL]
Rule 6 — File Naming:               [PASS/FAIL]
Rule 7 — Branch Naming:                  [PASS/FAIL/SKIPPED]
Rule 8 — License/Copyright Header:       [PASS/FAIL]
Rule 9 — Exchange Registration:          [PASS/FAIL]
Rule 10 — Secret Detection Scan:         [PASS/FAIL]

Overall: X/10 rules passed

--- VIOLATIONS ---
<For each FAIL, list the rule, the issue, the line number(s), and a recommended fix>

--- NOTEBOOK-SPECIFIC WARNINGS (if applicable) ---
<Check for and flag these additional notebook concerns:>
- Cell outputs containing data that should be cleared before commit
- Cells with !pip or %pip installing packages from non-standard indices
- Unused imports or dead code cells
- Missing cell execution order (cells should run top-to-bottom without errors)
- Large dataframe displays that may contain sensitive data

--- REMEDIATION STEPS ---
<Numbered list of specific actions the user should take to achieve full compliance>
```
