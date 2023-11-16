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


