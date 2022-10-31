# Danny's Diner - Solution

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT customer_id AS customer, SUM(menu.price) AS spending
FROM dannys_diner.sales 
	INNER JOIN dannys_diner.menu
    	ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY customer_id ORDER BY customer_id ASC;
````

### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS visits
FROM dannys_diner.sales
GROUP BY customer_id ORDER BY COUNT(DISTINCT(order_date)) DESC
````


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

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT product_name, COUNT(dannys_diner.sales.product_id) AS orders 
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY product_name
ORDER BY orders DESC;
````

### 5. Which item was the most popular for each customer?

````sql
SELECT product_name, customer_id, COUNT(dannys_diner.sales.product_id) AS orders 
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY product_name, customer_id
ORDER BY customer_id ASC, orders DESC;
````

### 6. Which item was purchased first by the customer after they became a member?

````sql

````

### 7. Which item was purchased just before the customer became a member?

````sql

````

### 8. What is the total items and amount spent for each member before they became a member?

````sql

````

### 9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

````sql

````

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

````sql

````
