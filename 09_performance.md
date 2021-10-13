#### 0.Measure

To analyze the performance type 'EXPLAIN ANALYZE' before the query:

``sql
EXPLAIN ANALYZE

WITH min_max_values AS (
SELECT
  MIN(measure_value) as minimum_value,
  MAX(measure_value) as maximum_value
FROM health.user_logs
WHERE measure = 'weight'
)

SELECT
  minimum_value,
  maximum_value,
  maximum_value - minimum_value AS range
FROM min_max_values;
```




#### 1.Don't use a aggregate function twice, create a CTE.

``sql
EXPLAIN ANALYZE

WITH min_max_values AS (
SELECT
  MIN(measure_value) as minimum_value,
  MAX(measure_value) as maximum_value
FROM health.user_logs
WHERE measure = 'weight'
)

SELECT
  minimum_value,
  maximum_value,
  maximum_value - minimum_value AS range
FROM min_max_values;
```

is better than 

```sql
SELECT
  MIN(measure_value) as min,
  MAX(measure_value) as max,
  MAX(measure_value) - MIN(measure_value) as range
FROM health.user_logs
WHERE measure = 'weight';
```

Because it is filtering rows before the query with the CTE.