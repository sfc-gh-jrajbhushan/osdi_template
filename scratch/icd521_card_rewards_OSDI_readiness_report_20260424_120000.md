# OSDI Readiness Report

```
=== OSDI READINESS REPORT ===
File: icd521_card_rewards.sql
Date: 2026-04-24

Rule 1  — No Secrets/Credentials:        PASS
Rule 2  — No PII/Customer Data:          PASS
Rule 3  — No Hardcoded Connections:       PASS
Rule 4  — Approved File Types:            PASS
Rule 5  — Mandatory Metadata:             FAIL
Rule 6  — File Naming:                    FAIL
Rule 7  — Branch Naming:                  SKIPPED
Rule 8  — License/Copyright Header:       FAIL
Rule 9  — Exchange Registration:          FAIL
Rule 10 — Secret Detection Scan:          PASS

Overall: 5/10 rules passed (1 skipped)
```

## VIOLATIONS

### Rule 5 — Mandatory Metadata (FAIL)
- **Issue:** The header block (lines 1–7) includes `Owner` and `Description` but is missing the **Team** field.
- **Location:** Lines 1–7

### Rule 6 — File Naming Convention (FAIL)
- **Issue:** Filename is `icd521_card_rewards.sql`. Expected format: `<username>_<ticket>_<pipeline>.sql`. The username prefix is missing.
- **Current:** `icd521_card_rewards.sql`
- **Expected:** `jrajbhushan_icd521_card_rewards.sql`

### Rule 8 — License/Copyright Header (FAIL)
- **Issue:** No copyright or license header found at the top of the file. Expected a `-- Copyright`, `-- License`, `-- (c)`, or `-- SPDX-License-Identifier` comment near the top.
- **Location:** Top of file (before line 1)

### Rule 9 — Exchange Registration (FAIL)
- **Issue:** The file creates 2 persisted tables but has no exchange registration block at the end:
  1. `LOB_DB_COLLAB.LOB_REWARDS.STG_CATEGORIZED_TRANSACTIONS` (line 12)
  2. `LOB_DB_COLLAB.LOB_REWARDS.STG_TRANSACTION_REWARDS` (line 32)
- **Location:** End of file — registration block is missing entirely

## REMEDIATION STEPS

1. **Add Team metadata (Rule 5):** Add a `-- Team:` line to the header block. Example:
   ```sql
   -- Team: LOB Rewards Engineering
   ```

2. **Rename the file (Rule 6):** Rename the file to include your username prefix:
   ```
   jrajbhushan_icd521_card_rewards.sql
   ```

3. **Add a copyright header (Rule 8):** Add a copyright/license comment at the very top of the file, before the metadata block:
   ```sql
   -- Copyright (c) 2026 <Company>. All rights reserved.
   -- SPDX-License-Identifier: <license>
   ```

4. **Add Exchange Registration block (Rule 9):** Append the following registration block to the end of the file:
   ```sql
   -- ============================================================
   -- EXCHANGE REGISTRATION
   -- ============================================================
   -- Table: LOB_DB_COLLAB.LOB_REWARDS.STG_CATEGORIZED_TRANSACTIONS
   -- Description: Approved card transactions categorized by spending type using MCC codes
   -- Columns:
   --   TRANSACTION_ID: Unique identifier for the transaction
   --   ACCOUNT_ID: Customer account identifier
   --   MERCHANT_NAME: Name of the merchant
   --   MCC_CODE: Merchant Category Code
   --   TRANSACTION_AMOUNT: Dollar amount of the transaction
   --   TRANSACTION_DATE: Date the transaction occurred
   --   SPENDING_CATEGORY: Derived category (Dining, Grocery, Travel, Electronics, General)
   -- ============================================================
   -- Table: LOB_DB_COLLAB.LOB_REWARDS.STG_TRANSACTION_REWARDS
   -- Description: Transaction-level reward points calculated using category multipliers
   -- Columns:
   --   TRANSACTION_ID: Unique identifier for the transaction
   --   ACCOUNT_ID: Customer account identifier
   --   MERCHANT_NAME: Name of the merchant
   --   SPENDING_CATEGORY: Spending category of the transaction
   --   TRANSACTION_AMOUNT: Dollar amount of the transaction
   --   TRANSACTION_DATE: Date the transaction occurred
   --   POINTS_MULTIPLIER: Reward multiplier for the spending category
   --   POINTS_EARNED: Calculated reward points for the transaction
   -- ============================================================
   ```

5. **Verify branch naming (Rule 7):** Before committing, ensure your branch follows the pattern `feature/`, `bugfix/`, `hotfix/`, `release/`, or `chore/`.
