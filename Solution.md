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
- Use DISTINCT and wrap with COUNT to find out the number of visits for each customer
- We use DISTINCT for the order_date to avoid double counting  as a customer may order multiple items for each visit
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
- Create a temp table by using windows function and rank to create rank column based on order date
- Use WHERE to retrieve rows with RANK = 1 only and then GROUP BY to group it by customer
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
- COUNT the product_id and then arrange in descending order to find out the item that is most purchased by all customer
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
- Use COUNT function to count for number of orders for each item
- Use GROUP BY to sort the orders by customer and use ORDER by to arrange the order count in descending order
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
- Create a temp table and use DENSE RANK function to list the sales and arrange them in ascending order
- Use WHERE function to show only sales that are made on or after the join_date of the customer on the temp table
- Use WHERE function for the table to only show data with rank = 1 
#### Output
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

### 7. Which item was purchased just before the customer became a member?

````sql
WITH after_member_sales_cte AS
(
 SELECT sales.customer_id, members.join_date, sales.order_date, sales.product_id,
 DENSE_RANK() OVER(PARTITION BY sales.customer_id 
                   ORDER BY sales.order_date DESC) AS RANK
 FROM dannys_diner.sales
 JOIN dannys_diner.members 
 ON sales.customer_id = members.customer_id
 WHERE sales.order_Date < members.join_date
) 
SELECT customer_id, product_name, order_date
FROM after_member_sales_cte
JOIN dannys_diner.menu
ON after_member_sales_cte.product_id = menu.product_id
WHERE RANK = 1
ORDER BY customer_id;
````
- Create a temp table and rank the sales of each customer in descending order (i.e. most recent order ranks higher)
- SELECT only sales with oder_date ealier than the join_date
- SELECT all the sales order with rank =1
#### Output
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| B           | sushi        | 2021-01-04T00:00:00.000Z |

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT sales.customer_id, COUNT (order_date) AS total_item, SUM(price) AS total_spending
 FROM dannys_diner.sales
 JOIN dannys_diner.members 
 	ON sales.customer_id = members.customer_id
 JOIN dannys_diner.menu
 ON sales.product_id = menu.product_id
 
 WHERE sales.order_Date < members.join_date
 GROUP BY sales.customer_id
 ORDER BY customer_id;
````
- COUNT order_date to get the total items ordered, SUM price to get the total spending
- Use GROUP BY function to show sales by each customer
- Use WHERE function to filter out only sales that mare made before the customer become a member
#### Output
| customer_id | total_item | total_spending |
| ----------- | ---------- | -------------- |
| A           | 2          | 25             |
| B           | 3          | 40             |

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
- Create a temp table and use Case function to create 2 scenarios: 
1) when the product_id = 1 (i.e. sushi)  points = price * 20
2) when the product_id != 1 (i.e. other items)  points = price*10
- join the temp table and the sales table and SUM the points column and GROUP them by customer to find the total points for each customer

#### Output
| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

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
- Create a temp table to record the range of dates where client would be eligible for 2x points
- Use case function to create 3 cases for points calculation
1) For product_id = 1 (i.e. ramen) points = price *20
2) For order_date is between join_date and member_date (last day of promotion) points = price * 20 
3) For all other orders points = price*10
- Select all the required information and GROUP and ORDER them by customer_id
#### Output
| customer_id | order_date               | product_name | points |
| ----------- | ------------------------ | ------------ | ------ |
| A           | 2021-01-01T00:00:00.000Z | curry        | 150    |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 200    |
| A           | 2021-01-07T00:00:00.000Z | curry        | 300    |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 240    |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 480    |
| B           | 2021-01-01T00:00:00.000Z | curry        | 150    |
| B           | 2021-01-02T00:00:00.000Z | curry        | 150    |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 200    |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 200    |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 120    |

