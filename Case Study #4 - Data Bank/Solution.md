# 🏦 Case Study #4 - Data Bank

![Data Bank](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/7e07a5b1-7260-4089-934d-e773278f68fa)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-4/)

## 📚 Table of Contents

## ✏️ Business Task

## ⚠️ Data Limitations

## 🗺️ Data


## 📈 Case Study Questions
### 🖥️ A. Customer Nodes Exploration
#### 1. How many unique nodes are there on the Data Bank system?

    SELECT COUNT(DISTINCT node_id) AS unique_nodes
    FROM data_bank.customer_nodes;

| unique_nodes |
| ------------ |
| 5            |

---
#### 2. What is the number of nodes per region?

    SELECT r.region_name,
    	COUNT(c.node_id) AS num_nodes
    FROM data_bank.regions AS r
    JOIN data_bank.customer_nodes AS c
    ON r.region_id = c.region_id
    GROUP BY r.region_name
    ORDER BY r.region_name;

| region_name | num_nodes |
| ----------- | --------- |
| Africa      | 714       |
| America     | 735       |
| Asia        | 665       |
| Australia   | 770       |
| Europe      | 616       |

---
#### 3. How many customers are allocated to each region?

    SELECT r.region_name,
    	COUNT(DISTINCT c.customer_id) AS num_customers
    FROM data_bank.regions AS r
    JOIN data_bank.customer_nodes AS c
    ON r.region_id = c.region_id
    GROUP BY r.region_name
    ORDER BY r.region_name;

| region_name | num_customers |
| ----------- | ------------- |
| Africa      | 102           |
| America     | 105           |
| Asia        | 95            |
| Australia   | 110           |
| Europe      | 88            |

---
#### 4. How many days on average are customers reallocated to a different node?

    SELECT ROUND(AVG(end_date - start_date), 1) AS avg_days
    FROM data_bank.customer_nodes;

| avg_days |
| -------- |
| 416373.4 |

---
#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

    WITH num_days AS (
    	SELECT r.region_name,
      		(end_date - start_date) AS days
    	FROM data_bank.customer_nodes AS c
      	JOIN data_bank.regions AS r
      	ON c.region_id = r.region_id
    	ORDER BY r.region_name,
      		days)
        
    SELECT region_name,
    	PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY (days)) AS median,
    	PERCENTILE_CONT(0.80) WITHIN GROUP(ORDER BY (days)) AS percentile_80,
        PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY (days)) AS percentile_95
    FROM num_days
    GROUP BY region_name;

| region_name | median | percentile_80 | percentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 17.5   | 27            | 2914535.35    |
| America     | 18     | 27            | 2914534.3     |
| Asia        | 17     | 27            | 2914538       |
| Australia   | 17     | 28            | 2914533.55    |
| Europe      | 18     | 28            | 2914527       |

---

### 👩‍💻 B. Customer Transactions

#### 1. What is the unique count and total amount for each transaction type?

    SELECT txn_type,
    	COUNT(txn_type) AS num_type,
        SUM(txn_amount) AS total_amount
    FROM data_bank.customer_transactions
    GROUP BY txn_type;

| txn_type   | num_type | total_amount |
| ---------- | -------- | ------------ |
| purchase   | 1617     | 806537       |
| deposit    | 2671     | 1359168      |
| withdrawal | 1580     | 793003       |

---
#### 2. What is the average total historical deposit counts and amounts for all customers?

    WITH counts AS (
    	SELECT customer_id,
    		COUNT(txn_type) AS total_deposits,
        	SUM(txn_amount) AS total_amount
    	FROM data_bank.customer_transactions
    	WHERE txn_type = 'deposit'
    	GROUP BY customer_id)
    
    SELECT ROUND(AVG(total_deposits), 1) AS avg_num_deposits,
    	ROUND(AVG(total_amount), 1) AS avg_amount_per_cust
    FROM counts;

| avg_num_deposits | avg_amount_per_cust |
| ---------------- | ------------------- |
| 5.3              | 2718.3              |

---
#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

    WITH txn_counts AS (
    	SELECT customer_id,
    		EXTRACT(month FROM txn_date) AS month,
        	COUNT(CASE WHEN txn_type = 'purchase' THEN 1 END) AS purchase,
        	COUNT(CASE WHEN txn_type = 'deposit' THEN 1 END) AS deposit,
        	COUNT(CASE WHEN txn_type = 'withdrawal' THEN 1 END) AS withdrawal
    	FROM data_bank.customer_transactions
    	GROUP BY customer_id,
    		EXTRACT(month FROM txn_date)
    	ORDER BY customer_id)
    
    SELECT month,
    	COUNT(DISTINCT customer_id) AS customer_count
    FROM txn_counts
    WHERE deposit > 1 AND (
      purchase > 0 OR withdrawal > 0)
    GROUP BY month
    ORDER BY month;

| month | customer_count |
| ----- | -------------- |
| 1     | 168            |
| 2     | 181            |
| 3     | 192            |
| 4     | 70             |

---
#### 4. What is the closing balance for each customer at the end of the month?

    WITH end_of_month AS (
    	SELECT customer_id,
    		txn_date,
    		(date_trunc('month', txn_date) + interval '1 month - 1 day')::date AS eom,
        	txn_type,
        	txn_amount
    	FROM data_bank.customer_transactions)
    
    , amounts AS (
      SELECT customer_id,
    	eom,
        SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS deposit,
        SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS purchase,
        SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS withdrawal
      FROM end_of_month
      WHERE EXTRACT(month FROM txn_date) = EXTRACT(month FROM eom)
      GROUP BY customer_id,
    	eom
      ORDER BY customer_id,
    	eom)
    
    SELECT customer_id,
    	eom,
        SUM(deposit) - SUM(purchase + withdrawal) AS closing_balance
    FROM amounts
    GROUP BY customer_id,
    	eom
    ORDER BY customer_id,
    	eom
    LIMIT 20;

| customer_id | eom                      | closing_balance |
| ----------- | ------------------------ | --------------- |
| 1           | 2020-01-31T00:00:00.000Z | 312             |
| 1           | 2020-03-31T00:00:00.000Z | -952            |
| 2           | 2020-01-31T00:00:00.000Z | 549             |
| 2           | 2020-03-31T00:00:00.000Z | 61              |
| 3           | 2020-01-31T00:00:00.000Z | 144             |
| 3           | 2020-02-29T00:00:00.000Z | -965            |
| 3           | 2020-03-31T00:00:00.000Z | -401            |
| 3           | 2020-04-30T00:00:00.000Z | 493             |
| 4           | 2020-01-31T00:00:00.000Z | 848             |
| 4           | 2020-03-31T00:00:00.000Z | -193            |
| 5           | 2020-01-31T00:00:00.000Z | 954             |
| 5           | 2020-03-31T00:00:00.000Z | -2877           |
| 5           | 2020-04-30T00:00:00.000Z | -490            |
| 6           | 2020-01-31T00:00:00.000Z | 733             |
| 6           | 2020-02-29T00:00:00.000Z | -785            |
| 6           | 2020-03-31T00:00:00.000Z | 392             |
| 7           | 2020-01-31T00:00:00.000Z | 964             |
| 7           | 2020-02-29T00:00:00.000Z | 2209            |
| 7           | 2020-03-31T00:00:00.000Z | -640            |
| 7           | 2020-04-30T00:00:00.000Z | 90              |

---
#### 5. What is the percentage of customers who increase their closing balance by more than 5%?

    WITH end_of_month AS (
    	SELECT customer_id,
    		txn_date,
    		(date_trunc('month', txn_date) + interval '1 month - 1 day')::date AS eom,
        	txn_type,
        	txn_amount
    	FROM data_bank.customer_transactions)
    
    , amounts AS (
      SELECT customer_id,
    	eom,
        SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS deposit,
        SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS purchase,
        SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS withdrawal
      FROM end_of_month
      WHERE EXTRACT(month FROM txn_date) = EXTRACT(month FROM eom)
      GROUP BY customer_id,
    	eom
      ORDER BY customer_id,
    	eom)
    
    , closings AS (
      SELECT customer_id,
    	eom,
        SUM(deposit) - SUM(purchase + withdrawal) AS closing_balance
      FROM amounts
      GROUP BY customer_id,
    	eom
      ORDER BY customer_id,
    	eom)
        
    , balances AS (
      SELECT customer_id,
    	FIRST_VALUE(closing_balance) OVER(
          PARTITION BY customer_id
          ORDER BY customer_id) AS starting_balance,
        LAST_VALUE(closing_balance) OVER(
          PARTITION BY customer_id
          ORDER BY customer_id) AS ending_balance
      FROM closings)
    
    , growth AS (
      SELECT customer_id,
    	((ending_balance - starting_balance) / starting_balance) AS growth_rate
      FROM balances
      WHERE ((ending_balance - starting_balance) / starting_balance) >= 0.05
      GROUP BY customer_id,
    	starting_balance,
    	ending_balance)
    
    SELECT ROUND(COUNT(DISTINCT customer_id)::numeric / (
      SELECT COUNT(DISTINCT customer_id)::numeric
      FROM closings) * 100, 1) AS percent
    FROM growth;

| percent |
| ------- |
| 28.2    |

---

### 📊 Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time
  
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?
## 🚀 Final Thoughts
