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

#### Answer:

| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

Customer A visited 4 days, Customer B visited 6 days, Customer C visited 2 days.

***

** Question #3: What was the first item from the menu purchased by each customer? **

```sql
WITH ranked_orders AS (
  SELECT 
  	s.customer_id,
  	s.order_date,
  	s.product_id,
  	m.product_name,
  		ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m ON s.product_id = m.product_id
  )
  SELECT customer_id, order_date, product_name
  FROM ranked_orders
  WHERE ranked_orders.rn = 1
```

### Steps:
1. The ranked_order CTE provides a list of all orders from the sales table, then uses a LEFT JOIN to add the product name to the table and ROW_NUMBER and PARTITION BY to add ascending numbers (1, 2, 3...) to the order dates PARTITIONED BY each customer_id. The ROW_NUMBER column is labled rn.
2. Now, the first order for each customer is assigned a "1" in the rn column. The outer SELECT pulls the customer_id, order_date, and product_name contained in the ranked_orders CTE for any row that has "1" in the rn column (the first order for each customer_id).

### Answer:

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |
