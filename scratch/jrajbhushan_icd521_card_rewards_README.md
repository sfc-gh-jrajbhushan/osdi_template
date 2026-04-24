## jrajbhushan_icd521_card_rewards.sql

**OSDI Pipeline: Rewards Points Calculation** (Pipeline ID: `REWARDS_CALC_001`)

Categorize card transactions and calculate reward points per customer based on spending category multipliers.

### Source Tables
- **LOB_DB_COLLAB.LOB_REWARDS.CARD_TRANSACTIONS** — Raw card transaction records including merchant, amount, MCC code, and approval status
- **LOB_DB_COLLAB.LOB_REWARDS.REWARDS_CATEGORIES** — Spending category definitions with points multipliers

### Pipeline Steps

**Query 1: Categorize Transactions** → `LOB_DB_COLLAB.LOB_REWARDS.STG_CATEGORIZED_TRANSACTIONS`
- Filters to APPROVED transactions only
- Maps MCC codes to human-readable spending categories (Dining, Grocery, Travel, Electronics, General)
- Special handling for Uber Eats (MCC 5499) as a distinct Dining subcategory

**Query 2: Calculate Reward Points** → `LOB_DB_COLLAB.LOB_REWARDS.STG_TRANSACTION_REWARDS`
- Joins categorized transactions with the rewards multiplier table
- Computes POINTS_EARNED as TRANSACTION_AMOUNT × POINTS_MULTIPLIER (rounded to whole number)
