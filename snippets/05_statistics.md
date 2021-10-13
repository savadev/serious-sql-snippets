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


#### 3.Variance & Standard Deviation

These two important values are used to describe the “spread” of the data about the mean value. Also, the variance is simply the square of the standard deviation

To explain this formula in simple terms - the variance is the sum of the (difference between each X value and the mean) squared, divided by N, the total number of values.

```sql
WITH sample_data (example_values) AS (
 VALUES
 (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
)
SELECT
  ROUND(VARIANCE(example_values), 2) AS variance_value,
  ROUND(STDDEV(example_values), 2) AS standard_dev_value,
  ROUND(AVG(example_values), 2) AS mean_value,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value,
  MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
FROM sample_data;
```

#### 4.Summary statistics

```sql
SELECT
  ROUND( MIN(measure_value), 2) AS minimum_value,
  ROUND( MAX(measure_value), 2) AS maximum_value,
  ROUND( AVG(measure_value), 2) AS mean_value,
  ROUND(
    PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value) :: NUMERIC,
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP(ORDER BY measure_value):: NUMERIC,
    2
  ) AS mode,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';
```


EXERCISES

1. What is the average, median and mode values of blood glucose values to 2 decimal places?

```sql
SELECT
  ROUND(
    AVG(measure_value),
    2
  ) AS mean_value,
  
  ROUND(
    PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value)::NUMERIC,
    2
  ) AS median_value,
  
  ROUND(
    MODE() WITHIN GROUP(ORDER BY measure_value),
    2
  ) AS mode
FROM health.user_logs
WHERE measure = 'blood_glucose';
```

2. What is the most frequently occuring measure_value value for all blood glucose measurements?

```sql
SELECT
  measure_value,
  COUNT(*) AS frequency
FROM health.user_logs
WHERE measure = 'blood_glucose'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

3. Calculate the 2 Pearson Coefficient of Skewness for blood glucose measures given the following formulas:

```sql
WITH statistic_values AS (
  SELECT
    AVG(measure_value) AS mean_value,
    PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value) AS median_value,
    MODE() WITHIN GROUP(ORDER BY measure_value) AS mode_value,
    STDDEV(measure_value) AS standard_deviation
  FROM health.user_logs
  WHERE measure = 'blood_glucose'
)

SELECT
  ROUND(
    (mean_value - mode_value) / standard_deviation,
    2
  ) AS pearson_coefficent_1,
  
  ROUND(
    3 * (mean_value - median_value)::NUMERIC / standard_deviation,
    2
  ) AS pearson_coefficient_2
FROM statistic_values;
```
