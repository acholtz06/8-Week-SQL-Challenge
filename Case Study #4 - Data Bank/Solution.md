# 🏦 Case Study #4 - Data Bank

![Data Bank](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/7e07a5b1-7260-4089-934d-e773278f68fa)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-4/)

## 📚 Table of Contents
- [Business Task](#%EF%B8%8F-business-task)
- [Data Limitations](#%EF%B8%8F-data-limitations)
- [Data](#%EF%B8%8F-data)
- [Case Study Questions](#-case-study-questions)
    - [A. Customer Nodes Exploration](#%EF%B8%8F-a-customer-nodes-exploration)
    - [B. Customer Transactions](#-b-customer-transactions)
    - [C. Data Allocation Challenge](#-c-data-allocation-challenge)
    - [D. Extra Challenge](#-d-extra-challenge)
- [Final Thoughts](#-final-thoughts)
          


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
    	COUNT(DISTINCT c.node_id) AS num_nodes
    FROM data_bank.regions AS r
    JOIN data_bank.customer_nodes AS c
    ON r.region_id = c.region_id
    GROUP BY r.region_name
    ORDER BY r.region_name;

| region_name | num_nodes |
| ----------- | --------- |
| Africa      | 5         |
| America     | 5         |
| Asia        | 5         |
| Australia   | 5         |
| Europe      | 5         |


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

   
    SELECT ROUND(AVG(end_date - start_date), 0) AS avg_days
    FROM data_bank.customer_nodes
    WHERE end_date <> '9999-12-31';

| avg_days |
| -------- |
| 15       |

---
#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

    WITH num_days AS (
    	SELECT r.region_name,
      		(end_date - start_date) AS days
    	FROM data_bank.customer_nodes AS c
      	JOIN data_bank.regions AS r
      	ON c.region_id = r.region_id
        WHERE end_date <> '9999-12-31'
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
| Africa      | 15     | 24            | 28            |
| America     | 15     | 23            | 28            |
| Asia        | 15     | 23            | 28            |
| Australia   | 15     | 23            | 28            |
| Europe      | 15     | 24            | 28            |


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
        	AVG(txn_amount) AS total_amount
    	FROM data_bank.customer_transactions
    	WHERE txn_type = 'deposit'
    	GROUP BY customer_id)
    
    SELECT ROUND(AVG(total_deposits), 0) AS avg_num_deposits,
    	ROUND(AVG(total_amount), 2) AS avg_amount_per_cust
    FROM counts;

| avg_num_deposits | avg_amount_per_cust |
| ---------------- | ------------------- |
| 5                | 508.61              |

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
      	txn_date,
    	eom,
        SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) +
        SUM(CASE WHEN txn_type <> 'deposit' THEN -txn_amount ELSE 0 END) AS purchase
      FROM end_of_month
      GROUP BY customer_id,
    	eom,
      	txn_date
      ORDER BY customer_id,
    	eom)
    
    , balance AS (
      SELECT customer_id,
      	txn_date,
      	eom,
      	purchase,
      SUM(purchase) OVER(
        PARTITION BY customer_id
        ORDER BY txn_date) AS running_balance,
      RANK() OVER(
        PARTITION BY customer_id, eom
        ORDER BY txn_date DESC) AS rank
      FROM amounts)
    
    SELECT customer_id,
    	eom,
        running_balance
    FROM balance
    WHERE rank = 1
    ORDER BY customer_id,
    	eom
    LIMIT 20;

| customer_id | eom                      | running_balance |
| ----------- | ------------------------ | --------------- |
| 1           | 2020-01-31T00:00:00.000Z | 312             |
| 1           | 2020-03-31T00:00:00.000Z | -640            |
| 2           | 2020-01-31T00:00:00.000Z | 549             |
| 2           | 2020-03-31T00:00:00.000Z | 610             |
| 3           | 2020-01-31T00:00:00.000Z | 144             |
| 3           | 2020-02-29T00:00:00.000Z | -821            |
| 3           | 2020-03-31T00:00:00.000Z | -1222           |
| 3           | 2020-04-30T00:00:00.000Z | -729            |
| 4           | 2020-01-31T00:00:00.000Z | 848             |
| 4           | 2020-03-31T00:00:00.000Z | 655             |
| 5           | 2020-01-31T00:00:00.000Z | 954             |
| 5           | 2020-03-31T00:00:00.000Z | -1923           |
| 5           | 2020-04-30T00:00:00.000Z | -2413           |
| 6           | 2020-01-31T00:00:00.000Z | 733             |
| 6           | 2020-02-29T00:00:00.000Z | -52             |
| 6           | 2020-03-31T00:00:00.000Z | 340             |
| 7           | 2020-01-31T00:00:00.000Z | 964             |
| 7           | 2020-02-29T00:00:00.000Z | 3173            |
| 7           | 2020-03-31T00:00:00.000Z | 2533            |
| 7           | 2020-04-30T00:00:00.000Z | 2623            |

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
      	txn_date,
    	eom,
        SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) +
        SUM(CASE WHEN txn_type <> 'deposit' THEN -txn_amount ELSE 0 END) AS purchase
      FROM end_of_month
      GROUP BY customer_id,
    	eom,
      	txn_date
      ORDER BY customer_id,
    	eom)
    
    , balance AS (
      SELECT customer_id,
      	txn_date,
      	eom,
      	purchase,
      SUM(purchase) OVER(
        PARTITION BY customer_id
        ORDER BY txn_date) AS running_balance,
      RANK() OVER(
        PARTITION BY customer_id, eom
        ORDER BY txn_date DESC) AS rank
      FROM amounts)
    
    , lead_total AS (
      SELECT customer_id,
    	eom,
        running_balance AS starting_balance,
        (LEAD(running_balance) OVER(
          PARTITION BY customer_id)) AS ending_balance
      FROM balance
      WHERE rank = 1
      ORDER BY customer_id,
    	eom)
        
    , negative AS (
      SELECT customer_id,
      	eom,
      	starting_balance,
      	ending_balance,
      	CASE WHEN ending_balance < starting_balance AND starting_balance < 0 THEN -starting_balance ELSE starting_balance END AS divide
      FROM lead_total
      WHERE starting_balance <> 0 AND ending_balance IS NOT NULL)
    
    , growth AS (
      SELECT customer_id,
    	((ending_balance - starting_balance) / divide) AS growth_rate
      FROM negative
      WHERE ((ending_balance - starting_balance) / divide) > 0.05)
     
    SELECT ROUND(COUNT(DISTINCT(customer_id))::numeric / (
      SELECT COUNT(DISTINCT(customer_id))::numeric
      FROM data_bank.customer_transactions) * 100, 0) AS percent
    FROM growth;

| percent |
| ------- |
| 37      |

---

### 📊 C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time
  
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?
---

### 🔥 D. Extra Challenge
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

- Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!
---
## 🚀 Final Thoughts


