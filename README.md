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

**Question #3: What was the first item from the menu purchased by each customer?**

```sql
WITH ranked_orders AS (
  SELECT 
  	s.customer_id,
  	s.order_date,
  	s.product_id,
  	m.product_name,
  		DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m ON s.product_id = m.product_id
  )
  
  SELECT customer_id, product_name
  FROM ranked_orders
  WHERE ranked_orders.rn = 1
  GROUP BY customer_id, product_name
```

### Steps:
1. The ranked_order CTE provides a list of all orders from the sales table, then uses a LEFT JOIN to add the product name to the table and DENSE_RANK and PARTITION BY to add ascending numbers (1, 2, 3...) to the order dates PARTITIONED BY each customer_id. The DENSE_RANK column is labled rn. DENSE_RANK (assigns the same rank to duplicate/tie values) is used instead of ROW_NUMBER because customer A ordered two items on their first visit; with no way to know which order actually came first, they are both assigned a "1" value and are both counted within the first order.
2. Now, the first order for each customer is assigned a "1" in the rn column. The outer SELECT pulls the customer_id, order_date, and product_name contained in the ranked_orders CTE for any row that has "1" in the rn column (the first order for each customer_id).
3. The results are GROUP(ED) BY customer_id and product_name because customer C had two orders of ramen in their first order, so those are grouped together as a single product.

### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

**Question 4:  What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT 
	product_name, 
    COUNT(s.product_id) as orders
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY orders DESC
LIMIT 1
```

### Steps:
1. The product_name is LEFT JOIN(ED) from the menu table to the sales table using product_id, and the COUNT of product_id(s) labled as orders are pulled from the sales table.
2. Results are GROUP(ED) BY product name, and the count for the top product is returned by limiting ORDER BY output to only the first result in a DESCending list.

### Answer:

| product_name | orders |
| ------------ | ------ |
| ramen        | 8      |

***
