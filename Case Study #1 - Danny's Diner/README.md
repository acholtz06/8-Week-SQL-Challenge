![Danny's Diner](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/4ef18033-2b1c-4efe-8591-39adfe66dcb1)


### Sales Table

    SELECT *
    FROM dannys_diner.sales;

| customer_id | order_date               | product_id |
| ----------- | ------------------------ | ---------- |
| A           | 2021-01-01T00:00:00.000Z | 1          |
| A           | 2021-01-01T00:00:00.000Z | 2          |
| A           | 2021-01-07T00:00:00.000Z | 2          |
| A           | 2021-01-10T00:00:00.000Z | 3          |
| A           | 2021-01-11T00:00:00.000Z | 3          |
| A           | 2021-01-11T00:00:00.000Z | 3          |
| B           | 2021-01-01T00:00:00.000Z | 2          |
| B           | 2021-01-02T00:00:00.000Z | 2          |
| B           | 2021-01-04T00:00:00.000Z | 1          |
| B           | 2021-01-11T00:00:00.000Z | 1          |
| B           | 2021-01-16T00:00:00.000Z | 3          |
| B           | 2021-02-01T00:00:00.000Z | 3          |
| C           | 2021-01-01T00:00:00.000Z | 3          |
| C           | 2021-01-01T00:00:00.000Z | 3          |
| C           | 2021-01-07T00:00:00.000Z | 3          |

---
### Menu Table

    SELECT *
    FROM dannys_diner.menu;

| product_id | product_name | price |
| ---------- | ------------ | ----- |
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

---
### Members Table

    SELECT *
    FROM dannys_diner.members;

| customer_id | join_date                |
| ----------- | ------------------------ |
| A           | 2021-01-07T00:00:00.000Z |
| B           | 2021-01-09T00:00:00.000Z |

---
### 1. What is the total amount each customer spent at the restaurant?

    SELECT
    	s.customer_id,
        SUM(m.price) AS total_spent
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS m
    ON s.product_id = m.product_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
### 2. How many days has each customer visited the restaurant?

    SELECT
    	customer_id,
        COUNT(DISTINCT order_date)
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
### 3. What was the first item from the menu purchased by each customer?

    SELECT
        customer_id,
        MIN(order_date)
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | min                      |
| ----------- | ------------------------ |
| A           | 2021-01-01T00:00:00.000Z |
| B           | 2021-01-01T00:00:00.000Z |
| C           | 2021-01-01T00:00:00.000Z |


    SELECT
    	s.customer_id,
        m.product_name
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS m
    ON s.product_id = m.product_id
    WHERE s.order_date = '2021-01-01'
    ORDER BY s.customer_id;

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

---
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

    WITH product_count AS (
      SELECT product_id,
      		COUNT(product_id) AS total_purchased
      FROM dannys_diner.sales
      GROUP BY product_id
      )
    
    SELECT m.product_name,
    		p.total_purchased
    FROM product_count AS p
    JOIN dannys_diner.menu AS m
    ON p.product_id = m.product_id
    ORDER BY p.total_purchased DESC
    LIMIT 1;

| product_name | total_purchased |
| ------------ | --------------- |
| ramen        | 8               |

---
### 5. Which item was the most popular for each customer?

    WITH purchased AS(
    	SELECT customer_id,
      			product_id,
    			COUNT(product_id) AS n_purchased,
      			 DENSE_RANK() OVER(
                  PARTITION BY customer_id
                  ORDER BY COUNT(product_id) DESC) AS product_rank
    	FROM dannys_diner.sales
    	GROUP BY customer_id,
    			product_id
    	ORDER BY customer_id)
    
    SELECT p.customer_id,
    	m.product_name
    FROM purchased AS p
    JOIN dannys_diner.menu AS m
    ON p.product_id = m.product_id
    WHERE p.product_rank = 1
    ORDER BY customer_id;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |
| B           | curry        |
| B           | ramen        |
| C           | ramen        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/5358)
