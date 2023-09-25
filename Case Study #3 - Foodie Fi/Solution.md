
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
#### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

    WITH plans AS (
    	SELECT *
    	FROM foodie_fi.subscriptions
    	WHERE plan_id NOT IN (1, 2, 3)
    	ORDER by customer_id,
    		plan_id)
    
    , counts AS (
      SELECT customer_id,
      	COUNT(plan_id) AS num_plans
      FROM plans
      GROUP BY customer_id
      HAVING COUNT(plan_id) = 2
      ORDER BY customer_id)
      
    , dates AS (
      SELECT p.customer_id,
      	p.plan_id,
      	p.start_date
      FROM plans AS p
      JOIN counts AS c
      ON p.customer_id = c.customer_id)
      
    , lagged AS (
      SELECT customer_id,
      	plan_id,
      	start_date,
      	LAG(start_date + 7, 1) OVER(
          PARTITION BY customer_id
          ORDER BY plan_id) AS churn_date
      FROM dates)
    
    , num_churned AS (
      SELECT COUNT(customer_id) AS churned_after_trial
      FROM lagged
      WHERE start_date = churn_date)
    
    
    SELECT churned_after_trial,
    	ROUND((churned_after_trial::numeric / 1000) * 100, 0) AS percent_churn_after_trial
    FROM num_churned;

| churned_after_trial | percent_churn_after_trial |
| ------------------- | ------------------------- |
| 92                  | 9                         |

---
#### 6. What is the number and percentage of customer plans after their initial free trial?

    WITH ranks AS (
      SELECT *,
      	RANK() OVER(
          PARTITION BY customer_id
          ORDER BY start_date) AS plan_rank
      FROM foodie_fi.subscriptions)
    
    SELECT p.plan_name,
    	COUNT(r.plan_id) AS num_in_plan,
        ROUND((COUNT(r.plan_id)::numeric / (
        	SELECT COUNT(DISTINCT customer_id)
        	FROM ranks)) * 100, 0) AS percent_in_plan
    FROM ranks AS r
    JOIN foodie_fi.plans AS p
    ON r.plan_id = p.plan_id
    WHERE r.plan_rank = 2
    GROUP BY p.plan_name,
    	r.plan_id
    ORDER BY r.plan_id;

| plan_name     | num_in_plan | percent_in_plan |
| ------------- | ----------- | --------------- |
| basic monthly | 546         | 55              |
| pro monthly   | 325         | 33              |
| pro annual    | 37          | 4               |
| churn         | 92          | 9               |

---
#### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

    WITH date AS (
      SELECT customer_id,
      	MAX(start_date) AS current_plan
      FROM foodie_fi.subscriptions
      WHERE start_date <= '2020-12-31'
      GROUP BY customer_id
      ORDER BY customer_id)
    
    SELECT p.plan_name,
    	COUNT(s.plan_id) AS num_in_plan,
        ROUND((COUNT(s.plan_id)::numeric / (
          SELECT COUNT(DISTINCT customer_id)
          FROM date)) * 100, 0) AS percent_in_plan
    FROM date AS d
    JOIN foodie_fi.subscriptions AS s
    ON d.customer_id = s.customer_id AND d.current_plan = s.start_date
    JOIN foodie_fi.plans AS p
    ON s.plan_id = p.plan_id
    GROUP BY p.plan_name,
    	s.plan_id
    ORDER BY s.plan_id;

| plan_name     | num_in_plan | percent_in_plan |
| ------------- | ----------- | --------------- |
| trial         | 19          | 2               |
| basic monthly | 224         | 22              |
| pro monthly   | 326         | 33              |
| pro annual    | 195         | 20              |
| churn         | 236         | 24              |

---
#### 8. How many customers have upgraded to an annual plan in 2020?

    SELECT COUNT(DISTINCT customer_id) AS num_upgraded
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3 AND start_date IN (
      SELECT start_date
      FROM foodie_fi.subscriptions
      WHERE start_date BETWEEN '2020-01-01' AND '2020-12-31');

| num_upgraded |
| ------------ |
| 195          |

---
#### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

    WITH join_date AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 0)
    
    , annual AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 3)
    
    SELECT ROUND(AVG(a.start_date - j.start_date), 0) AS avg_days_to_upgrade
    FROM join_date AS j
    JOIN annual AS a
    ON j.customer_id = a.customer_id;

| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---
