# DLC-001: Column Obscurer

## Concept
Take a user-provided SQL query and break the 1:1 link between output columns
and internal naming, so shared queries/results don't leak schema, business
logic, or personal data. For each column the agent recommends one of:

- REMOVE    — column shouldn't leave the building at all
- HASH      — identifier needed as a join key; replace with salted SHA2
- GENERALIZE— quasi-identifier; bucket/truncate (age → age band, zip → zip3)
- ALIAS     — name reveals business logic; rename to a neutral token

It then produces a rewritten query: the original wrapped as a CTE with an
outer SELECT applying every transform and generic aliases (id_01, dim_01,
metric_01...).

## The mapping file (read this part)
Every run writes outputs/mappings/<task>.map.md — the real-name → alias key.
That file re-identifies everything, so it is GITIGNORED by default and stays
on the machine that made it. If your repo is private and you want mappings to
sync across devices, remove the `outputs/mappings/` line from .gitignore —
an explicit choice, not a default.

## Usage
In Claude Code: "Use column-obscurer on outputs/TASK-xxx.sql" or paste raw
SQL. Review the recommendation table, approve, receive the obscured query +
mapping. Human QA still applies: you confirm no sensitive column survived.

## Sensitivity tiers the agent applies
1. Direct identifiers (email, phone, name, ip, account no.) → REMOVE or HASH
2. Quasi-identifiers (birth date, zip, precise geo, age)    → GENERALIZE
3. Business-sensitive names (margin, churn_score, fraud_*)  → ALIAS
4. Neutral columns                                           → ALIAS (consistency)
