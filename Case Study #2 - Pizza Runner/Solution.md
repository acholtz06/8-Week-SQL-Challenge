# 🍕 Case Study #2 - Pizza Runner

![Pizza Runner Img](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/781423cd-5ac7-4c48-807f-c2678a274182)
###### All data and case study questions were taken from Data With Danny and can be found [here](https://8weeksqlchallenge.com/case-study-2/)

## ✏️ Business Task

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

**Query #23**

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

### 💰 D. Pricing and Ratings

### 🏆 E. Bonus Questions

## 🚀 Final Thoughts
