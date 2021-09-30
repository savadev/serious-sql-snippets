## Exploratory analysis

### 1. Count, distinct and frequencies.
Distinct values from a column and **frequencies** with two decimals:

```sql
SELECT
  rating,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY 2 DESC;
```

NUMERIC is used to avoid dreaded integer floor division.
When we divide a INT data type with another INT data type, SQL engine automatically returns you the floor division. 

```sql
SELECT
  15 / 20 AS integer_division;
```
It returns 0 instead of 0.75.

BE CAREFUL WITH THIS BEHAVIOUR.

So we use a cast to an FLOAT type as NUMERIC
```sql
SELECT
  15::NUMERIC / 20 AS integer_division;
```


### 2. Frequencies with multiple columns.

```sql
SELECT
  rating,
  category,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS frequency
FROM dvd_rentals.film_list
GROUP BY
  rating,
  category
ORDER BY frequency DESC
LIMIT 5;
```