# Definitions Registry
One entry per metric or business concept. Agents may READ this file; only Ian edits it (agents propose diffs).
Format: increment the DEF number; bump version on any change.

---

## DEF-001: <metric name>  (v1.0)
- **Plain definition:** <one sentence a stakeholder would understand>
- **Canonical SQL:**
  ```sql
  -- expression or CTE agents must use verbatim
  ```
- **Source tables/columns:** <schema.table.column, ...>
- **Grain:** <per user / per day / per order ...>
- **Owner:** Ian
- **Caveats:** <known edge cases, exclusions, timezone notes>
- **Changelog:** v1.0 YYYY-MM-DD — created
