# 🍣 Case Study #1 - Danny's Diner



![Danny's Diner](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/4ef18033-2b1c-4efe-8591-39adfe66dcb1)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-1/)

## ✏️ Business Task
Danny wants to take a closer look at the habits of his customers, such as spending patterns, popular menu items, and number of visits. The diner has a customer rewards program that already exists, and he wants to see if there are ways that the program can be modified to improve the overall customer experience. The case study has provided a list of questions that can help to aid in making desicions about the menu and the rewards program.

## ⚠️ Data Limitations
The biggest limitation I noticed with the data is the lack of timestamps. When looking at the first item purchased by a customer, I can't give a definitive answer when there are multiple purchases on the same day. The other limitation with the timestamps is determining exactly when a customer signed up for the rewards program. If a customer made a purchase and also joined the rewards program on the same day, I have no way of knowing if they joined before or after making the purchase. Because the double points period of the rewards system includes the join date, I considered any purchases made on the join date to be counted as a member purchase. This method was used for every question to keep everything consistent.

## 🍛 Data
There were three datasets provided for this case study:
- sales
- menu
- members

![Table Relationships - Danny's Diner](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/2fcfec00-f2cd-4896-a9b3-4a7773d79399)

## 🍜 Case Study Questions
---
#### 1. What is the total amount each customer spent at the restaurant?

    SELECT s.customer_id,
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


Steps:
- Joined the sales table with the menu table to find the price of each sale
- Took the sum of all of the sales
- Grouped by the customer id to see the total amount spent per customer

Insights:
- Customers A and B spent over twice as much as customer C

---

#### 2. How many days has each customer visited the restaurant?

    SELECT customer_id,
            COUNT(DISTINCT order_date)
      FROM dannys_diner.sales
      GROUP BY customer_id
      ORDER BY customer_id;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

Steps:
- Counted the distinct order dates in case a customer visited more than once in one day
- Grouped by customer id

Insights:
- Customer B visited the diner the most days
- Customer C visited the diner the least days

---
#### 3. What was the first item from the menu purchased by each customer?

    SELECT customer_id,
            MIN(order_date)
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | min                      |
| ----------- | ------------------------ |
| A           | 2021-01-01T00:00:00.000Z |
| B           | 2021-01-01T00:00:00.000Z |
| C           | 2021-01-01T00:00:00.000Z |


    SELECT s.customer_id,
    	m.product_name AS first_purchase
    FROM dannys_diner.sales AS s
    JOIN dannys_diner.menu AS m
    ON s.product_id = m.product_id
    WHERE s.order_date = '2021-01-01'
    ORDER BY s.customer_id;

| customer_id | first_purchase |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

Steps:
- Found the minimum, or first, order date
- Grouped by customer
- Joined the sales and menu tables to get the product names for the sales
- Filter for purchases made on 2021-01-01 because all customers made their first purchase on the same day

Insights:
- Each customer made their first purchase on the same day
- Each menu item was purchased at least once
- Becuase there are no time stamps, it is impossible to tell exactly which item was the first purchase, so all items purchased on the first day are included


---
#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

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

Steps:
- Created a CTE called product_count
- Counted each product id in the sales table to find out how many times a sale was made
- Grouped the results by product id
- Joined the CTE to the menu table to get the product names
- Ordered the results in descending order so that the most popular item was first
- Limited the results to 1

Insights:
- Ramen is the most popular menu item
---
#### 5. Which item was the most popular for each customer?

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

Steps:
- Created a CTE called purchased
- Counted the products to find the number purchased
- Used DENSE_RANK() to rank the total number of items purchased, partitioned by the customer id and ordered by the count of each item purchased
- Grouped by the customer id and product id to find out how many of each item was purchased by each customer
- Joined the CTE with the menu table to get the product names
- Filtered to only show the products that had a rank of 1

Insights:
- Ramen is the most popular item with customers A and C
- Customer B likes all of the menu items equally

---

#### 6. Which item was purchased first by the customer after they became a member?

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
#### 7. Which item was purchased just before the customer became a member?

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
#### 8. What is the total items and amount spent for each member before they became a member?

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
#### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

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
#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


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





