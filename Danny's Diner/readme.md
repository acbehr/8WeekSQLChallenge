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

***

**Question #4:  What is the most purchased item on the menu and how many times was it purchased by all customers?**

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

**Question #5: Which item was the most popular for each customer?**
```sql
WITH favorite AS (
	SELECT 
		s.customer_id,
		m.product_name,
		COUNT(m.product_id) as order_count,
		DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) as rank
	FROM dannys_diner.sales s
	LEFT JOIN dannys_diner.menu m
	ON s.product_id = m.product_id
	GROUP BY s.customer_id, m.product_name
	ORDER BY customer_id
	)

SELECT
	customer_id, 
	product_name,
	order_count   
FROM favorite
WHERE rank = 1
```
### Steps:
1. The "favorite" CTE selects the customer_id, product_name, and the sum of each product_id from the sales table as 'order count'.
2. The number of orders are given ranks using DENSE_RANK() to each of the customer_ids (A, B, C) based on the COUNT of the product_ids associated with their respective orders. The product_id with the greatest sum for each customer_id is assgined rank = 1 in this column.
3. The second query selects the relevant data (customer_id, product_name, and the number of orders as order_count) from the "favorite" CTE, returning the top value for each customer_id by only returing values WHERE rank = 1.
4. Because customer_id B ordered each menu item twice, there is no top rank, so all equally ranked orders are shown 

### Answer:

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |
***
**Question #6: Which item was purchased first by the customer after they became a member?**

```sql
WITH member_orders AS (
SELECT
	s.customer_id,
    s.order_date,
    m.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date asc) AS rank
FROM sales s
JOIN menu m
	ON m.product_id = s.product_id
RIGHT JOIN members me
	ON me.customer_id = s.customer_id
WHERE s.order_date > me.join_date
GROUP BY s.customer_id, s.order_date, m.product_name
	)

SELECT
	customer_id,
    order_date,
    product_name
FROM member_orders
WHERE rank = 1
```
### Steps:
1. The CTE member_orders selects FROM sales (s) the s.customer_id, s.order_date, m.product_name, joining the menu table (m) ON m.product_name using product_id.
2. JOIN members table (me) ON customer_id. The WHERE clause filters orders after members joined using s.order_date values that are greater than me.join_date for each s.customer_id.
3. DENSE_RANK() adds ranks to each s.order_date in ascending order PARTITION(ED) BY s.customer_id, making the first s.order_date > me.join_date rank 1.
4. The statement SELECT(S) customer_id, order_date, and product_name from the member_orders CTE. The WHERE clause filters each customer_id where rank = 1, the first order_date > the member join date.

### Answer:

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-10 | ramen        |
| B           | 2021-01-11 | sushi        |

***
**Question 7: Which item was purchased just before the customer became a member?**

```sql
WITH pre_member_orders AS (
SELECT
	s.customer_id,
    s.order_date,
    m.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date desc) AS rank
FROM sales s
JOIN menu m
	ON m.product_id = s.product_id
RIGHT JOIN members me
	ON me.customer_id = s.customer_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id, s.order_date, m.product_name
	)

SELECT
	customer_id,
    order_date,
    product_name
FROM pre_member_orders
WHERE rank = 1
```

### Steps:
1. The pre_member_orders CTE contains the sales (s) s.customer_id, s.order_date, and menu (m) m.porduct_name JOIN(ED) on product_id.
2. The WHERE clause in pre_member_orders selects only the order dates less than members (me) me.join_date.
3. The DENSE_RANK assigns ranks (1, 2, 3...) to each s.order_date PARTITION(ED) BY s.customer_id in descending order, so the highest date (before the customers became members) is assigned rank 1.
4. The statement SELECT(S) the customer_id, order_date, and product_name from the pre_member_orders CTE, WHERE rank = 1. Customer A ordered both sushi and curry on their first visit and became a member on their second visit, so both are included because the true order is unknown. Customer B got sushi on their third visit, and became a member between the third and forth visit.

### Answer:
| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |
