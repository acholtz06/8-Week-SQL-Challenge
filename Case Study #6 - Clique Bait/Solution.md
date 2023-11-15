# 🎣 Case Study #6 - Clique Bait

![Clique Bait PIcture](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/60554132-d756-4b27-ac8b-28f4fad59faa)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-6/)

## 📚 Table of Contents
- [Business Task](#-business-task)
- [Data Limitations](#%EF%B8%8F-data-limitations)
- [Data](#-data)
- [Case Study Questions](#-case-study-questions)
    - [A. Digital Analysis](#-a-digital-analysis)
    - [B. Product Funnel Analysis](#-b-product-funnel-analysis)
    - [C. Campaigns Analysis](#-c-campaigns-analysis)
- [Final Thoughts](#-final-thoughts)
  
## 🦀 Business Task
## ⚠️ Data Limitations
## 🦪 Data
## 🐟 Case Study Questions
### 🦞 A. Digital Analysis
---

#### 1. How many users are there?

    SELECT COUNT(DISTINCT user_id) AS num_users
    FROM clique_bait.users;

| num_users |
| --------- |
| 500       |

---
#### 2. How many cookies does each user have on average? FIX THIS

    WITH count AS (
      SELECT user_id,
    	COUNT(cookie_id) AS num_cookie_id
      FROM clique_bait.users
      GROUP BY user_id
      ORDER BY user_id)
    
    SELECT ROUND(AVG(num_cookie_id), 2) AS avg_num_cookies
    FROM count;

| avg_num_cookies |
| --------------- |
| 3.56            |

---
#### 3. What is the unique number of visits by all users per month?

    SELECT DATE_TRUNC('month', event_time) AS month,
    	COUNT(DISTINCT visit_id) AS num_visits
    FROM clique_bait.events
    GROUP BY month
    ORDER BY month;

| month                    | num_visits |
| ------------------------ | ---------- |
| 2020-01-01T00:00:00.000Z | 876        |
| 2020-02-01T00:00:00.000Z | 1488       |
| 2020-03-01T00:00:00.000Z | 916        |
| 2020-04-01T00:00:00.000Z | 248        |
| 2020-05-01T00:00:00.000Z | 36         |

---
#### 4. What is the number of events for each event type?

    SELECT i.event_name,
    	COUNT(e.event_type)
    FROM clique_bait.events AS e
    JOIN clique_bait.event_identifier AS i
    ON e.event_type = i.event_type
    GROUP BY i.event_name,
    	e.event_type
    ORDER BY e.event_type;

| event_name    | count |
| ------------- | ----- |
| Page View     | 20928 |
| Add to Cart   | 8451  |
| Purchase      | 1777  |
| Ad Impression | 876   |
| Ad Click      | 702   |

---
#### 5. What is the percentage of visits which have a purchase event?

    SELECT ROUND((COUNT(DISTINCT visit_id)::numeric / (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)::numeric) * 100, 2) AS percent_purchase
    FROM clique_bait.events
    WHERE event_type = 3;

| percent_purchase |
| ---------------- |
| 49.86            |

---
#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event? FIX THIS

    WITH filter AS (
      SELECT visit_id,
    	event_type
      FROM clique_bait.events
      WHERE event_type IN (2, 3))
    
    , num AS (
      SELECT visit_id,
      event_type,
      ROW_NUMBER() OVER(
        PARTITION BY visit_id
        ORDER BY event_type DESC)
      FROM filter)
    
    SELECT ROUND((COUNT(DISTINCT visit_id)::numeric / (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)::numeric) * 100, 2) AS percent_no_purchase
    FROM num 
    WHERE row_number = 1 AND event_type <> 3;

| percent_no_purchase |
| ------------------- |
| 20.57               |

---
#### 7. What are the top 3 pages by number of views?

    SELECT h.page_name,
    	COUNT(e.page_id) AS num_views
    FROM clique_bait.events AS e
    JOIN clique_bait.page_hierarchy AS h
    ON e.page_id = h.page_id
    GROUP BY h.page_name
    ORDER BY num_views DESC
    LIMIT 3;

| page_name    | num_views |
| ------------ | --------- |
| All Products | 4752      |
| Lobster      | 2515      |
| Crab         | 2513      |

---
#### 8. What is the number of views and cart adds for each product category?

    SELECT h.product_category,
    	COUNT(e.visit_id) AS num_views,
        SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS num_cart_adds
    FROM clique_bait.events AS e
    JOIN clique_bait.page_hierarchy AS h
    ON e.page_id = h.page_id
    WHERE h.product_category IS NOT NULL
    GROUP BY h.product_category;

| product_category | num_views | num_cart_adds |
| ---------------- | --------- | ------------- |
| Luxury           | 4902      | 1870          |
| Shellfish        | 9996      | 3792          |
| Fish             | 7422      | 2789          |

---
#### 9. What are the top 3 products by purchases?

    WITH purchases AS (
      SELECT visit_id,
    	event_type AS purchased
      FROM clique_bait.events
      WHERE event_type = 3)
      
    , cart AS (
      SELECT e.visit_id,
    	e.event_type,
        e.page_id
      FROM clique_bait.events AS e
      JOIN purchases AS p
      ON e.visit_id = p.visit_id
      WHERE e.event_type = 2)
    
    SELECT h.page_name AS product,
    	COUNT(c.visit_id) AS times_purchased
    FROM cart AS c
    JOIN clique_bait.page_hierarchy AS h
    ON c.page_id = h.page_id
    GROUP BY product
    ORDER BY times_purchased DESC
    LIMIT 3;

| product | times_purchased |
| ------- | --------------- |
| Lobster | 754             |
| Oyster  | 726             |
| Crab    | 719             |

---

### 🦐 B. Product Funnel Analysis

#### Using a single SQL query - create a new output table which has the following details:  
**- How many times was each product viewed?**  
**- How many times was each product added to cart?**   
**- How many times was each product added to a cart but not purchased (abandoned)?**  
**- How many times was each product purchased?**  

---

    DROP TABLE IF EXISTS products;

    CREATE TABLE products (
      product_name VARCHAR,
      times_viewed INT,
      times_added_to_cart INT,
      times_abandoned INT,
      times_purchased INT
      );

    WITH cte AS (
      SELECT h.page_name AS product_name,
    	COUNT(e.page_id) AS times_viewed
      FROM clique_bait.events AS e
      JOIN clique_bait.page_hierarchy AS h
      ON e.page_id = h.page_id
      WHERE e.page_id NOT IN (1, 2, 12, 13) AND e.event_type = 1
      GROUP BY h.page_name
      ORDER BY h.page_name)
      
    , cte2 AS (
      SELECT h.page_name AS product_name,
      	COUNT(e.event_type) AS times_added_to_cart
      FROM clique_bait.events AS e
      JOIN clique_bait.page_hierarchy AS h
      ON e.page_id = h.page_id
      WHERE e.event_type = 2
      GROUP BY h.page_name
      ORDER BY h.page_name)
      
    , purchases AS (
      SELECT visit_id,
    	event_type AS purchased
      FROM clique_bait.events
      WHERE event_type = 3)
      
    , cart AS (
      SELECT e.visit_id,
    	e.event_type,
        e.page_id
      FROM clique_bait.events AS e
      JOIN purchases AS p
      ON e.visit_id = p.visit_id
      WHERE e.event_type = 2)
    
    , num_pur AS (
      SELECT h.page_name AS product_name,
    	COUNT(c.visit_id) AS times_purchased
      FROM cart AS c
      JOIN clique_bait.page_hierarchy AS h
      ON c.page_id = h.page_id
      GROUP BY product_name
      ORDER BY product_name)
    
    , abandon AS (
      SELECT c.product_name,
    	c.times_added_to_cart - p.times_purchased AS times_abandoned
      FROM cte2 AS c
      JOIN num_pur AS p
      ON c.product_name = p.product_name
      ORDER BY c.product_name)
    
    , final AS (
      SELECT v.product_name,
    	v.times_viewed,
        c.times_added_to_cart,
        a.times_abandoned,
        p.times_purchased
      FROM cte AS v
      JOIN cte2 AS c
      ON v.product_name = c.product_name
      JOIN abandon AS a
      ON c.product_name = a.product_name
      JOIN num_pur AS p
      ON a.product_name = p.product_name
      ORDER BY v.product_name)
      
    INSERT INTO products
    (product_name, times_viewed, times_added_to_cart, times_abandoned, times_purchased)
    SELECT product_name, times_viewed, times_added_to_cart, times_abandoned, times_purchased
    FROM final;

    SELECT *
    FROM products;

| product_name   | times_viewed | times_added_to_cart | times_abandoned | times_purchased |
| -------------- | ------------ | ------------------- | --------------- | --------------- |
| Abalone        | 1525         | 932                 | 233             | 699             |
| Black Truffle  | 1469         | 924                 | 217             | 707             |
| Crab           | 1564         | 949                 | 230             | 719             |
| Kingfish       | 1559         | 920                 | 213             | 707             |
| Lobster        | 1547         | 968                 | 214             | 754             |
| Oyster         | 1568         | 943                 | 217             | 726             |
| Russian Caviar | 1563         | 946                 | 249             | 697             |
| Salmon         | 1559         | 938                 | 227             | 711             |
| Tuna           | 1515         | 931                 | 234             | 697             |

---

#### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

---

    DROP TABLE IF EXISTS product_categories;

    CREATE TABLE product_categories (
      product_category VARCHAR,
      times_viewed INT,
      times_added_to_cart INT,
      times_abandoned INT,
      times_purchased INT
      );

    WITH cte AS (
      SELECT h.product_category,
    	COUNT(e.page_id) AS times_viewed
      FROM clique_bait.events AS e
      JOIN clique_bait.page_hierarchy AS h
      ON e.page_id = h.page_id
      WHERE e.page_id NOT IN (1, 2, 12, 13) AND e.event_type = 1
      GROUP BY h.product_category
      ORDER BY h.product_category)
      
    , cte2 AS (
      SELECT h.product_category,
      	COUNT(e.event_type) AS times_added_to_cart
      FROM clique_bait.events AS e
      JOIN clique_bait.page_hierarchy AS h
      ON e.page_id = h.page_id
      WHERE e.event_type = 2
      GROUP BY h.product_category
      ORDER BY h.product_category)
      
    , purchases AS (
      SELECT visit_id,
    	event_type AS purchased
      FROM clique_bait.events
      WHERE event_type = 3)
      
    , cart AS (
      SELECT e.visit_id,
    	e.event_type,
        e.page_id
      FROM clique_bait.events AS e
      JOIN purchases AS p
      ON e.visit_id = p.visit_id
      WHERE e.event_type = 2)
    
    , num_pur AS (
      SELECT h.product_category,
    	COUNT(c.visit_id) AS times_purchased
      FROM cart AS c
      JOIN clique_bait.page_hierarchy AS h
      ON c.page_id = h.page_id
      GROUP BY product_category
      ORDER BY product_category)
    
    , abandon AS (
      SELECT c.product_category,
    	c.times_added_to_cart - p.times_purchased AS times_abandoned
      FROM cte2 AS c
      JOIN num_pur AS p
      ON c.product_category = p.product_category
      ORDER BY c.product_category)
    
    , final AS (
      SELECT v.product_category,
    	v.times_viewed,
        c.times_added_to_cart,
        a.times_abandoned,
        p.times_purchased
      FROM cte AS v
      JOIN cte2 AS c
      ON v.product_category = c.product_category
      JOIN abandon AS a
      ON c.product_category = a.product_category
      JOIN num_pur AS p
      ON a.product_category = p.product_category
      ORDER BY v.product_category)
      
    INSERT INTO product_categories
    (product_category, times_viewed, times_added_to_cart, times_abandoned, times_purchased)
    SELECT product_category, times_viewed, times_added_to_cart, times_abandoned, times_purchased
    FROM final;


    SELECT *
    FROM product_categories;

| product_category | times_viewed | times_added_to_cart | times_abandoned | times_purchased |
| ---------------- | ------------ | ------------------- | --------------- | --------------- |
| Fish             | 4633         | 2789                | 674             | 2115            |
| Luxury           | 3032         | 1870                | 466             | 1404            |
| Shellfish        | 6204         | 3792                | 894             | 2898            |

---

#### Use your 2 new output tables - answer the following questions:  

#### 1. Which product had the most views, cart adds and purchases?

    SELECT product_name,
    	times_viewed
    FROM products
    ORDER BY times_viewed DESC
    LIMIT 1;

| product_name | times_viewed |
| ------------ | ------------ |
| Oyster       | 1568         |


    SELECT product_name,
    	times_added_to_cart
    FROM products
    ORDER BY times_added_to_cart DESC
    LIMIT 1;

| product_name | times_added_to_cart |
| ------------ | ------------------- |
| Lobster      | 968                 |


    SELECT product_name,
    	times_purchased
    FROM products
    ORDER BY times_purchased DESC
    LIMIT 1;

| product_name | times_purchased |
| ------------ | --------------- |
| Lobster      | 754             |

---
#### 2. Which product was most likely to be abandoned?

    SELECT product_name,
    	times_abandoned
    FROM products
    ORDER BY times_abandoned DESC
    LIMIT 1;

| product_name   | times_abandoned |
| -------------- | --------------- |
| Russian Caviar | 249             |

---
#### 3. Which product had the highest view to purchase percentage?

    SELECT product_name,
    	ROUND((times_purchased::NUMERIC / times_viewed::NUMERIC) * 100, 2) AS percentage
    FROM products
    ORDER BY percentage DESC
    LIMIT 1;

| product_name | percentage |
| ------------ | ---------- |
| Lobster      | 48.74      |

---
#### 4. What is the average conversion rate from view to cart add?

    SELECT ROUND(AVG((times_added_to_cart::NUMERIC / times_viewed::NUMERIC) * 100), 2) AS conv_view_to_cart_add
    FROM products;

| conv_view_to_cart_add |
| --------------------- |
| 60.95                 |

---
#### 5. What is the average conversion rate from cart add to purchase?

    SELECT ROUND(AVG((times_purchased::NUMERIC / times_added_to_cart::NUMERIC) * 100), 2) AS conv_add_cart_to_pur
    FROM products;

| conv_add_cart_to_pur |
| -------------------- |
| 75.93                |

---
### 🦑 C. Campaigns Analysis

#### Generate a table that has 1 single row for every unique visit_id record and has the following columns:

**- user_id**  
**- visit_id**  
**- visit_start_time: the earliest event_time for each visit**  
**- page_views: count of page views for each visit**  
**- cart_adds: count of product cart add events for each visit**  
**- purchase: 1/0 flag if a purchase event exists for each visit**  
**- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date**  
**- impression: count of ad impressions for each visit**  
**- click: count of ad clicks for each visit**  
**- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)**  
## 🚀 Final Thoughts
