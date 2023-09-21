
---

#### 1. How many customers has Foodie-Fi ever had?

    SELECT COUNT(DISTINCT customer_id) AS total_customers
    FROM foodie_fi.subscriptions;

| total_customers |
| --------------- |
| 1000            |

---
#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

    SELECT DATE_TRUNC('month', start_date) AS month,
    	COUNT(plan_id) AS trial_plan
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | trial_plan |
| ------------------------ | ---------- |
| 2020-01-01T00:00:00.000Z | 88         |
| 2020-02-01T00:00:00.000Z | 68         |
| 2020-03-01T00:00:00.000Z | 94         |
| 2020-04-01T00:00:00.000Z | 81         |
| 2020-05-01T00:00:00.000Z | 88         |
| 2020-06-01T00:00:00.000Z | 79         |
| 2020-07-01T00:00:00.000Z | 89         |
| 2020-08-01T00:00:00.000Z | 88         |
| 2020-09-01T00:00:00.000Z | 87         |
| 2020-10-01T00:00:00.000Z | 79         |
| 2020-11-01T00:00:00.000Z | 75         |
| 2020-12-01T00:00:00.000Z | 84         |

---
#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

    SELECT DATE_TRUNC('month', start_date) AS month,
    	COUNT(plan_id) AS basic_monthly
    FROM foodie_fi.subscriptions
    WHERE plan_id = 1
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | basic_monthly |
| ------------------------ | ------------- |
| 2020-01-01T00:00:00.000Z | 31            |
| 2020-02-01T00:00:00.000Z | 37            |
| 2020-03-01T00:00:00.000Z | 49            |
| 2020-04-01T00:00:00.000Z | 43            |
| 2020-05-01T00:00:00.000Z | 53            |
| 2020-06-01T00:00:00.000Z | 51            |
| 2020-07-01T00:00:00.000Z | 44            |
| 2020-08-01T00:00:00.000Z | 54            |
| 2020-09-01T00:00:00.000Z | 38            |
| 2020-10-01T00:00:00.000Z | 46            |
| 2020-11-01T00:00:00.000Z | 49            |
| 2020-12-01T00:00:00.000Z | 43            |
| 2021-01-01T00:00:00.000Z | 8             |


    SELECT DATE_TRUNC('month', start_date) AS month,
    	COUNT(plan_id) AS pro_monthly
    FROM foodie_fi.subscriptions
    WHERE plan_id = 2
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | pro_monthly |
| ------------------------ | ----------- |
| 2020-01-01T00:00:00.000Z | 29          |
| 2020-02-01T00:00:00.000Z | 29          |
| 2020-03-01T00:00:00.000Z | 37          |
| 2020-04-01T00:00:00.000Z | 31          |
| 2020-05-01T00:00:00.000Z | 39          |
| 2020-06-01T00:00:00.000Z | 39          |
| 2020-07-01T00:00:00.000Z | 40          |
| 2020-08-01T00:00:00.000Z | 56          |
| 2020-09-01T00:00:00.000Z | 52          |
| 2020-10-01T00:00:00.000Z | 47          |
| 2020-11-01T00:00:00.000Z | 32          |
| 2020-12-01T00:00:00.000Z | 48          |
| 2021-01-01T00:00:00.000Z | 26          |
| 2021-02-01T00:00:00.000Z | 12          |
| 2021-03-01T00:00:00.000Z | 15          |
| 2021-04-01T00:00:00.000Z | 7           |


    SELECT DATE_TRUNC('month', start_date) AS month,
    	COUNT(plan_id) AS pro_annual
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | pro_annual |
| ------------------------ | ---------- |
| 2020-01-01T00:00:00.000Z | 2          |
| 2020-02-01T00:00:00.000Z | 5          |
| 2020-03-01T00:00:00.000Z | 7          |
| 2020-04-01T00:00:00.000Z | 11         |
| 2020-05-01T00:00:00.000Z | 13         |
| 2020-06-01T00:00:00.000Z | 16         |
| 2020-07-01T00:00:00.000Z | 20         |
| 2020-08-01T00:00:00.000Z | 24         |
| 2020-09-01T00:00:00.000Z | 25         |
| 2020-10-01T00:00:00.000Z | 32         |
| 2020-11-01T00:00:00.000Z | 20         |
| 2020-12-01T00:00:00.000Z | 20         |
| 2021-01-01T00:00:00.000Z | 24         |
| 2021-02-01T00:00:00.000Z | 17         |
| 2021-03-01T00:00:00.000Z | 9          |
| 2021-04-01T00:00:00.000Z | 13         |


    SELECT DATE_TRUNC('month', start_date) AS month,
    	COUNT(plan_id) AS churn
    FROM foodie_fi.subscriptions
    WHERE plan_id = 4
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | churn |
| ------------------------ | ----- |
| 2020-01-01T00:00:00.000Z | 9     |
| 2020-02-01T00:00:00.000Z | 9     |
| 2020-03-01T00:00:00.000Z | 13    |
| 2020-04-01T00:00:00.000Z | 18    |
| 2020-05-01T00:00:00.000Z | 21    |
| 2020-06-01T00:00:00.000Z | 19    |
| 2020-07-01T00:00:00.000Z | 28    |
| 2020-08-01T00:00:00.000Z | 13    |
| 2020-09-01T00:00:00.000Z | 23    |
| 2020-10-01T00:00:00.000Z | 26    |
| 2020-11-01T00:00:00.000Z | 32    |
| 2020-12-01T00:00:00.000Z | 25    |
| 2021-01-01T00:00:00.000Z | 19    |
| 2021-02-01T00:00:00.000Z | 18    |
| 2021-03-01T00:00:00.000Z | 21    |
| 2021-04-01T00:00:00.000Z | 13    |

---
#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

    SELECT COUNT(customer_id) AS num_churn
    		FROM foodie_fi.subscriptions
    		WHERE plan_id = 4;

| num_churn |
| --------- |
| 307       |


    WITH churn AS (
    	SELECT COUNT(DISTINCT customer_id) AS customers,
    		(
          	    SELECT COUNT(customer_id)
    		    FROM foodie_fi.subscriptions
    		    WHERE plan_id = 4
        	) AS churn
    	FROM foodie_fi.subscriptions)
    
    SELECT ROUND((churn::numeric / customers::numeric) * 100, 1) AS churn_percent
    FROM churn;

| churn_percent |
| ------------- |
| 30.7          |

---

