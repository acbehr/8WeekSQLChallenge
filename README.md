# 8WeekSQLChallenge
Working through https://8weeksqlchallenge.com/

Starting 5/21/25 with the Danny's Diner Dataset, using the recommended DB Fiddle dashboard: https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138

***

**Question #2: How many days has each customer visited the restaurant?**

```sql
SELECT
  	s.customer_id,
    COUNT(DISTINCT s.order_date)
FROM dannys_diner.sales s
GROUP BY customer_id
ORDER BY s.customer_id ASC
LIMIT 5;
```
#### Steps:
1. Select customer_id from the sales table
2. GROUP BY customer_id to organize aggregate (COUNT) function
3. COUNT the DISTINCT instances of order dates, to count only the number of days each customer visited Danny's Diner (not the total number of visits, because some customers visited more than once in a single day. Customer A visited twice on 2021-01-01, so DISTINCT is needed to return only one day they visited, instead of 2 times they visited that day)
