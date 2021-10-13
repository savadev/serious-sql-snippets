#### 1.Dates
POSTGRE

DATE_TRUNC('quarter', date)

TO_CHAR(date, 'FMDay DD, FMMonth YYYY') --> Friday 13, August 2018
* Dy: Abbreviated day name (Fri)
* DD: Day number (12)
* FMDay: Full day name (Monday)
* MM: Month of year (03)
* Mon: Abbreviated month (Jan)
* FMMonth: Full month (January)
* YY: Last two digits of year (18)
* YYYY: Full 4 digit year(2018)

#### 2.Window functions
SUM(...) OVER (...)

```sql
-- Set up the user_count_orders CTE
WITH user_count_orders AS (
  SELECT
    user_id,
    COUNT(DISTINCT order_id) AS count_orders
  FROM orders
  -- Only keep orders in August 2018
  WHERE DATE_TRUNC('month', order_date) = '2018-08-01'
  GROUP BY user_id)

SELECT
  -- Select user ID, and rank user ID by count_orders
  user_id,
  RANK() OVER(ORDER BY count_orders DESC) AS count_orders_rank
FROM user_count_orders
ORDER BY count_orders_rank ASC
-- Limit the user IDs selected to 3
LIMIT 3;
```

### 3.Pivoting (row into column)
