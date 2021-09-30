## Clean Data

### Introduction
Overall inspection:

```sql
SELECT * FROM health.user_logs;
```

```sql
SELECT
  COUNT(DISTINCT id)
FROM
  health.user_logs;
```


Inspection per column (freequencies and percentages)
```sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY percentage DESC;
```

Inspect by user (id), to know who are the users with more meditions:
```sql
SELECT
  id,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT (*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY id
ORDER BY percentage DESC
LIMIT 10;
```

### 1. Find NULL values.
```sql
SELECT
  diastolic,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT (*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY diastolic
ORDER BY percentage DESC
LIMIT 10;
```

There are a lot of null's and 0 values. We need to go deeper into specific values.

Then we can count the number and frequency of null values

```sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE diastolic IS NULL
GROUP BY measure;

SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE systolic IS NULL
GROUP BY measure;
 ```

### 2. How to deal with duplicates?
* Remove them in a SELECT statement
* Recreating a “clean” version of our dataset
* Identify exactly which rows are duplicated for further investigation or
* Simply ignore the duplicates and leave the dataset alone
  

#### 1.1 Detecting duplicates.

We can use 3 options:
* Subquery
* CTE
* Temporary table

When should be used one instead of each other? We should ask this question:

Will I need to use the deduplicated data later?

* Yes: Temporary tables
* No: CTE

#### 1.1.1 CTE
```sql
WITH uniques AS (
  SELECT DISTINCT *
  FROM health.user_logs
)


SELECT
  COUNT(*) AS total
FROM uniques;
```

#### 1.1.2 Temporary table
```sql
DROP TABLE IF EXISTS temp_unique_health;

CREATE TEMP TABLE temp_unique_health AS
SELECT DISTINCT *
FROM health.user_logs;

SELECT COUNT(*) 
FROM temp_unique_health;
```

Comparing the numbers of COUNT(*) and our new query we see duplicates.

### 2. Identifying duplicates.
We want to see:
* Duplicates rows.
* Unique duplicate rows (only one time)
* Duplicate counts (how many times a single row appears in dataset)


#### 2.1 Group by counts on all columns
As we can see - there are different ways to approach this simple problem of returning duplicate rows depending on what is required.

We can implement a single SQL statement that will help us achieve both outputs and also something even more useful!

The trick is to use a GROUP BY clause which has every single column in the grouping element and a COUNT aggregate function - this is an elegant solution to quickly find the unique combinations of all the rows.

```sql
SELECT 
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1
ORDER BY frequency DESC;
```

Using a CTE:

```sql
WITH health_duplicate_records AS (
  SELECT
    id,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    measure,
    measure_value,
    systolic,
    diastolic
  ORDER BY frequency DESC
)

SELECT
  id,
  measure,
  frequency
FROM health_duplicate_records
WHERE frequency > 1
LIMIT 10;
```

### 3. Ignoring duplicate values
For context, this real world messy dataset captures data taken from individuals logging their health measurements via an online portal throughout the day.

For example, multiple measurements can be taken on the same day at different times, but you may notice this information is missing as the log_date column does not show timestamp values!

Maybe is not a duplicate value after all. THINK CRITICALLY!!

#### 4. EXERCISES

##### 4.1. Which id value has the most number of duplicate records in the health.user_logs table?

```sql
WITH duplicated_records AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    diastolic,
    systolic,
    COUNT (*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    diastolic,
    systolic
  ORDER BY frequency DESC
)
  

SELECT
  id,
  SUM(frequency) AS total
FROM duplicated_records
WHERE frequency > 1
GROUP BY id
ORDER BY total DESC
LIMIT 5;
```

##### 4.2 Which log_date value had the most duplicate records after removing the max duplicate id value from question 1?

```sql
WITH duplicated_records AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    diastolic,
    systolic,
    COUNT (*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    diastolic,
    systolic
  ORDER BY frequency DESC
)
  

SELECT
  log_date,
  SUM(frequency) AS total_duplicate_rows
FROM duplicated_records
WHERE frequency > 1
AND id != '054250c692e07a9fa9e62e345231df4b54ff435d'
GROUP BY log_date
ORDER BY total_duplicate_rows DESC
LIMIT 5;
```

##### 4.3. Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?

```sql 
SELECT
  measure_value,
  COUNT(measure_value) AS frequency
FROM health.user_logs
WHERE measure = 'weight'
GROUP BY measure_value
ORDER BY frequency DESC
LIMIT 10;
```

##### 4.4. How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?

```sql
WITH duplicate_rows AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  WHERE measure = 'blood_pressure'
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
  ORDER BY frequency DESC
)

SELECT
  COUNT(*) AS single_duplicate_rows,
  SUM(frequency) AS total_duplicate_records
FROM duplicate_rows
WHERE frequency > 1;
```

##### 4.5. What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?

```sql
WITH all_measure_values AS (
  SELECT
    measure_value,
    COUNT(*) AS total_records,
    SUM(COUNT(*)) OVER () AS overall_total
  FROM health.user_logs
  WHERE measure = 'blood_pressure'
  GROUP BY measure_value
  ORDER BY 2 DESC
)

SELECT
  measure_value,
  total_records,
  overall_total,
  ROUND(
    100 * total_records::NUMERIC / overall_total,
    2
  ) AS percentage
FROM all_measure_values
ORDER BY percentage DESC;
```

##### 4.6. What percentage of records are duplicates in the health.user_logs table?

```sql 
WITH unique_records AS (
  SELECT DISTINCT * FROM health.user_logs
)

SELECT
  ROUND(
    100 * (
      (SELECT COUNT(*) FROM health.user_logs) - 
      (SELECT COUNT(*) FROM unique_records)
    )::NUMERIC / 
    (SELECT COUNT(*) FROM health.user_logs),
    2
  ) AS duplicate_percentage;
```