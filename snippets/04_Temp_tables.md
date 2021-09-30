## Temporary tables
This is a very common approach when you know that you will only be analyzing the deduplicated dataset, and you will ignore the original one with duplicates.

The main benefit of using temporary tables is removing the need to always run the same DISTINCT command everytime you want to run a query on the deduplicated records.

### Example query
```sql
DROP TABLE IF EXISTS deduplicated_user_logs;

CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;
```

After the temp. table is created we can take a look:

```sql
SELECT *
FROM deduplicated_user_logs
LIMIT 10;
```