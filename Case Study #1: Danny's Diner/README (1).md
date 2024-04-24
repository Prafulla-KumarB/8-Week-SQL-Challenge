
🍜 Case Study #1: Danny's Diner

![App Screenshot](https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png)


## 📚Table of Contents

1. Business Task  
2. Entity Relationship Diagram  
3. Question and Solution

## Business Task

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

## Entity Relationship Diagram

![ERD](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Question and Solution

### 1. What is the total amount each customer spent at the restaurant?
``` sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC; 
```

| customer id| total_sales | 
| ---------- | ------------| 
| A          | 76          | 
| B          | 74          | 
| C          | 36          | 

---

### 2. How many days has each customer visited the restaurant?
``` sql
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id; 
```

| customer id| Visit_count | 
| ---------- | ------------| 
| A          | 4           | 
| B          | 6           | 
| C          | 2           | 

### 3. What was the first item from the menu purchased by each customer?
``` sql
WITH ordered_sales AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
```
| customer id| Product_name| 
| ---------- | ------------| 
| A          | curry       |
| A	     | sushi	   |
| B          | curry       | 
| C          | ramen       | 

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql
Select TOP 1
	m.product_name,
	COUNT(m.product_name) as Times
from dannys_diner.sales s
INNER JOIN dannys_diner.menu m
	ON m.product_id = s.product_id
GROUP BY m.product_name
ORDER BY Times desc;
```
| Product_name| Times	    | 
| ----------  | ------------| 
| ramen       | 8           |

### 5. Which item was the most popular for each customer?
``` sql
WITH Ranked_orderedNTimes as
(
Select 
	s.customer_id,
	m.product_name,
	COUNT(m.product_name)as Ordered_Ntimes,
	DENSE_RANK () over (Partition by customer_id order by COUNT(m.product_name)desc) RANK
FROM 
	dannys_diner.sales s
INNER JOIN dannys_diner.menu m
	ON m.product_id = s.product_id
GROUP BY 
	s.customer_id, m.product_name
)
Select 
	customer_id,
	product_name,
	Ordered_NTimes
from 
	Ranked_orderedNTimes
WHERE 
	RANK = 1;
```
| customer_id | product_name| Ordered_NTimes |
| ----------  | ------------| -------------- |
| A	      |ramen        |3               |
| B	      |sushi        |2               |
| B	      |curry        |2               |
| B           |ramen        |2               |
| C           |ramen        |3               |


### 6. What is the number and percentage of customer plans after their initial free trial?
``` sql
   WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```
| customer id| product_name| 
| ---------- | ------------| 
| A          | ramen       | 
| B          | sushi       | 

### 7. Which item was purchased just before the customer became a member?
``` sql
WITH purchased_prior_member AS (
  SELECT 
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date < members.join_date
)

SELECT 
  p_member.customer_id, 
  menu.product_name 
FROM purchased_prior_member AS p_member
INNER JOIN dannys_diner.menu
  ON p_member.product_id = menu.product_id
WHERE rank = 1
ORDER BY p_member.customer_id ASC;
```

| customer id| product_name| 
| ---------- | ------------| 
| A          | sushi       | 
| B          | sushi       |

### 8. What is the total items and amount spent for each member before they became a member?
``` sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;

| customer_id | total_items	| total_sales |
| ----------  | --------------- | ----------- |
| A	      |2                |25           |
| B	      |3                |40           |

``` sql

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

WITH points_cte AS (
  SELECT 
    menu.product_id, 
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10 END AS points
  FROM dannys_diner.menu
)

SELECT 
  sales.customer_id, 
  SUM(points_cte.points) AS total_points
FROM dannys_diner.sales
INNER JOIN points_cte
  ON sales.product_id = points_cte.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
| customer id| total_points| 
| ---------- | ------------| 
| A          | 860         | 
| B          | 940         |
| C          | 360         |


