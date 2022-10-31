# Danny's Diner - Solution

### 1. What is the total amount each customer spent at the restaurant?
''''sql
SELECT customer_id AS customer, SUM(menu.price) AS spending
FROM dannys_diner.sales 
	INNER JOIN dannys_diner.menu
    	ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY customer_id ORDER BY customer_id ASC
''''
