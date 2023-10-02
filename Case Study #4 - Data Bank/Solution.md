
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
