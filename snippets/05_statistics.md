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

### 3. Mode.
The mode focuses on the most frequent values. It could be more than one value.