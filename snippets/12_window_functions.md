## Window functions

### 1. For Sorting Values.

```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  measure_value,
  -- these are examples of window functions below
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
  RANK() OVER (ORDER BY measure_value DESC) as rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;
```

![alt text](https://github.com/ismaelcazalilla/serious-sql-snippets/blob/eb268c65e8352a9aa06a9974fbe26122777ae699/assets/images/window_functions_sorting.png)

Differences between ROW_NUMBER, RANK and DENSE_RANK
* ROW_NUMBER: assign uniques and incremental numbers from 1 for each partition.
* RANK: assign NOT incremental either uniques numbers.
* RANK_DENSE: assign incremental, but NOT uniques numbers.

It looks like there are 3 huge values (maybe outliers).
39642120
39642120
576484
200.487664

Maybe 200 is a valid value, but not the rest. The process should be repeated with the 1st percentile.