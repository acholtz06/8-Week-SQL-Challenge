# 🥑 Case Study #3 - Foodie-Fi

![Foodie-FI](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/7fb14a2f-eb1f-4903-809e-b3fd7431f31e)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-3/)

## 📚 Table of Contents
- [Business Task](#%EF%B8%8F-business-task)
- [Data Limitations](#%EF%B8%8F-data-limitations)
- [Data](#-data)
- [Case Study Questions](#-case-study-questions)
    - [A. Customer Journey](#-a-customer-journey)
    - [B. Data Analysis Questions](#-b-data-analysis-questions)
    - [C. Challenge Payment Question](#-c-challenge-payment-question)
    - [D. Outside the Box Questions](#-d-outside-the-box-questions)
- [Final Thoughts](#-final-thoughts)

## ✏️ Business Task

Danny has created a new subscription service that streams only food related content - like Netflix for cooking and food shows from around the world. He started the service in 2020 and has a few options for monthly and annual subscriptions.  

The data that is provided will help Danny to make data driven descisions about future investments or new features that can be added.

## ⚠️ Data Limitations

This has been the case study with the least limitations out of the ones that I have done so far. There are plenty of customers to analyze and the time period spans more than a year, which is great for spotting trends in the data.  

The one thing that I feel could have been included to provide more insight is a customer survey asking why they decided to upgrade or drop the service. Since this wasn't included in the beginning, we can look at trends in the data we have and further investigate by sending out surveys to see why these trends are happening.

## 🔪 Data

There are two tables provided:
- plans
- subscriptions
  
![Foodie-Fi Data](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/dad8a118-fb61-4b26-9296-b4ea353e96aa)


## 🥘 Case Study Questions

### 👩‍🍳 A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

---

    SELECT s.customer_id,
    	p.plan_name,
        s.start_date
    FROM foodie_fi.subscriptions AS s
    JOIN foodie_fi.plans AS p
    ON s.plan_id = p.plan_id
    WHERE s.customer_id = 1;

Steps:
- Joined the subscriptions and the plans tables so that I could use the plan_name instead of the plan_id
- Selected the customer_id, plan_name, and start_date
- Filtered for one customer in the sample subscriptions table (provided on the Data With Danny page linked above)
- Changed the filter for each customer in the sample table


| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 1           | trial         | 2020-08-01T00:00:00.000Z |
| 1           | basic monthly | 2020-08-08T00:00:00.000Z |

- Customer 1 started their free trial on 2020-08-01
- After the 1 week free trial, they upgraded to the basic monthly plan


| customer_id | plan_name  | start_date               |
| ----------- | ---------- | ------------------------ |
| 2           | trial      | 2020-09-20T00:00:00.000Z |
| 2           | pro annual | 2020-09-27T00:00:00.000Z |

- Customer 2 started their free trial on 2020-09-20
- After the 1 week free trial, they upgraded to the pro annual plan


| customer_id | plan_name | start_date               |
| ----------- | --------- | ------------------------ |
| 11          | trial     | 2020-11-19T00:00:00.000Z |
| 11          | churn     | 2020-11-26T00:00:00.000Z |

- Customer 11 started their free trial on 2020-11-19
- After the 1 week free trial, they cancelled their supscription


| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 13          | trial         | 2020-12-15T00:00:00.000Z |
| 13          | basic monthly | 2020-12-22T00:00:00.000Z |
| 13          | pro monthly   | 2021-03-29T00:00:00.000Z |

- Customer 13 started their free trial on 2020-12-15
- After their 1 week free trial, they upgraded to the basic monthly plan
- Three months later, they upgraded to the pro monthly plan


| customer_id | plan_name   | start_date               |
| ----------- | ----------- | ------------------------ |
| 15          | trial       | 2020-03-17T00:00:00.000Z |
| 15          | pro monthly | 2020-03-24T00:00:00.000Z |
| 15          | churn       | 2020-04-29T00:00:00.000Z |

- Customer 15 started their free trial on 2020-03-17
- After their 1 week free trial, they upgraded to the pro monthly plan
- One month later, they cancelled their subscription

| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 16          | trial         | 2020-05-31T00:00:00.000Z |
| 16          | basic monthly | 2020-06-07T00:00:00.000Z |
| 16          | pro annual    | 2020-10-21T00:00:00.000Z |

- Customer 16 started their free trial on 2020-05-31
- After their 1 week free trial, they upgraded to the basic monthly plan
- Four monthls later, they upgraded to the pro annual plan


| customer_id | plan_name   | start_date               |
| ----------- | ----------- | ------------------------ |
| 18          | trial       | 2020-07-06T00:00:00.000Z |
| 18          | pro monthly | 2020-07-13T00:00:00.000Z |

- Customer 18 started their free trial on 2020-07-06
- After their 1 week free trial, they upgraded to the pro monthly plan

| customer_id | plan_name   | start_date               |
| ----------- | ----------- | ------------------------ |
| 19          | trial       | 2020-06-22T00:00:00.000Z |
| 19          | pro monthly | 2020-06-29T00:00:00.000Z |
| 19          | pro annual  | 2020-08-29T00:00:00.000Z |

- Customer 19 started their free trial on 2020-06-22
- After their 1 week free trial, they upgraded to the pro monthly plan
- Two months later, they upgraded to the pro annual plan
---

### 📺 B. Data Analysis Questions
#### 1. How many customers has Foodie-Fi ever had?

    SELECT COUNT(DISTINCT customer_id) AS total_customers
    FROM foodie_fi.subscriptions;

| total_customers |
| --------------- |
| 1000            |

Steps:
- Counted the distinct customer_ids from the subscriptions table

Insights:
- There have been 1000 total customers

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

Steps:
- Used DATE_TRUNC for the start_dates to have all dates start on the first of the month
- Counted the plan_ids
- Filtered for only trial plans
- Grouped by the truncated dates
- Ordered by month

Insights:
- The second month had the lowest number of new trial members
- The third month had the highest number of new trial members
- The number of new trial members per month doesn't vary too much
- The only outliers that stand out are the second and third months

---
#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

    SELECT DATE_TRUNC('month', start_date) AS month,
    	SUM(CASE WHEN plan_id = 0 THEN 1 ELSE 0 END) AS trial,
    	SUM(CASE WHEN plan_id = 1 THEN 1 ELSE 0 END) AS basic_monthly,
        SUM(CASE WHEN plan_id = 2 THEN 1 ELSE 0 END) AS pro_monthly,
        SUM(CASE WHEN plan_id = 3 THEN 1 ELSE 0 END) AS pro_annual,
        SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churn
    FROM foodie_fi.subscriptions
    WHERE EXTRACT(year FROM start_date) > '2020'
    GROUP BY DATE_TRUNC('month', start_date)
    ORDER BY month;

| month                    | trial | basic_monthly | pro_monthly | pro_annual | churn |
| ------------------------ | ----- | ------------- | ----------- | ---------- | ----- |
| 2021-01-01T00:00:00.000Z | 0     | 8             | 26          | 24         | 19    |
| 2021-02-01T00:00:00.000Z | 0     | 0             | 12          | 17         | 18    |
| 2021-03-01T00:00:00.000Z | 0     | 0             | 15          | 9          | 21    |
| 2021-04-01T00:00:00.000Z | 0     | 0             | 7           | 13         | 13    |

Steps:
- Used DATE_TRUNC for the start_dates to have all dates start on the first of the month
- Used CASE statements for each of the plans where the plan = 1 if it matched the plan I was counting, and 0 if it didin't
- Filtered for only dates after the year 2020
- Grouped by the truncated month

Insights:
- No new trial customers started after 2020
- After the 8 upgrades in January, no one else upgraded to the basic monthly plan
- Most pro monthly and pro annual upgrades happened in January
- The most churned accounts happened in March

---
#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?


    WITH churn AS (
    	SELECT COUNT(DISTINCT customer_id) AS customers,
    		(
          	    SELECT COUNT(customer_id)
    		    FROM foodie_fi.subscriptions
    		    WHERE plan_id = 4
        	) AS num_churn
    	FROM foodie_fi.subscriptions)
    
    SELECT num_churn,
    	ROUND((num_churn::numeric / customers::numeric) * 100, 1) AS churn_percent
    FROM churn;

| num_churn | churn_percent |
| --------- | ------------- |
| 307       | 30.7          |

Steps:
- Created a CTE called churn
- Counted the distinct customer_ids to get the total number of customers
- Used a subquery to count the customer_ids that had a plan_id of 4 (or churn)
- Used the CTE to select the number of customers that churned
- Divided the number of churned customers by the total number of customers and cast both of those numbers as the numeric data type
- Multiplied by 100 to show the percentage
- Rounded that answer to 1 decimal place

Insighs:
- 307 out of 1000 customer accounts churned
- 30.7% of the total customers churned

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

Steps:
- CTE #1
    - Filtered out any plans from the subscriptions table that weren't either trial or churn
- CTE #2
    - Selected the customer_id and counted the plan_ids
    - Grouped by the customer_id
    - Filtered for only customers that had a count of 2, meaning that they had both a trial plan and churn
- CTE #3
    - Joined the plans and counts CTEs on customer_id
    - Selected the customer_id, plan_id, and start_date
    - The results of this CTE showed each customer, their trial start date, and their churn date
- CTE #4
    - Selected everything from the previous CTE
    - Used the LAG function to create a new column that diplayed the dates in the previous column by 1, and with 7 days added. What this did is showed when the free trial ended, and put that date in the same record as the actual churn date
- CTE #5
    - Here is where I could look at each record and see if the end date of the free trial matched the churn date. Any dates that matched meant that the customer churned immediately after their free trial
    - I counted the customer_ids
    - Filtered for only ids where the start_date = churn_date
- Final Query
    - Selected the number of customers who churned right after the trial
    - Cast the number of customers who churned after the trial as numeric and divided that number by 1000. I used 1000 becuase I know that to be the total number of customers based on previous queries
    - Multiplied that number by 100 to show the percentage
    - Rounded to the nearest whole number

Insights:
- 92 customers cancelled their subscriptions immediately after their free trial period
- That comes to 9% of total customers

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

#### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

    WITH join_date AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 0)
    
    , annual AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 3)
    
    
    , bucket AS (
    SELECT WIDTH_BUCKET((a.start_date - j.start_date), 1, 360, 12) AS buckets,
    	ROUND(AVG(a.start_date - j.start_date), 0) AS days
    FROM join_date as j
    JOIN annual AS a
    ON j.customer_id = a.customer_id
    GROUP BY buckets
    ORDER BY buckets)
    
    SELECT CONCAT((buckets - 1) * 30 + 1, ' - ', buckets * 30) AS range_of_days,
    	days AS avg_time_to_upgrade
    FROM bucket;

| range_of_days | avg_time_to_upgrade |
| ------------- | ------------------- |
| 1 - 30        | 10                  |
| 31 - 60       | 42                  |
| 61 - 90       | 71                  |
| 91 - 120      | 101                 |
| 121 - 150     | 133                 |
| 151 - 180     | 162                 |
| 181 - 210     | 191                 |
| 211 - 240     | 224                 |
| 241 - 270     | 257                 |
| 271 - 300     | 285                 |
| 301 - 330     | 327                 |
| 331 - 360     | 346                 |

---

#### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

    WITH pro_monthly AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 2 AND EXTRACT(year FROM start_date) = '2020')
    
    , basic_monthly AS (
      SELECT *
      FROM foodie_fi.subscriptions
      WHERE plan_id = 1 AND EXTRACT(year FROM start_date) = '2020')
      
    SELECT COUNT(p.customer_id) AS num_downgraded
    FROM pro_monthly AS p
    JOIN basic_monthly AS b
    ON p.customer_id = b.customer_id
    WHERE b.start_date > p.start_date;

| num_downgraded |
| -------------- |
| 0              |

---

### 💵 C. Challenge Payment Question

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

**Payments Table:**

    WITH lead AS (
    	SELECT *,
    		LEAD(start_date) OVER(
          	PARTITION BY customer_id
          	ORDER BY customer_id, start_date) AS end_date
    	FROM foodie_fi.subscriptions
    	WHERE plan_id <> 0
    	ORDER BY customer_id)
    
    , ending AS (
      SELECT customer_id,
      	plan_id,
      	start_date,
      	CASE WHEN end_date IS NULL
      		THEN '2020-12-31'
      		WHEN start_date = end_date
      		THEN end_date - 1
      		ELSE end_date END AS end_date
      FROM lead
      ORDER BY customer_id)
    
    SELECT e.customer_id,
    	e.plan_id,
        p.plan_name,
        GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month') AS payment_date,
        p.price,
        RANK() OVER(
          PARTITION BY e.customer_id
          ORDER BY GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month')) AS payment_order
    FROM ending AS e
    JOIN foodie_fi.plans AS p
    ON e.plan_id = p.plan_id
    WHERE e.plan_id <> 4
    ORDER BY e.customer_id
    LIMIT 50;

| customer_id | plan_id | plan_name     | payment_date             | price  | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 2           | 3       | pro annual    | 2020-10-27T00:00:00.000Z | 199.00 | 2             |
| 2           | 3       | pro annual    | 2020-11-27T00:00:00.000Z | 199.00 | 3             |
| 2           | 3       | pro annual    | 2020-12-27T00:00:00.000Z | 199.00 | 4             |
| 3           | 1       | basic monthly | 2020-01-20T00:00:00.000Z | 9.90   | 1             |
| 3           | 1       | basic monthly | 2020-02-20T00:00:00.000Z | 9.90   | 2             |
| 3           | 1       | basic monthly | 2020-03-20T00:00:00.000Z | 9.90   | 3             |
| 3           | 1       | basic monthly | 2020-04-20T00:00:00.000Z | 9.90   | 4             |
| 3           | 1       | basic monthly | 2020-05-20T00:00:00.000Z | 9.90   | 5             |
| 3           | 1       | basic monthly | 2020-06-20T00:00:00.000Z | 9.90   | 6             |
| 3           | 1       | basic monthly | 2020-07-20T00:00:00.000Z | 9.90   | 7             |
| 3           | 1       | basic monthly | 2020-08-20T00:00:00.000Z | 9.90   | 8             |
| 3           | 1       | basic monthly | 2020-09-20T00:00:00.000Z | 9.90   | 9             |
| 3           | 1       | basic monthly | 2020-10-20T00:00:00.000Z | 9.90   | 10            |
| 3           | 1       | basic monthly | 2020-11-20T00:00:00.000Z | 9.90   | 11            |
| 3           | 1       | basic monthly | 2020-12-20T00:00:00.000Z | 9.90   | 12            |
| 4           | 1       | basic monthly | 2020-01-24T00:00:00.000Z | 9.90   | 1             |
| 4           | 1       | basic monthly | 2020-02-24T00:00:00.000Z | 9.90   | 2             |
| 4           | 1       | basic monthly | 2020-03-24T00:00:00.000Z | 9.90   | 3             |
| 5           | 1       | basic monthly | 2020-08-10T00:00:00.000Z | 9.90   | 1             |
| 5           | 1       | basic monthly | 2020-09-10T00:00:00.000Z | 9.90   | 2             |
| 5           | 1       | basic monthly | 2020-10-10T00:00:00.000Z | 9.90   | 3             |
| 5           | 1       | basic monthly | 2020-11-10T00:00:00.000Z | 9.90   | 4             |
| 5           | 1       | basic monthly | 2020-12-10T00:00:00.000Z | 9.90   | 5             |
| 6           | 1       | basic monthly | 2020-12-30T00:00:00.000Z | 9.90   | 1             |
| 6           | 1       | basic monthly | 2021-01-30T00:00:00.000Z | 9.90   | 2             |
| 7           | 1       | basic monthly | 2020-02-12T00:00:00.000Z | 9.90   | 1             |
| 7           | 1       | basic monthly | 2020-03-12T00:00:00.000Z | 9.90   | 2             |
| 7           | 1       | basic monthly | 2020-04-12T00:00:00.000Z | 9.90   | 3             |
| 7           | 1       | basic monthly | 2020-05-12T00:00:00.000Z | 9.90   | 4             |
| 7           | 2       | pro monthly   | 2020-05-22T00:00:00.000Z | 19.90  | 5             |
| 7           | 2       | pro monthly   | 2020-06-22T00:00:00.000Z | 19.90  | 6             |
| 7           | 2       | pro monthly   | 2020-07-22T00:00:00.000Z | 19.90  | 7             |
| 7           | 2       | pro monthly   | 2020-08-22T00:00:00.000Z | 19.90  | 8             |
| 7           | 2       | pro monthly   | 2020-09-22T00:00:00.000Z | 19.90  | 9             |
| 7           | 2       | pro monthly   | 2020-10-22T00:00:00.000Z | 19.90  | 10            |
| 7           | 2       | pro monthly   | 2020-11-22T00:00:00.000Z | 19.90  | 11            |
| 7           | 2       | pro monthly   | 2020-12-22T00:00:00.000Z | 19.90  | 12            |
| 8           | 1       | basic monthly | 2020-06-18T00:00:00.000Z | 9.90   | 1             |
| 8           | 1       | basic monthly | 2020-07-18T00:00:00.000Z | 9.90   | 2             |
| 8           | 2       | pro monthly   | 2020-08-03T00:00:00.000Z | 19.90  | 3             |
| 8           | 2       | pro monthly   | 2020-09-03T00:00:00.000Z | 19.90  | 4             |
| 8           | 2       | pro monthly   | 2020-10-03T00:00:00.000Z | 19.90  | 5             |
| 8           | 2       | pro monthly   | 2020-11-03T00:00:00.000Z | 19.90  | 6             |
| 8           | 2       | pro monthly   | 2020-12-03T00:00:00.000Z | 19.90  | 7             |

---

### 🔥 D. Outside the Box Questions
#### 1. How would you calculate the rate of growth for Foodie-Fi?

    WITH lead AS (
    	SELECT *,
    		LEAD(start_date) OVER(
          	PARTITION BY customer_id
          	ORDER BY customer_id, start_date) AS end_date
    	FROM foodie_fi.subscriptions
    	WHERE plan_id <> 0
    	ORDER BY customer_id)
    
    , ending AS (
      SELECT customer_id,
      	plan_id,
      	start_date,
      	CASE WHEN end_date IS NULL
      		THEN '2020-12-31'
      		WHEN start_date = end_date
      		THEN end_date - 1
      		ELSE end_date END AS end_date
      FROM lead
      ORDER BY customer_id)
    
    , payments AS (
      SELECT e.customer_id,
    	e.plan_id,
        p.plan_name,
        GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month') AS payment_date,
        p.price,
        RANK() OVER(
          PARTITION BY e.customer_id
          ORDER BY GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month')) AS payment_order
      FROM ending AS e
      JOIN foodie_fi.plans AS p
      ON e.plan_id = p.plan_id
      WHERE e.plan_id <> 4
      ORDER BY e.customer_id)
    
    SELECT EXTRACT(month FROM payment_date) AS month,
    	COUNT(DISTINCT customer_id) AS customer_count
    FROM payments
    WHERE EXTRACT(month FROM payment_date) IN (1, 12)
    GROUP BY EXTRACT(month FROM payment_date);

| month | customer_count |
| ----- | -------------- |
| 1     | 216            |
| 12    | 752            |


    SELECT ((752 - 216) / 216) * 100 AS customer_growth_rate_percentage;

| customer_growth_rate_percentage |
| ------------------------------- |
| 200                             |


    WITH lead AS (
    	SELECT *,
    		LEAD(start_date) OVER(
          	PARTITION BY customer_id
          	ORDER BY customer_id, start_date) AS end_date
    	FROM foodie_fi.subscriptions
    	WHERE plan_id <> 0
    	ORDER BY customer_id)
    
    , ending AS (
      SELECT customer_id,
      	plan_id,
      	start_date,
      	CASE WHEN end_date IS NULL
      		THEN '2020-12-31'
      		WHEN start_date = end_date
      		THEN end_date - 1
      		ELSE end_date END AS end_date
      FROM lead
      ORDER BY customer_id)
    
    , payments AS (
      SELECT e.customer_id,
    	e.plan_id,
        p.plan_name,
        GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month') AS payment_date,
        p.price,
        RANK() OVER(
          PARTITION BY e.customer_id
          ORDER BY GENERATE_SERIES(e.start_date, e.end_date, INTERVAL '1 month')) AS payment_order
      FROM ending AS e
      JOIN foodie_fi.plans AS p
      ON e.plan_id = p.plan_id
      WHERE e.plan_id <> 4
      ORDER BY e.customer_id)
    
    SELECT EXTRACT(month FROM payment_date) AS month,
    	SUM(price) AS profit
    FROM payments
    WHERE EXTRACT(month FROM payment_date) IN (1, 12)
    GROUP BY EXTRACT(month FROM payment_date);

| month | profit   |
| ----- | -------- |
| 12    | 47907.20 |
| 1     | 4620.60  |



    SELECT ROUND(((47907.20 - 4620.60) / 4620.60) * 100, 0) AS profit_growth_rate_percentage;

| profit_growth_rate_percentage |
| ----------------------------- |
| 937                           |

---

## 🚀 Final Thoughts

