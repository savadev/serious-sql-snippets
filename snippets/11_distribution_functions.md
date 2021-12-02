## Distribution functions

### 1. Summary statistics.

```sql
SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value,
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    -- this function actually returns a float which is incompatible with ROUND!
    -- we use this cast function to convert the output type to NUMERIC
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';
```

Example questions:
* Does it make sense to have such low minimum values and such a high value?
* Why is the average value 28,786kg but the median is 75.98kg?
* The standard deviation of values is WAY too large at 1,062,759kg

### 2. Breakdown into percentiles
```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;
```

#### Critical Thinking
It is a weight value in KG units.

When we think of those small values in the 1st percentile under 29kg, a few things should come to mind:
* Maybe there were some incorrectly measured values - leading to some 0kg weight measurements
* Perhaps some of the low weights under 29kg were actually valid measurements from young children who were part of the customer base
  
For the 100th percentile we could consider:
* Does that 136KG floor value make sense?
* How many error values were there actually in the dataset?