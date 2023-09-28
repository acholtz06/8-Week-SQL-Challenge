# 🍕 Case Study #2 - Pizza Runner

![Pizza Runner Img](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/781423cd-5ac7-4c48-807f-c2678a274182)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-2/)

## Table of Contents

[Business Task](#-business-task)  


    
## ✏️ Business Task

Danny has a pizza restaurant and he wants to expand his business by hiring runners to pick up pizzas from Pizza Runner Headquarters and deliver them right the the customer's home. He would like to take a look at how the runner system is working and see if it is worth expanding the program.


## ⚠️ Data Limitations

The first limitation I noticed with this data is the fact that there aren't enough runners in the program yet to spot any real trends in some areas. Another limitation is that a couple of the fields are a bit ambiguous. In a real situation, I would be able to reach out for clarification, but in this case I used my best judgement. An example of this is the difference between the order time and the pickup time. There are a couple of questions that ask how long the pizza takes to prepare and how long the runner takes to show up for the order. I am unable to know if the pizza has been done and waiting when the runner arrives, or if the runners are waiting on the pizzas to finish cooking. For this case study, I used the difference between the two times for both situations and if there is an area of concern, it can be investigated further.


## 🍅 Data

![Pizza Runner Data](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/ce45cbaa-f2ec-491d-a0db-7034c50be8df)


## 🧀 Data Cleaning
    UPDATE pizza_runner.customer_orders
    SET exclusions = NULL
    WHERE exclusions = 'null';

    UPDATE pizza_runner.customer_orders
    SET exclusions = NULL
    WHERE exclusions = '';

    UPDATE pizza_runner.customer_orders
    SET extras = NULL
    WHERE extras = 'null';

    UPDATE pizza_runner.customer_orders
    SET extras = NULL
    WHERE extras = '';

    UPDATE pizza_runner.runner_orders
    SET pickup_time = NULL
    WHERE pickup_time = 'null';

    ALTER TABLE pizza_runner.runner_orders
    ALTER COLUMN pickup_time TYPE TIMESTAMP
    USING pickup_time::TIMESTAMP;
    
    UPDATE pizza_runner.runner_orders
    SET distance = NULL
    WHERE distance = 'null';

    UPDATE pizza_runner.runner_orders
    SET duration = NULL
    WHERE duration = 'null';

    UPDATE pizza_runner.runner_orders
    SET cancellation = NULL
    WHERE cancellation = 'null';

    UPDATE pizza_runner.runner_orders
    SET cancellation = NULL
    WHERE cancellation = '';

    UPDATE pizza_runner.runner_orders
    SET distance = TRIM('km' FROM distance);

    ALTER TABLE pizza_runner.runner_orders
    RENAME COLUMN distance to distance_in_km;

    ALTER TABLE pizza_runner.runner_orders
    ALTER COLUMN distance_in_km TYPE NUMERIC
    USING distance_in_km::NUMERIC;

    UPDATE pizza_runner.runner_orders
    SET duration = TRIM('minutes' FROM duration);

    ALTER TABLE pizza_runner.runner_orders
    RENAME COLUMN duration to duration_in_mins;

    ALTER TABLE pizza_runner.runner_orders
    ALTER COLUMN duration_in_mins TYPE NUMERIC
    USING duration_in_mins::NUMERIC;

There were several items that needed cleaned within the provided data:
- There were columns in multiple tables that had null values as *null*, NULL, or '' - I updated all of these columns to reflect *null* uniformly
- The values distance and duration columns of the runner_order table both were not formatted the same way (ex. 15km vs 15 kilometers vs 15). To fix this, I trimmed the columns of any version of abbreviations and then renamed the columns to reflect the units (distance_in_km and duration_in_mins)
- A few of the columns were not the correct data type (changed pickup_time to TIMESTAMP, distnace_in_km to NUMERIC, duration_in_mins to NUMERIC)

## 🫑 Case Study Questions

### 🍕 A. Pizza Metrics
#### 1. How many pizzas were ordered?

    SELECT COUNT(pizza_id) AS num_pizzas_ordered
    FROM pizza_runner.customer_orders;

| num_pizzas_ordered |
| ------------------ |
| 14                 |

Steps:
- Counted the number of pizza ids

Insights:
- There were 14 pizzas ordered through the runner program

---
#### 2. How many unique customer orders were made?

    SELECT COUNT(DISTINCT order_id) AS num_orders
    FROM pizza_runner.customer_orders;

| num_orders |
| ---------- |
| 10         |

Steps:
- Counted the distinct order ids

Insights:
- There were 10 total orders made through the runner program

---
#### 3. How many successful orders were delivered by each runner?

    SELECT runner_id,
    	COUNT(order_id) AS successful_deliveries
    FROM pizza_runner.runner_orders
    WHERE pickup_time IS NOT NULL
    GROUP BY runner_id
    ORDER BY runner_id;

| runner_id | successful_deliveries |
| --------- | --------------------- |
| 1         | 4                     |
| 2         | 3                     |
| 3         | 1                     |

Steps:
- Counted the order ids
- Grouped by the runner id to show the number of deliveries per runner
- Filtered for orderes that were not cancelled

Insights:
- There is no real pattern for how many orders runners are taking
- Runner 1 had the most successful deliveries
- Runner 3 had the least successful deliveries
---
#### 4. How many of each type of pizza was delivered?

    SELECT c.pizza_id,
    	COUNT(c.pizza_id) AS num_delivered
    FROM pizza_runner.customer_orders AS c
    JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL
    GROUP BY c.pizza_id
    ORDER BY c.pizza_id;

| pizza_id | num_delivered |
| -------- | ------------- |
| 1        | 9             |
| 2        | 3             |

Steps:
- Joined the customer_orders and the runner_orders tables in order to find which pizzas were ordered and if they were successfully delivered
- Counted the pizza ids
- Grouped by pizza_id to see how many of each pizza was delivered
- Filtered for pizzas that were not cancelled

Insights:
- Looking at the rest of the data shows that pizza_id 1 is Meatlovers and pizza_id 2 is Vegetarian
- pizza_id 1 (Meatlovers) is the more popular pizza
---
#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

    SELECT c.customer_id,
    	n.pizza_name,
        COUNT(n.pizza_name) AS num_ordered
    FROM pizza_runner.customer_orders AS c
    JOIN pizza_runner.pizza_names AS n
    ON c.pizza_id = n.pizza_id
    GROUP BY c.customer_id,
    	n.pizza_name
    ORDER BY c.customer_id;

| customer_id | pizza_name | num_ordered |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

Steps:
- Joined the customer_orders and the pizza_names tables to see which pizzas were ordered and what their names are
- Counted the pizza names
- Grouped by the customer id and pizza name to see how many of each pizza was ordered for each customer

Insights:
- Most customers have tried both pizzas
- All but one customer prefers the Meatlovers pizza

---
#### 6. What was the maximum number of pizzas delivered in a single order?

    SELECT c.order_id,
    	COUNT(c.pizza_id) AS max_delivered
    FROM pizza_runner.customer_orders AS c
    JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL
    GROUP BY c.order_id
    ORDER BY max_delivered DESC
    LIMIT 1;

| order_id | max_delivered |
| -------- | ------------- |
| 4        | 3             |

Steps:
- Joined the customer_orders and the runner_orders tables to see the pizzas in each order and whether or not they were delivered successfully
- Counted the pizza ids
- Grouped by the order id to see how many pizzas were in each order
- Filtered for only successful deliveries
- Ordered by the count in descending so that the order with the most pizzas was at the top
- Limited the results to 1 to show the maximum number delivered

Insights:
- Orders through the pizza runner program all have 3 or fewer pizzas per order

---
#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

    WITH pizzas AS (
    	SELECT c.customer_id,
    		CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL
    			THEN 'Yes'
            	ELSE 'No'
            	END AS changes
    	FROM pizza_runner.customer_orders AS c
    	JOIN pizza_runner.runner_orders AS r
    	ON c. order_id = r.order_id
    	WHERE r.pickup_time IS NOT NULL)
    
    SELECT customer_id,
    	changes,
    	COUNT(changes) AS num_pizzas
    FROM pizzas
    GROUP BY customer_id, changes
    ORDER BY customer_id;

| customer_id | changes | num_pizzas |
| ----------- | ------- | ---------- |
| 101         | No      | 2          |
| 102         | No      | 3          |
| 103         | Yes     | 3          |
| 104         | Yes     | 2          |
| 104         | No      | 1          |
| 105         | Yes     | 1          |

Steps:
- Created a CTE called pizzas
- Joined the customer_orders and the runner_orders tables to see the changes that were made as well as successful deliveries
- Used a CASE statement to denote wheather a pizza had changes or not (any changes in the extras OR exclusions columns = 'Yes' and pizzas with no changes in either column = 'No')
- Filtered for successful deliveries
- Using the pizzas CTE, I counted the pizzas that had any changes
- Grouped by the customer id to see the changes made for each individual customer

Insights:
- 3 customers made changes while 3 did not
- Only customer 104 ordered pizzas with changes and without
- Half of the pizzas ordered had changes
  
---
#### 8. How many pizzas were delivered that had both exclusions and extras?

    WITH pizzas AS (
    	SELECT c.customer_id,
    		CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL
    			THEN 'Yes'
            	ELSE 'No'
            	END AS changes
    	FROM pizza_runner.customer_orders AS c
    	JOIN pizza_runner.runner_orders AS r
    	ON c. order_id = r.order_id
    	WHERE r.pickup_time IS NOT NULL)
    
    SELECT COUNT(changes) AS num_pizzas
    FROM pizzas
    WHERE changes = 'Yes';

| num_pizzas |
| ---------- |
| 1          |

Steps:
- I used the same CTE as the problem above, except I changed the CASE statement to say AND instead of OR so that both the extras and the exclusions columns had to have changes
- Using the pizzas CTE, I counted the number of pizzas in the changes column
- Filtered for only pizzas where changes = 'Yes' (had both exlusions and extras)

Insights:
- Only 1 pizza was ordered that had both exlusions and extras

---
#### 9. What was the total volume of pizzas ordered for each hour of the day?

    SELECT EXTRACT(hour FROM order_time) AS hour,
    	COUNT(order_id) AS vol_pizzas
    FROM pizza_runner.customer_orders
    GROUP BY hour
    ORDER BY hour;

| hour | vol_pizzas |
| ---- | ---------- |
| 11   | 1          |
| 13   | 3          |
| 18   | 3          |
| 19   | 1          |
| 21   | 3          |
| 23   | 3          |

Steps:
- Extracted the hour from the order time
- Counted the orders
- Grouped by hour

Insights:
- 2:00 PM, 6:00 PM, 9:00 PM, and 11:00 PM are the hours with the highest volume with 3 orders each
- All other hours had 0-1 orders each
---
#### 10. What was the volume of orders for each day of the week?

    WITH dow AS (
      SELECT order_id,
      	EXTRACT(dow FROM order_time) AS num_day
      FROM pizza_runner.customer_orders)
    
    SELECT num_day,
    	CASE WHEN num_day = 0
    		THEN 'Sunday'
            WHEN num_day = 1
            THEN 'Monday'
            WHEN num_day = 2
            THEN 'Tuesday'
            WHEN num_day = 3
            THEN 'Wednesday'
            WHEN num_day = 4
            THEN 'Thursday'
            WHEN num_day = 5
            THEN 'Friday'
            Else 'Saturday' END AS day_of_week,
    	COUNT(order_id) AS num_pizzas
    FROM dow
    GROUP BY day_of_week, num_day
    ORDER BY num_day;

| num_day | day_of_week | num_pizzas |
| ------- | ----------- | ---------- |
| 3       | Wednesday   | 5          |
| 4       | Thursday    | 3          |
| 5       | Friday      | 1          |
| 6       | Saturday    | 5          |

Steps:
- Created a CTE called dow
- Used EXTRACT() to get the day of the week from the order time
- Used the dow CTE to create a CASE statement where I gave each dow number a name so that the table can be more easily interpreted by stakeholders
- Counted the orders
- Grouped by the day of week to see the number of pizzas ordered per day

Insights:
- Wednesdays and Saturdays are the busiest days for the pizza runner program
- Friday is the slowest day for the pizza runner program

---

### 🏃 B. Runner and Customer Experience
#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

    SELECT runner_id,
    	EXTRACT(WEEK FROM registration_date)
    FROM pizza_runner.runners;

| runner_id | date_part |
| --------- | --------- |
| 1         | 53        |
| 2         | 53        |
| 3         | 1         |
| 4         | 2         |


    SELECT EXTRACT(WEEK FROM registration_date + 3) AS week,
    	COUNT(runner_id) AS runners_registered
    FROM pizza_runner.runners
    GROUP BY week
    ORDER BY week;

| week | runners_registered |
| ---- | ------------------ |
| 1    | 2                  |
| 2    | 1                  |
| 3    | 1                  |

Steps:
- When I extrcted the week from the registration_date, I noticed that I got two weeks that said 53, which didn't make sense because I knew all of the orders that were tracked were at the beginning of the year
- Looking at the calendar, 2021-01-01 was on a Friday, so it looks like SQL included those early January dates with the last days of December for a week 53
- To fix this, I extracted the week from the registration date and added 3 days so that 2021-01-01 started at the beginning of week 1
- Counted the runner ids
- Grouped by week

Insights:
- Two runners registered in the first week
- One runner registered each week after the first week
---
#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

    SELECT r.runner_id,
    	EXTRACT(minute FROM AVG(r.pickup_time - c.order_time)) AS avg_mins_for_pickup
    FROM pizza_runner.customer_orders AS c
    JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
    GROUP BY r.runner_id
    ORDER BY r.runner_id;

| runner_id | avg_mins_for_pickup |
| --------- | ------------------- |
| 1         | 15                  |
| 2         | 23                  |
| 3         | 10                  |

❗ There is no actual arrival time listed for the runners so we can't know if the pizzas were finished and waiting to be picked up when the runner arrived, or if the runners arrived before the pizzas were finished

Steps:
- Joined the customer orders and the runner orders tables so that I could have the runner ids, pickup times, and order times
- Took the average of the pickup time minus the order time and then extracted the minutes from that time
- Grouped by the runner id

Insights:
- Runner 3 averaged the fastest times, while runner 2 averaged the slowest times
- This doesn't show the whole picture because we don't know how long the pizzas took to prepare or what time the runner showed up. It is possible that runner 2 happened to get orders that took longer to prepare
---
#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

    WITH pizzas AS (
      SELECT order_id,
      	order_time,
      	COUNT(pizza_id) AS num_pizzas
      FROM pizza_runner.customer_orders
      GROUP BY order_id, order_time)
    
    SELECT p.num_pizzas,
    	AVG(EXTRACT(MINUTE FROM r.pickup_time - p.order_time)) AS avg_time_in_mins
    FROM pizzas AS p
    JOIN pizza_runner.runner_orders AS r
    ON p.order_id = r.order_id
    GROUP BY p.num_pizzas
    ORDER BY p.num_pizzas;

| num_pizzas | avg_time_in_mins |
| ---------- | ---------------- |
| 1          | 12               |
| 2          | 18               |
| 3          | 29               |

❗ We have the pickup time, but we can't know for sure if that is how long the pizzas took to prepare or if the runner showed up after the pizzas were finished

Steps:
- Creted a CTE where I counted the pizza ids to find the number of pizzas
- Grouped by the order id and order time to show how many pizzas were in each order
- Joined the CTE to the runner orders table to get the pickup time
- I extracted the minutes from the pickup time minus the order time and found the average to find the average amount of time it took to prepare the pizzas
- Grouped by the number of pizzas in the order

Insights:
- It is clear that the more pizzas there are in an order, the longer the order will take to prepare
- There is a larger jump in time when going from 2 to 3 pizzas than there is when going from 1 to 2 pizzas
---
#### 4. What was the average distance travelled for each customer?

    SELECT c.customer_id,
    	ROUND(AVG(r.distance_in_km), 1) AS avg_distance_in_km
    FROM pizza_runner.customer_orders AS c
    JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
    GROUP BY c.customer_id
    ORDER BY c.customer_id;

| customer_id | avg_distance_in_km |
| ----------- | ------------------ |
| 101         | 20.0               |
| 102         | 16.7               |
| 103         | 23.4               |
| 104         | 10.0               |
| 105         | 25.0               |

Steps:
- Joined the customer orders and the runner orders tables to get the customer ids and the distnace travelled
- Found the average of the distance in km and rounded that number to 1 decimal place
- Grouped by customer id

Insights:
- Runners are travelling anywhere between 10 to 25 km depending on the customer

---
#### 5. What was the difference between the longest and shortest delivery times for all orders?

    SELECT MAX(duration_in_mins) - MIN(duration_in_mins) AS difference_in_mins
    FROM pizza_runner.runner_orders;

| difference_in_mins |
| ------------------ |
| 30                 |

Steps:
- Took the max(or longest) duration of delivery and subtrated the min(or shortest) duration of delivery

Insights:
- There is a 30 minute difference between the shortest and longest delivery times

---
#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

    SELECT runner_id,
    	order_id,
        distance_in_km,
        ROUND((distance_in_km / (duration_in_mins / 60)),1) AS avg_kph
    FROM pizza_runner.runner_orders
    WHERE distance_in_km IS NOT NULL
    ORDER BY runner_id, order_id;

| runner_id | order_id | distance_in_km | avg_kph |
| --------- | -------- | -------------- | ------- |
| 1         | 1        | 20             | 37.5    |
| 1         | 2        | 20             | 44.4    |
| 1         | 3        | 13.4           | 40.2    |
| 1         | 10       | 10             | 60.0    |
| 2         | 4        | 23.4           | 35.1    |
| 2         | 7        | 25             | 60.0    |
| 2         | 8        | 23.4           | 93.6    |
| 3         | 5        | 10             | 40.0    |

Steps:
- Took the distnace in km divided that by the duration in mins/60 (to get the number in hours)
- Rounded that number to 1 decimal place
- Filtered for orders that were successfully delivered

Insights:
- Most speeds look to be staying between 35-60 kph
- Runner 2 has a large vaiance in speeds, with one order averaging 93.6 kph

---
#### 7. What is the successful delivery percentage for each runner?

    WITH counts AS (
    	SELECT runner_id,
    		COUNT(order_id) AS orders,
        	COUNT(pickup_time) AS success
    	FROM pizza_runner.runner_orders
    	GROUP BY runner_id
    	ORDER BY runner_id)
        
    SELECT runner_id,
    	TRUNC((success::numeric / orders::numeric) * 100, 0) AS percent_success
    FROM counts
    ORDER BY runner_id;

| runner_id | percent_success |
| --------- | --------------- |
| 1         | 100             |
| 2         | 75              |
| 3         | 50              |

Steps:
- Created a CTE called counts
- Counted the number of orders
- Counted the pickup times to find the number of successful orders
- Grouped by the runner id
- Used the CTE to take the successes divided by the number of orders and the multiplied by 100 to find the percentage
- I set the success and orders to numeric data type because when using INT, anything between 0 and 1 came back as 0
- Truncated the percentage to the whole number

Insights:
- Only runner 1 had a 100% success rate
- Reasons for lower success rates should be investigated

---

### 🧄 C. Ingredient Optimisation

📢 This section has a lot of steps so stick with me! It took some trial and error, but I enjoyed figuring it out and loved seeing the results come out how I wanted.

#### 1. What are the standard ingredients for each pizza?

    WITH unnested AS (
          SELECT pizza_id,
          	(UNNEST(STRING_TO_ARRAY(toppings, ',')))::INT AS topping_id
          FROM pizza_runner.pizza_recipes)
        
    SELECT n.pizza_name,
        STRING_AGG(t.topping_name, ', ') AS toppings
    FROM unnested AS u
    JOIN pizza_runner.pizza_names AS n
        ON u.pizza_id = n.pizza_id
    JOIN pizza_runner.pizza_toppings AS t
        ON u.topping_id = t.topping_id
    GROUP BY n.pizza_name
    ORDER BY n.pizza_name;

| pizza_name | toppings                                                              |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

Steps:
- Created a CTE to fix the toppings column of the pizza recipes table
- Used STRING_TO_ARRAY to separate all of the topping ids by the ',' delimiter
- Unnested the toppings so that each topping moved to its own row
- Changed the topping ids from VARCHAR type to INT data type so that I would be able to join them with the topping ids from the pizza toppings table
- Joined the CTE to the pizza names table to get the names of the pizzas
- Joined the CTE to the pizza toppings table to get the topping names
- Took the topping names and used STRING_AGG to combine them into one row using the ', ' delimiter
- Grouped by the pizza name

Insights:
- This table just lists the two pizzas and what toppings come standard on each of them

---
#### 2. What was the most commonly added extra?

    WITH unnested AS (
      SELECT order_id,
      	pizza_id,
      	(UNNEST(STRING_TO_ARRAY(extras, ',')))::INT AS extras
      FROM pizza_runner.customer_orders)
    
    SELECT t.topping_name,
    	COUNT(t.topping_name) AS times_added
    FROM pizza_runner.customer_orders AS c
    LEFT JOIN unnested AS u
    ON c.order_id = u.order_id
    JOIN pizza_runner.pizza_toppings AS t
    ON u.extras = t.topping_id
    GROUP BY t.topping_name
    ORDER BY times_added DESC
    LIMIT 1;

| topping_name | times_added |
| ------------ | ----------- |
| Bacon        | 5           |

Steps:
- Created a CTE to fix the extras column
- Used STRING_TO_ARRAY to separate the extras by the ',' delimiter
- Unnested the extras so that each topping was moved to its own row
- Changed the data type to INT so I would be able to join it with the pizza toppings table
- Took the customer orders table and left joined the CTE so that I kept all orders
- Joined the pizza toppings table to the CTE to get the topping names
- Counted the topping names
- Grouped by the topping name
- Ordered by the times the topping was added in descending order so that the highest number was at the top
- Limited the results to 1 to show just the most commonly added extra

Insights:
- Bacon is the most commonly added extra
- It was added 5 times

---
#### 3. What was the most common exclusion? 

    WITH row_number AS (
      SELECT ROW_NUMBER() OVER(
        ORDER BY order_id) AS num_pizza,
      *
      FROM pizza_runner.customer_orders)
      
    , unnested AS (
      SELECT num_pizza,
      	order_id,
      	pizza_id,
      	(UNNEST(STRING_TO_ARRAY(exclusions, ',')))::INT AS exclusions
      FROM row_number)
    
    
    SELECT t.topping_name,
    	COUNT(t.topping_name) AS times_excluded
    FROM unnested AS u
    JOIN pizza_runner.pizza_toppings AS t
    ON u.exclusions = t.topping_id
    GROUP BY t.topping_name
    ORDER BY times_excluded DESC
    LIMIT 1;

| topping_name | times_excluded |
| ------------ | -------------- |
| Cheese       | 4              |


Steps:
- I used the same code as the question above, except I changed extras to exclusions
  - When I did this, I noticed that the number was wrong because I was getting duplicates. This was happening because there was one order that ordered two pizzas that were the same type, and both excluded cheese
- To fix this, I created a CTE to number the individual pizzas ordered
- Used that CTE to separate and unnest the exclusions
- Changed the exlusions to INT data type
- Joined the CTE to the pizza toppings table to get the topping names
- Counted the pizza toppings
- Grouped by pizza toppings to get the number of times each individual topping was counted
- Ordered by the times excluded in descending order so that the most excluded item was listed first
- Limited to 1 so that it only showed the most exlcluded topping

Insights:
- Cheese was the most commonly exluded topping

---

#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
#### - Meat Lovers
#### - Meat Lovers - Exclude Beef
#### - Meat Lovers - Extra Bacon
#### - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

    WITH row_number AS (
      SELECT ROW_NUMBER() OVER(
        ORDER BY order_id) AS num_pizza,
      *
      FROM pizza_runner.customer_orders)
    
    
    , extras AS (
    SELECT c.num_pizza,
      	c.order_id,
    	c.pizza_id,
      	c.extras AS e,
        STRING_AGG(t.topping_name, ', ') AS extras
    FROM (
    	SELECT *,
    		UNNEST(STRING_TO_ARRAY(extras, ', '))::INT
    	FROM row_number) AS c
    JOIN pizza_runner.pizza_toppings AS t
    ON c.unnest = t.topping_id
    WHERE c.extras IS NOT NULL
    GROUP BY c.num_pizza,
      	c.order_id, 
    	c.pizza_id, 
      	c.extras
    ORDER BY c.order_id)
    
    , exclusions AS (
    SELECT c.num_pizza,
    	c.order_id,
    	c.pizza_id,
      	c.exclusions AS e,
        STRING_AGG(t.topping_name, ', ') AS exclusions
    FROM (
      	SELECT *,
    		UNNEST(STRING_TO_ARRAY(exclusions, ', '))::INT
    	FROM row_number) AS c
    JOIN pizza_runner.pizza_toppings AS t
    ON c.unnest = t.topping_id
    WHERE c.exclusions IS NOT NULL
    GROUP BY c.num_pizza,
    	c.order_id,
    	c.pizza_id,
      	c.exclusions
    ORDER BY c.order_id)
    
    SELECT c.order_id,
    	CONCAT(n.pizza_name,
       		COALESCE(' - Extra ' || xt.extras, ''), 
        	COALESCE(' - Exclude ' || ex.exclusions, '')) AS pizza_order
    FROM row_number AS c
    LEFT JOIN extras AS xt
    ON c.num_pizza = xt.num_pizza AND c.order_id = xt.order_id AND c.pizza_id = xt.pizza_id AND c.extras = xt.e
    LEFT JOIN exclusions AS ex
    ON c.num_pizza = ex.num_pizza AND c.order_id = ex.order_id AND c.pizza_id = ex.pizza_id AND c.exclusions = ex.e
    JOIN pizza_runner.pizza_names AS n
    ON c.pizza_id = n.pizza_id;

| order_id | pizza_order                                                     |
| -------- | --------------------------------------------------------------- |
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 3        | Meatlovers                                                      |
| 4        | Vegetarian - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Extra Bacon, Chicken - Exclude Cheese              |
| 10       | Meatlovers                                                      |
| 10       | Meatlovers - Extra Bacon, Cheese - Exclude BBQ Sauce, Mushrooms |

Steps:
- CTE #1:
    - Numbered the rows and ordered by the order number so that each pizza ordered had a number and avoid future duplicates
- CTE #2:
    - Created a subquery in the FROM clause where I used STRING_TO_ARRAY and unnested so that the extras were separated. I also changed the data type to INT in this subquery
    - Joined the subquery to the pizza toppings table to get the topping names
    - Made sure to carry the row numbers through
    - Used STRING_AGG to create the list of extras for each pizza
    - Grouped by all non aggregated columns selected
- CTE #3:
    - Used the same code as the previous CTE, just changed extras to exlusions
- Final Query:
    - Used the first CTE where the rows were numbered and left joined the extras CTE on all appropriate columns
    - Left joined the exlusions CTE on all appropriate columns
    - Joined the pizza names table
    - Concatenated ' - Extra ' before the listed extras and ' - Exclude ' before the listed exclusions
    - Used COALESCE so that the text only appeared before the non null values
    - Used CONCAT to put that together with the pizza names

Insights:
- This is a table that is created for the restaurant to more easily read the orders
---

#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
#### - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

    WITH numbered AS (
      SELECT ROW_NUMBER() OVER(
      	ORDER BY order_id) AS num_pizza,
      	*
      FROM pizza_runner.customer_orders)

    
    , unnested AS (
    	SELECT num_pizza,
    		order_id,
        	pizza_id,
      		exclusions,
      		extras,
        	UNNEST(STRING_TO_ARRAY(exclusions, ','))::INT AS exclude,
        	UNNEST(STRING_TO_ARRAY(extras, ','))::INT AS add
    	FROM numbered)
        
      
    , exclusions AS (
      SELECT u.num_pizza,
      	u.order_id,
      	n.pizza_id,
      	STRING_AGG(t.topping_name, ', ') AS exclusions
      FROM unnested AS u
      JOIN pizza_runner.pizza_names AS n
      ON u.pizza_id = n.pizza_id
      JOIN pizza_runner.pizza_toppings AS t
      ON u.exclude = t.topping_id
      GROUP BY u.num_pizza,
      	u.order_id,
      	n.pizza_id
      ORDER BY u.order_id)
    
    , extras AS (
      SELECT u.num_pizza,
      	u.order_id,
      	n.pizza_id,
      	STRING_AGG(t.topping_name, ', ') AS extras
      FROM unnested AS u
     JOIN pizza_runner.pizza_names AS n
      ON u.pizza_id = n.pizza_id
      JOIN pizza_runner.pizza_toppings AS t
      ON u.add = t.topping_id
      GROUP BY u.num_pizza,
      	u.order_id,
      	n.pizza_id
      ORDER BY u.order_id)
      
    , all_orders AS (
      SELECT n.num_pizza,
      	n.order_id,
      	n.pizza_id,
      	ex.exclusions,
      	xt.extras
      FROM numbered AS n
      LEFT JOIN exclusions AS ex
      ON n.num_pizza = ex.num_pizza AND n.order_id = ex.order_id AND n.pizza_id = ex.pizza_id
      LEFT JOIN extras AS xt
      ON n.num_pizza = xt.num_pizza AND n.order_id = xt.order_id AND n.pizza_id = xt.pizza_id)
      
    , recipes AS (
      SELECT r.pizza_id,
      	n.pizza_name,
      	UNNEST(STRING_TO_ARRAY(r.toppings, ',')):: INT AS toppings
      FROM pizza_runner.pizza_recipes AS r
      JOIN pizza_runner.pizza_names AS n
      ON r.pizza_id = n.pizza_id)
      
     , all_ingredients AS (
       SELECT o.num_pizza,
       	o.order_id,
       	o.pizza_id,
       	r.pizza_name,
       	o.exclusions,
       	o.extras,
       	r.toppings
       FROM all_orders AS o
       JOIN recipes AS r
       ON o.pizza_id = r.pizza_id
       ORDER BY o.num_pizza)
       
    , names AS (
      SELECT a.num_pizza,
      	a.order_id,
      	a.pizza_name,
      	a.exclusions,
      	a.extras,
      	t.topping_name
      FROM all_ingredients AS a
      JOIN pizza_runner.pizza_toppings AS t
      ON a.toppings = t.topping_id
      ORDER BY a.num_pizza,
      		a.order_id,
    		t.topping_name)
    
    , exclusions_and_extras AS (
      SELECT num_pizza,
      	order_id,
      	pizza_name,
      	exclusions,
      	extras,
      	topping_name,
      	CASE WHEN exclusions LIKE ('%' || topping_name || '%')
      		THEN NULL
      		WHEN exclusions IS NULL
      		THEN 'X'
      		ELSE 'X' END AS exclude,
      	CASE WHEN extras LIKE ('%' || topping_name || '%')
      		THEN ('2x' || topping_name)
      		WHEN extras IS NULL
      		THEN topping_name
      		ELSE topping_name END AS add
      FROM names)
    
    
    , ingredients AS (
      SELECT *,
      	CASE WHEN exclude IS NULL
      		THEN NULL
      		ELSE add END AS ingredients
      FROM exclusions_and_extras)
    
    SELECT order_id,
    	CONCAT(pizza_name, ': ',
        STRING_AGG(ingredients, ', ')) AS pizza_recipe
    FROM ingredients
    GROUP BY num_pizza,
    	order_id,
    	pizza_name
    ORDER BY num_pizza;

| order_id | pizza_recipe                                                                        |
| -------- | ----------------------------------------------------------------------------------- |
| 1        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 3        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                      |
| 4        | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 5        | Meatlovers: BBQ Sauce, 2xBacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 7        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 8        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | Meatlovers: BBQ Sauce, 2xBacon, Beef, 2xChicken, Mushrooms, Pepperoni, Salami       |
| 10       | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 10       | Meatlovers: 2xBacon, Beef, 2xCheese, Chicken, Pepperoni, Salami                     |

Steps:
- CTE #1:
    - Used the same code as the previous question to number the rows
- CTE #2:
    - Instead of using subqueries, this time I used a CTE to separate and unnest the extras and exlusions into their own rows
    - Changed the data types to INT
- CTE #3:
    - Used the same code from the previous problem, except I joined the CTE instead of the subquery
- CTE #4:
    - Used the same code from the previous problem, except I joined the CTE instead of the subquery
- CTE #5:
    - This was to bring together all pizzas ordered with their exlusions and extras
    - Used the first CTE and left joined the exclusions table on all appropriate columns
    - Left joined the extras table on all appropriate columns
- CTE #6:
    - Used the code from question #1 to separate and unnest all of the toppings
- CTE #7:
    - Used the code from question #1 to get all of the topping names
    - Did not used STRING_AGG like in question #1 because I wanted to keep them all separated for now
- CTE #8:
    - Joined the previous CTE to the pizza toppings table to get all of the topping names
    - Included the topping name in the ORDER BY statement so that they were ordered alphabetically
    - Ended up with a list of every pizza ordered, all toppings included with the pizza, and any exclusions or extras
- CTE #9:
    - Used the previous CTE
    - Created a CASE statement. For exclusions, if the text in the exlusions column matched any part of the toppings column, then it would be left null. Otherwise, there would be 'X'
    - For extras, if the text in the extras column matched any part of the toppings column, then it would concatenate '2x' with the topping name. Otherwise, it would just show the topping name
- CTE #10:
    - Used the previous CTE
    - Created a CASE statement that combined the two previous CASE statements
    - If exlusions was null, then it would be null
    - Otherwise, it would show what as in the extras column
- Final Query:
    - Used STRING_AGG to put all ingredients into one row separated by the ', ' delimiter
    - Used CONCAT to add the pizza name and ': ' before the list of ingredients
    - Made sure to group by the row number, order id, and pizza id to make sure that there was one row for each pizza ordered

Insights:
- This is a table created to show a readable recipe of ingredients for each pizza ordered
---
#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

    WITH numbered AS (
      SELECT ROW_NUMBER() OVER(
      	ORDER BY c.order_id) AS num_pizza,
      	c.*
      FROM pizza_runner.customer_orders AS c
      JOIN pizza_runner.runner_orders AS r
    	ON c.order_id = r.order_id
      WHERE r.pickup_time IS NOT NULL)
    
    , unnested AS (
    	SELECT num_pizza,
    		order_id,
        	pizza_id,
      		exclusions,
      		extras,
        	UNNEST(STRING_TO_ARRAY(exclusions, ','))::INT AS exclude,
        	UNNEST(STRING_TO_ARRAY(extras, ','))::INT AS add
    	FROM numbered)
        
    , exclusions AS (
      SELECT t.topping_name AS ingredient,
      	COUNT(t.topping_name) AS times_excluded
      FROM unnested AS u
      JOIN pizza_runner.pizza_toppings AS t
      ON u.exclude = t.topping_id
      GROUP BY ingredient
      ORDER BY times_excluded DESC)
      
    , extras AS (
      SELECT t.topping_name AS ingredient,
      	COUNT(t.topping_name) AS times_used
      FROM unnested AS u
      JOIN pizza_runner.pizza_toppings AS t
      ON u.add = t.topping_id
      GROUP BY ingredient
      ORDER BY times_used DESC)
      
    , recipes AS (
      SELECT r.pizza_id,
      	n.pizza_name,
      	UNNEST(STRING_TO_ARRAY(r.toppings, ',')):: INT AS toppings
      FROM pizza_runner.pizza_recipes AS r
      JOIN pizza_runner.pizza_names AS n
      ON r.pizza_id = n.pizza_id)
    
    , counts AS (
    	SELECT t.topping_name,
    		COUNT(t.topping_name) AS topping_count
    	FROM numbered AS n
    	JOIN recipes AS r
    	ON n.pizza_id = r.pizza_id
    	JOIN pizza_runner.pizza_toppings AS t
    	ON r.toppings = t.topping_id
    	GROUP BY t.topping_name
    	ORDER BY topping_count DESC)
      
    
    , cleaned AS (
    	SELECT c.topping_name AS ingredient,
    		CASE WHEN c.topping_count IS NULL
        		THEN 0
            	ELSE c.topping_count END AS topping_count,
        	CASE WHEN ex.times_excluded IS NULL
        		THEN 0
            	ELSE ex.times_excluded END AS times_excluded,
        	CASE WHEN xt.times_used IS NULL
        		THEN 0
            	ELSE xt.times_used END AS times_added
    	FROM counts AS c
    	LEFT JOIN exclusions AS ex
    	ON c.topping_name = ex.ingredient
    	LEFT JOIN extras as xt
    	ON c.topping_name = xt.ingredient)
    
    SELECT ingredient,
      (topping_count - times_excluded + times_added) AS times_used
    FROM cleaned
    ORDER BY times_used DESC;

| ingredient   | times_used |
| ------------ | ---------- |
| Bacon        | 12         |
| Mushrooms    | 11         |
| Cheese       | 10         |
| Pepperoni    | 9          |
| Salami       | 9          |
| Chicken      | 9          |
| Beef         | 9          |
| BBQ Sauce    | 8          |
| Tomatoes     | 3          |
| Onions       | 3          |
| Peppers      | 3          |
| Tomato Sauce | 3          |


Steps:
- CTE #1:
    - Used the same CTE as above, except joined the runners table and filtered for only delivered pizzas
- CTE #2:
    - Used the same CTE as above to separate all of the extras and exclusions and separate them into their own rows
- CTE #3:
    - Joined the previous CTE to the pizza toppings table
    - Counted the number of times each topping was excluded
    - Grouped by the ingredient
- CTE #4:
    - Used the same code as the previous CTE, but changed to extas where applicable
- CTE #5:
    - Used the same code as previous questions to ungroup and unnest the pizza toppings used on each pizza
- CTE #6:
    - Joined the numbered table to the recipes table to get the order numbers, pizzas ordered, and the toppings that are included on each pizza
    - Joined the pizza toppings table to get the names of the toppings
    - Counted the topping names
    - Grouped by the topping name to see how many times each topping was used
- CTE #7:
    - Used the previous CTE and left joined the exclusions and extras tables to make sure I kept all pizzas ordered
    - Used CASE statements to change any *null* values to 0 so that they could be used in arethmetic
- Final Query:
    - Took the times the toppings were used in a standard recipe, subtracted the times the topping was exluded, and added the times the topping was added as an extra
    - Ordered the results in descending order so that the most frequently used topping was listed first

Insights:
- Bacon is the most used topping
- Everything besides tomatoes, onions, peppers, and tomato sauce was used between 8-12 times
- Tomatoes, onions, peppers, and tomato sauce were used only 3 times each
  
---

### 💰 D. Pricing and Ratings

#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

    WITH prices AS (
      SELECT order_id,
      	pizza_id,
      	CASE WHEN pizza_id = 1 THEN 12
      	ELSE 10 END AS price
      FROM pizza_runner.customer_orders)
    
    
    SELECT SUM(p.price) AS dollars
    FROM prices AS p
    JOIN pizza_runner.runner_orders AS r
    	ON p.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL;

| dollars |
| ------- |
| 138     |

Steps:
- Created a CTE called prices
- Used a CASE statement to set the price of each pizza offered
- Joined the CTE to the runner orders table
- Found the sum of the prices
- Filtered for only successful deliveries

Insights:
- The restaurant made $138 on pizzas ordered through the runner program with standard pricing

---
#### 2. What if there was an additional $1 charge for any pizza extras?

    WITH prices AS (
      SELECT order_id,
      	pizza_id,
      	CASE WHEN pizza_id = 1 THEN 12
      		ELSE 10 END AS pizza_price,
      	CASE WHEN extras LIKE '_' THEN 1
      		WHEN extras LIKE '____' THEN 2
      		WHEN extras LIKE '______' THEN 3
      		ELSE 0 END AS extras_price
      FROM pizza_runner.customer_orders)
      
    SELECT (SUM(p.pizza_price) + SUM(p.extras_price)) AS dollars
    FROM prices AS p
    JOIN pizza_runner.runner_orders AS r
    	ON p.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL;

| dollars |
| ------- |
| 142     |

Steps:
- Created a CTE called prices
- Created a CASE statement where I set the price for each of the pizzas offered
- Created a second CASE statemenet that stated if there was one extra, then it is 1 dollar extra, if there were 4 characters (or 2 extras), then it is 2 dollars extra, and so on
- Joined the CTE to the runner orders table
- Found the sum of the standard pizza prices and added it to the sum of the add ons
- Filtered for only successful deliveries

Insights:
- The restaurant made $142 on orders made through the runner program with extras added on
  
---
#### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

    CREATE TABLE ratings AS
    SELECT DISTINCT (c.order_id),
    	c.customer_id,
        r.runner_id
    FROM pizza_runner.runner_orders AS r
    LEFT JOIN pizza_runner.customer_orders AS c
    	ON r.order_id = c.order_id
    WHERE r.cancellation IS NULL;

    ALTER TABLE ratings
    ADD rating INTEGER;

    UPDATE ratings
    SET rating = 4 
    WHERE order_id = 1;

    UPDATE ratings
    SET rating = 5 
    WHERE order_id = 2;

    UPDATE ratings
    SET rating = 5 
    WHERE order_id = 3;

    UPDATE ratings
    SET rating = 3 
    WHERE order_id = 4;

    UPDATE ratings
    SET rating = 5 
    WHERE order_id = 5;

    UPDATE ratings
    SET rating = 4
    WHERE order_id = 7;

    UPDATE ratings
    SET rating = 5 
    WHERE order_id = 8;

    UPDATE ratings
    SET rating = 5 
    WHERE order_id = 10;

    SELECT *
    FROM ratings;

| order_id | customer_id | runner_id | rating |
| -------- | ----------- | --------- | ------ |
| 1        | 101         | 1         | 4      |
| 2        | 101         | 1         | 5      |
| 3        | 102         | 1         | 5      |
| 4        | 103         | 2         | 3      |
| 5        | 104         | 3         | 5      |
| 7        | 105         | 2         | 4      |
| 8        | 102         | 2         | 5      |
| 10       | 104         | 1         | 5      |

Steps:
- Created a new table called ratings
- Used the runner orders table and left joined the customer orders table
- Filtered for orders that were not cancelled
- Went through and added a rating for each order

---
#### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
**- order_id**  
**- customer_id**  
**- runner_id**  
**- rating**  
**- order_time**  
**- pickup_time**  
**- time between order and pickup**  
**- delivery duration**  
**- average speed**  
**- total number of pizzas**  

    SELECT rt.*,
    	c.order_time,
        r.pickup_time,
        EXTRACT(minutes FROM (r.pickup_time - c.order_time)) AS mins_btwn_order_and_pickup,
        r.duration_in_mins,
        ROUND((r.distance_in_km / (r.duration_in_mins / 60)),1) AS avg_kph,
        COUNT(c.pizza_id) AS num_pizzas
    FROM ratings AS rt
    JOIN pizza_runner.customer_orders AS c
    	ON rt.order_id = c.order_id
    JOIN pizza_runner.runner_orders AS r
    	ON rt.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL
    GROUP BY rt.order_id,
    	rt.customer_id,
        rt.runner_id,
        rt.rating,
    	c.order_time,
    	r.pickup_time,
        mins_btwn_order_and_pickup,
        r.duration_in_mins,
        avg_kph
    ORDER BY rt.order_id;

| order_id | customer_id | runner_id | rating | order_time               | pickup_time              | mins_btwn_order_and_pickup | duration_in_mins | avg_kph | num_pizzas |
| -------- | ----------- | --------- | ------ | ------------------------ | ------------------------ | -------------------------- | ---------------- | ------- | ---------- |
| 1        | 101         | 1         | 4      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10                         | 32               | 37.5    | 1          |
| 2        | 101         | 1         | 5      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10                         | 27               | 44.4    | 1          |
| 3        | 102         | 1         | 5      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21                         | 20               | 40.2    | 2          |
| 4        | 103         | 2         | 3      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29                         | 40               | 35.1    | 3          |
| 5        | 104         | 3         | 5      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10                         | 15               | 40.0    | 1          |
| 7        | 105         | 2         | 4      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10                         | 25               | 60.0    | 1          |
| 8        | 102         | 2         | 5      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20                         | 15               | 93.6    | 1          |
| 10       | 104         | 1         | 5      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15                         | 10               | 60.0    | 2          |

Steps:
- Joined the ratings table and the customer orders table
- Joined the ratings table and the runner orders table
- Subtracted the order time from the pickup time and extracted the minutes
- Used the same formula that I used before to calulate the avg kph
- Counted the pizzas
- Grouped by all non aggregated fields

Insights:
- This table provides a good overview of the journey of each order placed
  
---
#### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

    WITH prices AS (
      SELECT c.order_id,
      	c.pizza_id,
      	CASE WHEN c.pizza_id = 1 THEN 12
      	ELSE 10 END AS price,
      	ROUND((distance_in_km * 0.30), 2) AS runner_pay
      FROM pizza_runner.customer_orders AS c
    	JOIN pizza_runner.runner_orders AS r
    	ON c.order_id = r.order_id)
        
    SELECT ROUND((SUM(price) - SUM(runner_pay)), 2) AS dollars_left
    FROM prices
    WHERE runner_pay IS NOT NULL;

| dollars_left |
| ------------ |
| 73.38        |

Steps:
- Created a CTE called prices
- Joined the customer orders and the runner orders tables
- Used a CASE statement to set the pizza prices
- Took the distance in km and multiplied by $0.30 to get how much the runner made
- Used the CTE to take the sum of the total price and subtract the sum of the runner pay
- Filtered for successful deliveries

Insights:
- The restaurant had $73.38 left after paying the runners
- Looking back, I can see that almost half the money made from the pizzas went to the runners

---

## 🚀 Final Thoughts

This was a big jump in difficulty from the last case study and I really enjoyed the challenge!  
  
Overall, I don't think there is enough data to draw very many conclusions. It is hard to tell how successful the pizza runner program is when I don't have the regular restaurant data to see how many orders they usually get and compare it to the pizza runner orders. I can tell, however, that if the program is going to expand and take take more orders, the restaurant will need to get more runners to sign up. Out of the 4 runners listed, only 3 have taken any orders and they were the first 3 to sign up. Advertising the program as well as advertising that runners are needed could help keep the program balanced out. The data can also show which ingredients need to have larger quantities on hand, and which aren't as popular. Again, the limited amount of data makes this difficult because the data shows that cheese is the most commonly exluded item, but 3 out of the 4 times it was excluded was from the same customer.  
  
Another thing I noticed is that I can see the pizza runner program still made money after paying the runners, but I don't have any data showing how much overhead there is in making the pizzas, like the employees, ingredients, supplies, etc. Without this data, I do not know if the program actually made money in the end.  
  
Thank you so much for sticking with me on this one!

