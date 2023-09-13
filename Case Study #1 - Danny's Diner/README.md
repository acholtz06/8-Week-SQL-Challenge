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
        WHERE s.order_date >= m.join_date
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
| A           | curry                       |
| B           | sushi                       |

❗**Assuming the orders made on the join date count as member purchase**❗

Steps:
- Created a CTE called order_rank
- Joined the sales and the member tables to get the member join dates
- Ranked the order dates by customer id
- Filtered for order dates that were greater than or equal to the membership join date
- Joined the CTE to the menu table to get the product names
- Filtered for prodcuts that had a rank of 1

Insights:
- With only having two members to analyze, there are no trends
- Neither of the first items purchased after becoming a member was the diner's most popular item (ramen)

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
        WHERE s.order_date < m.join_date
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
| A           | sushi                       |
| A           | curry                       |
| B           | sushi                       |

❗**Assuming the orders made on the join date count as member purchase**❗

Steps:
- Created a CTE called last_order
- Joined the sales and the member tables to get the join dates
- Ranked the order dates by customer id in descending order so that the most recent date is first
- Filtered for order dates that were before the join date
- Joined the CTE to the menu table to get the product names
- Filtered for order dates that had a rank of 1

Insights:
- Customer A made two purchses on the day the last order before becoming a member was placed
- Both customers ordered sushi before becoming a member

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
    WHERE s.order_date < m.join_date
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

❗**Assuming the orders made on the join date count as member purchase**❗

Steps:
- Joined all three tables to get the join dates and the product prices
- Counted the total number of orders
- Took the sum of the prices
- Filtered for dates that we before the join date
- Grouped by the customer id

Insights:
- With only two customers to analyze, there are no real trends

---
#### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH points AS (
    	SELECT s.customer_id,
      		s.order_date,
        	CASE WHEN m.product_name = 'sushi'
        	THEN 200
        	ELSE m.price * 10 END AS points
        FROM dannys_diner.sales AS s
        JOIN dannys_diner.menu AS m
        ON s.product_id = m.product_id)
          
    SELECT p.customer_id,
    	SUM(p.points) AS total_points
    FROM points AS p
    JOIN dannys_diner.members AS m
    ON p.customer_id = m.customer_id
    WHERE p.order_date >= m.join_date
    GROUP BY p.customer_id
    ORDER BY p.customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 510          |
| B           | 440          |

❗**Assuming the orders made on the join date count as member prices**❗

Steps:
- Created a CTE called points to establish the points system
- Joined the sales tabke to the menu table to get the product prices
- Used a CASE statement to establish that sushi has a 2x multiplier and that all other items have the regular $1 equates to 10 points system
- Inner joined the CTE to the members table to make sure only customers who were members were counted
- Found the sum of the points
- Filtered for order dates that were greater than or equal to the join date
- Grouped by the customer id

Insights:
- There isn't a clear relationship between the two members, but this result can be used to get an idea of how many points can be typically accrued in a one month period of time

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
      		WHEN s.order_date < m.join_date
      		THEN 0
            ELSE u.price * 10 END AS points
        FROM dannys_diner.sales AS s
        JOIN dannys_diner.menu AS u
        ON s.product_id = u.product_id
        JOIN dannys_diner.members AS m
        ON s.customer_id = m.customer_id
    )
        
    SELECT customer_id,
    	SUM(points) AS points_with_multiplier
    FROM points
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | points_with_multiplier |
| ----------- | ---------------------- |
| A           | 1220                   |
| B           | 520                    |

❗**Assuming the orders made on the join date count as member prices**❗

Steps:
- Created a CTE called points to establish the points system
- Joined all three tables to have the member join date and the product prices
- Used a CASE statement so that if the purchase was made in the first week OR if the product was sushi, the points were doubled. This CASE statement also says that any orders placed before the join date or after the month of January do not count towards the points. Last, this statement says that any other purchases that weren't already listed have the normal 10 points per $1 spent system
- Used the points table to find the sum of points per customer with all of the multipliers included for the correct dates

Insights:
- The multiplier didn't change the points customer B had, but customer A was able to more than double their points once the multiplier was added

---

## 💡 Bonus Questions

In these bonus questions, Danny provided two tables using the data that can be used to quickly reference for insights. The task is to recreate these tables.

#### Join All The Things

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


This table gives a quick view of each order and whether the customer was a rewards member or not at the time of purchase

Steps:
- Joined the sales and the menu tables to get the product names and prices
- Full joined the member table to make sure that all customers, not just members, were included
- Used a CASE statement to establish that orders places after the join date count as 'Y' in the member column. This statement also reflects that all other orders should appear as 'N' in the member column
---

#### Rank All The Things

    WITH join_things AS (
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
    	ORDER BY s.customer_id
      )
      
    SELECT *,
    	CASE WHEN member = 'Y'
        THEN DENSE_RANK() OVER(
          		PARTITION BY customer_id, member
          		ORDER BY order_date)
        ELSE null END AS ranking
    FROM join_things
    ORDER BY customer_id;

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |         |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |         |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |         |

Steps:
- Created a CTE from the first bonus question query
- Used a CASE statment to rank the order dates for only member orders. This was done by saying that if the member staus was 'Y', then DENSE_RANK() the order dates, partitioning by customer id and also by member status so that the non-member orders aren't ranked. All other orders are set to null
---

## 🚀Final Thoughts

Thinking back to the busniess task, we wanted to analyze this data to see how the rewards program was doing and also to see if there were any potential changes that could be made to improve the customer experience. There were some insights that were gained while doing the analysis that I think could be helpful to Danny as he evaluates the membership program.

- Customers that visited more often/spent more money at the diner were more likely to join the membership program
- Ramen is the most popular menu item
- How many points do customers earn in a month with or without including the 2x multiplier
- Both members ordered sushi before deciding to sign up

These insights could lead to decisions like making ramen the menu item that earns double points, or deciding what points can be used on based on the number of points customers are earning over a period of time. I think that there could have been many more conclusions if there had been more customer data. There were several questions where it was too difficult to draw a conclusion because it is hard to spot a trend when there are only two customers.

This was my first full case study using SQL and I really enjoyed it! I know there are several different ways to write the queries to get these answers, and I look forward to looking through some other solutions to see some different points of view.

Thank you for being here!



