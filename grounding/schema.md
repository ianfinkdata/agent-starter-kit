# Schema Snapshot
Cloud Claude Code sessions cannot reach the local MySQL database. They ground
SQL against THIS file instead. Refresh it locally whenever schema changes:

    mysql yourdb --batch -e "SHOW TABLES;" 
    mysql yourdb -e "SHOW CREATE TABLE <table>\G"

Paste the relevant CREATE TABLE statements below, one section per table.
Last refreshed: <date>

---
## <table_name>
```sql
-- CREATE TABLE output here
```
