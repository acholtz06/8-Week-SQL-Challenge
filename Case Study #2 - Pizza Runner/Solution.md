# 🍕 Case Study #2 - Pizza Runner

![Pizza Runner Img](https://github.com/acholtz06/8-Week-SQL-Challenge/assets/110953602/781423cd-5ac7-4c48-807f-c2678a274182)

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
    ALTER COLUMN duration_in_mins TYPE INT
    USING duration_in_mins::INT;

---
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

