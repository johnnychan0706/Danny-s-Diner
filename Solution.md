# Danny's Diner - Solution

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT customer_id AS customer, SUM(menu.price) AS spending
FROM dannys_diner.sales 
	INNER JOIN dannys_diner.menu
    	ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY customer_id ORDER BY customer_id ASC;
````
- Use SUM and GROUP BY to find the total sales of each customer
- Use JOIN to merge the sales and menu table as price of product is in the menu table

#### Output
| customer | spending |
| -------- | -------- |
| A        | 76       |
| B        | 74       |
| C        | 36       |



### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS visits
FROM dannys_diner.sales
GROUP BY customer_id ORDER BY COUNT(DISTINCT(order_date)) DESC;
````
#### Output
| customer_id | visits |
| ----------- | ------ |
| B           | 6      |
| A           | 4      |
| C           | 2      |


### 3. What was the first item from the menu purchased by each customer?

````sql
WITH sales_time_cte AS
(
  SELECT customer_id, order_date, product_name,
  RANK() OVER(PARTITION BY dannys_diner.sales.customer_id
  ORDER BY dannys_diner.sales.order_date) AS rank
 FROM dannys_diner.sales
 JOIN dannys_diner.menu 
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
  )
SELECT customer_id, product_name
FROM sales_time_cte
WHERE rank = 1
GROUP BY customer_id, product_name;
````
#### Output
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT product_name, COUNT(dannys_diner.sales.product_id) AS orders 
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY product_name
ORDER BY orders DESC;
````
#### Output
| product_name | orders |
| ------------ | ------ |
| ramen        | 8      |
| curry        | 4      |
| sushi        | 3      |

### 5. Which item was the most popular for each customer?

````sql
SELECT product_name, customer_id, COUNT(dannys_diner.sales.product_id) AS orders 
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY product_name, customer_id
ORDER BY customer_id ASC, orders DESC;
````
#### Output
| product_name | customer_id | orders |
| ------------ | ----------- | ------ |
| ramen        | A           | 3      |
| curry        | A           | 2      |
| sushi        | A           | 1      |
| curry        | B           | 2      |
| sushi        | B           | 2      |
| ramen        | B           | 2      |
| ramen        | C           | 3      |

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH after_member_sales_cte AS
(
 SELECT sales.customer_id, members.join_date, sales.order_date, sales.product_id,
 DENSE_RANK() OVER(PARTITION BY sales.customer_id 
                   ORDER BY sales.order_date) AS RANK
 FROM dannys_diner.sales
 JOIN dannys_diner.members 
 ON sales.customer_id = members.customer_id
 WHERE sales.order_Date >= members.join_date
) 
SELECT customer_id, product_name
FROM after_member_sales_cte
JOIN dannys_diner.menu
ON after_member_sales_cte.product_id = menu.product_id
WHERE RANK = 1
ORDER BY customer_id;
````

### 7. Which item was purchased just before the customer became a member?

````sql

````

### 8. What is the total items and amount spent for each member before they became a member?

````sql

````

### 9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

````sql
WITH points_cte AS 
(
  SELECT *,
  CASE 
  	WHEN product_id = 1 THEN price*20
  	ELSE price*10
  END AS points
  FROM dannys_diner.menu
  )
  SELECT customer_id, SUM(points)
  FROM dannys_diner.sales
  JOIN points_cte
  ON dannys_diner.sales.product_id = points_cte.product_id
  GROUP BY customer_id
  ORDER BY customer_id ASC;
````

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

````sql
WITH membership_cte AS
(
  SELECT *, join_date + integer'6' AS member_date,
  		TO_DATE('2021-01-31','YYYY-MM-DD') AS last_date
  FROM dannys_diner.members
  )
SELECT sales.customer_id, sales.order_date, product_name,
SUM ( CASE
     WHEN sales.product_id = 1 THEN price*2*10
     WHEN (order_date >= join_date) AND (order_date <= member_date) THEN price*2*10
     ELSE price*10 
    END) AS points
FROM membership_cte
JOIN dannys_diner.sales
	ON membership_cte.customer_id = sales.customer_id
JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
WHERE sales.order_date <= membership_cte.last_date
GROUP BY sales.customer_id, sales.order_date, product_name
ORDER BY sales.customer_id;
````
