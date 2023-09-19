# 🍕 Case Study #2 - Pizza Runner

![Pizza Runner Img](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/781423cd-5ac7-4c48-807f-c2678a274182)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-2/)

## ✏️ Business Task

Danny has a pizza restaurant and he wants to expand his business by hiring runners to pick up pizzas from Pizza Runner Headquarters and deliver them right the the customer's home. He would like to take a look at how the runner system is working and see if it is worth expanding the program.

## ⚠️ Data Limitations



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

---
#### 2. How many unique customer orders were made?

    SELECT COUNT(DISTINCT order_id) AS num_orders
    FROM pizza_runner.customer_orders;

| num_orders |
| ---------- |
| 10         |

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

---
#### 5. What was the difference between the longest and shortest delivery times for all orders?

    SELECT MAX(duration_in_mins) - MIN(duration_in_mins) AS difference_in_mins
    FROM pizza_runner.runner_orders;

| difference_in_mins |
| ------------------ |
| 30                 |

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

---

### 🧄 C. Ingredient Optimisation

#### 1. What are the standard ingredients for each pizza?

    WITH unnested AS (
      SELECT pizza_id,
      	(UNNEST(STRING_TO_ARRAY(toppings, ',')))::INT AS topping_id
      FROM pizza_runner.pizza_recipes)
    
    SELECT n.pizza_name,
    	t.topping_name
    FROM unnested AS u
    JOIN pizza_runner.pizza_names AS n
    	ON u.pizza_id = n.pizza_id
    JOIN pizza_runner.pizza_toppings AS t
    	ON u.topping_id = t.topping_id
    ORDER BY n.pizza_name;

| pizza_name | topping_name |
| ---------- | ------------ |
| Meatlovers | BBQ Sauce    |
| Meatlovers | Pepperoni    |
| Meatlovers | Cheese       |
| Meatlovers | Salami       |
| Meatlovers | Chicken      |
| Meatlovers | Bacon        |
| Meatlovers | Mushrooms    |
| Meatlovers | Beef         |
| Vegetarian | Tomato Sauce |
| Vegetarian | Cheese       |
| Vegetarian | Mushrooms    |
| Vegetarian | Onions       |
| Vegetarian | Peppers      |
| Vegetarian | Tomatoes     |

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

---
#### 3. What was the most common exclusion?

    WITH unnested AS (
      SELECT order_id,
      	pizza_id,
      	(UNNEST(STRING_TO_ARRAY(exclusions, ',')))::INT AS exclusions
      FROM pizza_runner.customer_orders)
    
    SELECT t.topping_name,
    	COUNT(t.topping_name) AS times_excluded
    FROM pizza_runner.customer_orders AS c
    LEFT JOIN unnested AS u
    ON c.order_id = u.order_id
    JOIN pizza_runner.pizza_toppings AS t
    ON u.exclusions = t.topping_id
    GROUP BY t.topping_name
    ORDER BY times_excluded DESC
    LIMIT 1;

| topping_name | times_excluded |
| ------------ | -------------- |
| Cheese       | 10             |

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

---
#### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
#### - order_id
#### - customer_id
#### - runner_id
#### - rating
#### - order_time
#### - pickup_time
#### - time between order and pickup
#### - delivery duration
#### - average speed
#### - total number of pizzas

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

---

### 🏆 E. Bonus Questions
#### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

## 🚀 Final Thoughts
