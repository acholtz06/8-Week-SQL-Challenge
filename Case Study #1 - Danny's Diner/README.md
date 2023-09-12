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

### 6. Which item was purchased first by the customer after they became a member?

    WITH order_rank AS (
     	SELECT s.customer_id,
    			s.order_date,
            	s.product_id,
      			DENSE_RANK() OVER(
                  PARTITION BY s.customer_id
                  ORDER BY s.order_date) AS rank
    	FROM dannys_diner.sales AS s
    	JOIN dannys_diner.members AS m
    	ON s.customer_id = m.customer_id
    	WHERE s.order_date > m.join_date
    	ORDER BY s.customer_id)
        
    SELECT r.customer_id,
    	m.product_name AS first_purchase_after_member
    FROM order_rank AS r
    JOIN dannys_diner.menu as m
    ON r.product_id = m.product_id
    WHERE r.rank = 1
    ORDER BY customer_id;

| customer_id | first_purchase_after_member |
| ----------- | --------------------------- |
| A           | ramen                       |
| B           | sushi                       |

---
### 7. Which item was purchased just before the customer became a member?

    WITH last_order AS (
      SELECT s.customer_id,
      		s.order_date,
      		s.product_id,
      		DENSE_RANK() OVER(
              PARTITION BY s.customer_id
              ORDER BY s.order_date DESC) AS rank
      FROM dannys_diner.sales AS s
      JOIN dannys_diner.members AS m
      ON s.customer_id = m.customer_id
      WHERE s.order_date <= m.join_date
      ORDER BY s.customer_id)
    
    SELECT l.customer_id,
    		m.product_name AS last_purchase_before_member
    FROM last_order AS l
    JOIN dannys_diner.menu AS m
    ON l.product_id = m.product_id
    WHERE l.rank = 1
    ORDER BY l.customer_id;

| customer_id | last_purchase_before_member |
| ----------- | --------------------------- |
| A           | curry                       |
| B           | sushi                       |

---
### 8. What is the total items and amount spent for each member before they became a member?

    SELECT s.customer_id,
    		COUNT(s.*) AS total_items,
            SUM(u.price) AS total_spent
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.members AS m
    ON s.customer_id = m.customer_id
    JOIN dannys_diner.menu AS u
    ON s.product_id = u.product_id
    WHERE s.order_date <= m.join_date
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 3           | 40          |
| B           | 3           | 40          |

---
### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH points AS (
      SELECT s.customer_id,
      		CASE WHEN m.product_name = 'sushi'
      			THEN 200
      			ELSE m.price * 10 END AS points
      FROM dannys_diner.sales AS s
      JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id)
      
    SELECT customer_id,
      		SUM(points) AS total_points
    FROM points
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


    WITH points AS (
    SELECT s.customer_id,
    	s.product_id,
    	u.price,
    	CASE WHEN s.order_date BETWEEN m.join_date AND (m.join_date + 6) OR s.product_id = 1
        	THEN u.price * 10 * 2
      		WHEN s.order_date >= '2021-02-01'
      		THEN 0
            ELSE u.price * 10 END AS points
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS u
    ON s.product_id = u.product_id
    JOIN dannys_diner.members AS m
    ON s.customer_id = m.customer_id
    WHERE s.customer_id IN ('A', 'B'))
    
    SELECT customer_id,
    	SUM(points) AS points_with_multiplier
    FROM points
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | points_with_multiplier |
| ----------- | ---------------------- |
| A           | 1370                   |
| B           | 820                    |

---
**Query #9**

    SELECT s.customer_id,
    	s.order_date,
        u.product_name,
        u.price,
        CASE WHEN s.order_date >= m.join_date
        THEN 'Y'
        WHEN s.order_date < m.join_date
        THEN 'N'
        ELSE 'N' END AS member
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS u
    ON s.product_id = u.product_id
    FULL JOIN dannys_diner.members AS m
    ON s.customer_id = m.customer_id
    ORDER BY s.customer_id;

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---





