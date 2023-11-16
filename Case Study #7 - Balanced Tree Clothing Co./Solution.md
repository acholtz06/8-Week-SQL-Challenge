# Case Study #7 - Balanced Tree Clothing Co.

![Case Study 7](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/14ef1016-7398-4396-82bd-e43302db8b97)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-7/)

## Case Study Questions

### High Level Sales Analysis

---

#### 1. What was the total quantity sold for all products?

    SELECT SUM(qty) AS total_quantity_sold
    FROM balanced_tree.sales;

| total_quantity_sold |
| ------------------- |
| 45216               |

---
#### 2. What is the total generated revenue for all products before discounts?

    SELECT SUM(qty * price) AS total_revenue_before_discounts
    FROM balanced_tree.sales;

| total_revenue_before_discounts |
| ------------------------------ |
| 1289453                        |

---
#### 3. What was the total discount amount for all products?

    SELECT ROUND(SUM(qty * price * (discount::NUMERIC / 100)), 2) AS total_discount_amount
    FROM balanced_tree.sales;

| total_discount_amount |
| --------------------- |
| 156229.14             |

---

### Transaction Analysis

---

#### 1. How many unique transactions were there?

    SELECT COUNT(DISTINCT txn_id) AS unique_transactions
    FROM balanced_tree.sales;

| unique_transactions |
| ------------------- |
| 2500                |

---
#### 2. What is the average unique products purchased in each transaction?

    SELECT ROUND(AVG(prod_per_txn), 0) avg_unique_products_per_txn
    FROM (
      SELECT DISTINCT txn_id,
      	COUNT(prod_id) OVER(
          PARTITION BY txn_id) AS prod_per_txn
      FROM balanced_tree.sales
      ) AS count;

| avg_unique_products_per_txn |
| --------------------------- |
| 6                           |

---
#### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

    WITH revenue AS (
      SELECT txn_id,
      	SUM(qty * price)
      FROM balanced_tree.sales
      GROUP BY txn_id)
    
    SELECT PERCENTILE_CONT(.25) WITHIN GROUP (ORDER BY sum) AS percentile_25,
    PERCENTILE_CONT(.5) WITHIN GROUP (ORDER BY sum) AS percentile_50,
    PERCENTILE_CONT(.75) WITHIN GROUP (ORDER BY sum) AS percentile_75
    FROM revenue;

| percentile_25 | percentile_50 | percentile_75 |
| ------------- | ------------- | ------------- |
| 375.75        | 509.5         | 647           |

---
#### 4. What is the average discount value per transaction?

    SELECT ROUND(AVG(discount), 2) AS avg_discount_per_txn
    FROM (
      SELECT DISTINCT txn_id,
      	SUM(qty * price * (discount::NUMERIC / 100)) OVER(
      	PARTITION BY txn_id) AS discount
      FROM balanced_tree.sales
      ) AS d;

| avg_discount_per_txn |
| -------------------- |
| 62.49                |

---
#### 5. What is the percentage split of all transactions for members vs non-members?

    WITH counts AS (
      SELECT COUNT(txn_id) AS total_txn,
      	SUM(CASE WHEN member = 'true' THEN 1 ELSE 0 END) AS y,
      	SUM(CASE WHEN member = 'false' THEN 1 ELSE 0 END) AS n
      FROM (
        SELECT DISTINCT txn_id,
    		member
        FROM balanced_tree.sales
      	) AS m)
        
    SELECT ROUND((y::NUMERIC / total_txn::NUMERIC) * 100, 2) AS percent_member,
    	ROUND((n::NUMERIC / total_txn::NUMERIC) * 100, 2) AS percent_non_member
    FROM counts;

| percent_member | percent_non_member |
| -------------- | ------------------ |
| 60.20          | 39.80              |

---
#### 6. What is the average revenue for member transactions and non-member transactions?

    WITH t AS (
      SELECT DISTINCT txn_id,
    	member,
        SUM(qty * price) OVER(
          PARTITION BY txn_id) AS amount
      FROM balanced_tree.sales)
      
    SELECT member,
    	ROUND(AVG(amount), 2) AS avg_revenue_per_txn
    FROM t
    GROUP BY member
    ORDER BY member DESC;

| member | avg_revenue_per_txn |
| ------ | ------------------- |
| true   | 516.27              |
| false  | 515.04              |

---


