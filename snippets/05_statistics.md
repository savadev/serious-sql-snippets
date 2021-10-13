## Statistics

### 1. Arithmetic Mean or Average.
The mean is commonly used as a location summary statistic to show the central tendancy for a set of observations. Note that the mean can only be calculated for numbers and cannot be used on any other data type.

The following mathematical equation is commonly used to show the mean calculation.

```sql
SELECT
  AVG(measure_value)
FROM health.user_logs;
```
 Important to know that we have different types of measures. Take a look:

 ```sql
 SELECT
  measure,
  COUNT(*) AS counts,
  ROUND(
    AVG(measure_value),
    2
  ) as average
FROM health.user_logs
GROUP BY measure;
```

### 2. Median.
The median shows the central location based off the values of the middle values. If we have a collection of values N:

* If N is odd, the median is the middle.
* If N is even, the median is the AVG of the two central values.

Median is the 50th percentile value.

```sql
WITH sample_data (example_values) AS (
 VALUES
 (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
)
SELECT
  AVG(example_values) AS mean_value,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value,
  MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
from sample_data;
```

### 3. Mode.
The mode focuses on the most frequent values. It could be more than one value.

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
  MODE() WITHIN GROUP (ORDER BY measure_value) AS mode_value,
  AVG(measure_value) as mean_value
FROM health.user_logs
WHERE measure = 'weight';
```

### 2. Min, Max, Range

```sql
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

