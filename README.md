# 8WeekSQLChallenge
Working through https://8weeksqlchallenge.com/

Starting 5/21/25 with the Danny's Diner Dataset, using the recommended DB Fiddle dashboard: https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138

Question #2: How many days has each customer visited the restaurant?

```sql
SELECT
  	s.customer_id,
    COUNT(DISTINCT s.order_date)
FROM dannys_diner.sales s
GROUP BY customer_id
ORDER BY s.customer_id ASC
LIMIT 5;
```
