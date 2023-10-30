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
### 🦑 C. Campaigns Analysis
## 🚀 Final Thoughts
