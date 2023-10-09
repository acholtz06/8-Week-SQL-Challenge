# Case Study #5 - Data Mart

**Query #1**

    CREATE TEMPORARY TABLE clean_weekly_sales AS
    SELECT TO_DATE(week_date, 'dd/mm/yy') AS week_date,
    	CASE WHEN EXTRACT(year FROM (TO_DATE(week_date, 'dd/mm/yy'))) = '2018'
        	THEN EXTRACT(week FROM (TO_DATE(week_date, 'dd/mm/yy')))
        	WHEN EXTRACT(year FROM (TO_DATE(week_date, 'dd/mm/yy'))) = '2019'
            THEN EXTRACT(week FROM (TO_DATE(week_date, 'dd/mm/yy')) - INTERVAL '1 day')
            WHEN EXTRACT(year FROM (TO_DATE(week_date, 'dd/mm/yy'))) = '2020'
            THEN EXTRACT(week FROM (TO_DATE(week_date, 'dd/mm/yy')) - INTERVAL '2 days')
            END AS week_number,
        EXTRACT(month FROM (TO_DATE(week_date, 'dd/mm/yy'))) AS month_number,
        EXTRACT(year FROM (TO_DATE(week_date, 'dd/mm/yy'))) AS calendar_year,
        CASE WHEN region <> 'USA' THEN INITCAP(region)
        	ELSE region END AS region,
        platform,
        CASE WHEN segment IS NULL OR segment = 'null' THEN 'Unknown'
        	ELSE segment END AS segment,
        CASE WHEN segment LIKE '%1' THEN 'Young Adults'
        	WHEN segment LIKE '%2' THEN 'Middle Aged'
            WHEN segment LIKE '%3' OR segment LIKE '%4' THEN 'Retirees'
            ELSE 'Unknown' END AS age_band,
        CASE WHEN segment LIKE 'C%' THEN 'Couples'
        	WHEN segment LIKE 'F%' THEN 'Families'
            ELSE 'Unknown' END AS demographic,
        customer_type,
        transactions,
        sales,
        ROUND((sales / transactions), 2) AS avg_transaction
    FROM data_mart.weekly_sales;
  

    SELECT *
    FROM clean_weekly_sales
    LIMIT 10;

| week_date                | week_number | month_number | calendar_year | region | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------ | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Asia   | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.00           |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Asia   | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | USA    | Retail   | Unknown | Unknown      | Unknown     | Guest         | 529151       | 16509610 | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Europe | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.00           |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Africa | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.00           |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Canada | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.00          |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Africa | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.00          |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Asia   | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.00          |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Africa | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.00          |
| 2020-08-31T00:00:00.000Z | 35          | 8            | 2020          | Africa | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 3888162  | 35.00           |

---
#### 1. What day of the week is used for each week_date value?

    SELECT DISTINCT(TO_CHAR(week_date, 'Day')) AS day_of_week
    FROM clean_weekly_sales;

| day_of_week |
| ----------- |
| Monday      |

---
#### 2. What range of week numbers are missing from the dataset?

    WITH week_series AS (
    SELECT GENERATE_SERIES(1, 53) AS week)
    
    SELECT w.week
    FROM week_series AS w
    LEFT JOIN clean_weekly_sales AS c
    ON w.week = c.week_number
    WHERE c.week_date IS NULL
    ORDER BY w.week;

| week |
| ---- |
| 1    |
| 2    |
| 3    |
| 4    |
| 5    |
| 6    |
| 7    |
| 8    |
| 9    |
| 10   |
| 11   |
| 37   |
| 38   |
| 39   |
| 40   |
| 41   |
| 42   |
| 43   |
| 44   |
| 45   |
| 46   |
| 47   |
| 48   |
| 49   |
| 50   |
| 51   |
| 52   |
| 53   |

---
#### 3. How many total transactions were there for each year in the dataset?

    SELECT calendar_year,
    	SUM(transactions) AS number_transactions
    FROM clean_weekly_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;

| calendar_year | number_transactions |
| ------------- | ------------------- |
| 2018          | 346406460           |
| 2019          | 365639285           |
| 2020          | 375813651           |

---
#### 4. What is the total sales for each region for each month?

        SELECT region,
        	TO_CHAR(TO_DATE(month_number::VARCHAR, 'MM'), 'Month') AS month,
            SUM(sales) AS total_sales
        FROM clean_weekly_sales
        GROUP BY region,
        	month,
            month_number
        ORDER BY region,
        	month_number;
         

| region        | month     | total_sales |
| ------------- | --------- | ----------- |
| Africa        | March     | 567767480   |
| Africa        | April     | 1911783504  |
| Africa        | May       | 1647244738  |
| Africa        | June      | 1767559760  |
| Africa        | July      | 1960219710  |
| Africa        | August    | 1809596890  |
| Africa        | September | 276320987   |
| Asia          | March     | 529770793   |
| Asia          | April     | 1804628707  |
| Asia          | May       | 1526285399  |
| Asia          | June      | 1619482889  |
| Asia          | July      | 1768844756  |
| Asia          | August    | 1663320609  |
| Asia          | September | 252836807   |
| Canada        | March     | 144634329   |
| Canada        | April     | 484552594   |
| Canada        | May       | 412378365   |
| Canada        | June      | 443846698   |
| Canada        | July      | 477134947   |
| Canada        | August    | 447073019   |
| Canada        | September | 69067959    |
| Europe        | March     | 35337093    |
| Europe        | April     | 127334255   |
| Europe        | May       | 109338389   |
| Europe        | June      | 122813826   |
| Europe        | July      | 136757466   |
| Europe        | August    | 122102995   |
| Europe        | September | 18877433    |
| Oceania       | March     | 783282888   |
| Oceania       | April     | 2599767620  |
| Oceania       | May       | 2215657304  |
| Oceania       | June      | 2371884744  |
| Oceania       | July      | 2563459400  |
| Oceania       | August    | 2432313652  |
| Oceania       | September | 372465518   |
| South America | March     | 71023109    |
| South America | April     | 238451531   |
| South America | May       | 201391809   |
| South America | June      | 218247455   |
| South America | July      | 235582776   |
| South America | August    | 221166052   |
| South America | September | 34175583    |
| USA           | March     | 225353043   |
| USA           | April     | 759786323   |
| USA           | May       | 655967121   |
| USA           | June      | 703878990   |
| USA           | July      | 760331754   |
| USA           | August    | 712002790   |
| USA           | September | 110532368   |


---
#### 5. What is the total count of transactions for each platform

    SELECT platform,
    	SUM(transactions) AS total_transactions
    FROM clean_weekly_sales
    GROUP BY platform;

| platform | total_transactions |
| -------- | ------------------ |
| Shopify  | 5925169            |
| Retail   | 1081934227         |

---

#### 6. What is the percentage of sales for Retail vs Shopify for each month?

    WITH total_sales AS (
    	SELECT DATE_TRUNC('month', week_date) AS month,
    		SUM(sales) AS total_sales
    	FROM clean_weekly_sales
    	GROUP BY month
    	ORDER BY month)
    
    , platform_sales AS (
    	SELECT DATE_TRUNC('month', week_date) AS month,
    		platform,
        	SUM(c.sales) AS platform_sales
    	FROM clean_weekly_sales AS c
    	GROUP BY month,
    		platform
        ORDER BY month,
    		platform)
    
    SELECT CONCAT(TO_CHAR(TO_DATE((EXTRACT(month FROM t.month))::VARCHAR, 'MM'), 'Month'), ' ', 
                  EXTRACT(year FROM t.month)) AS month,
    	p.platform,
        ROUND((p.platform_sales::numeric / t.total_sales::numeric) * 100, 2) AS percent_of_sales
    FROM total_sales AS t
    JOIN platform_sales AS p
    ON t.month = p.month;

| month          | platform | percent_of_sales |
| -------------- | -------- | ---------------- |
| March     2018 | Retail   | 97.92            |
| March     2018 | Shopify  | 2.08             |
| April     2018 | Retail   | 97.93            |
| April     2018 | Shopify  | 2.07             |
| May       2018 | Retail   | 97.73            |
| May       2018 | Shopify  | 2.27             |
| June      2018 | Retail   | 97.76            |
| June      2018 | Shopify  | 2.24             |
| July      2018 | Retail   | 97.75            |
| July      2018 | Shopify  | 2.25             |
| August    2018 | Retail   | 97.71            |
| August    2018 | Shopify  | 2.29             |
| September 2018 | Retail   | 97.68            |
| September 2018 | Shopify  | 2.32             |
| March     2019 | Retail   | 97.71            |
| March     2019 | Shopify  | 2.29             |
| April     2019 | Retail   | 97.80            |
| April     2019 | Shopify  | 2.20             |
| May       2019 | Retail   | 97.52            |
| May       2019 | Shopify  | 2.48             |
| June      2019 | Retail   | 97.42            |
| June      2019 | Shopify  | 2.58             |
| July      2019 | Retail   | 97.35            |
| July      2019 | Shopify  | 2.65             |
| August    2019 | Retail   | 97.21            |
| August    2019 | Shopify  | 2.79             |
| September 2019 | Retail   | 97.09            |
| September 2019 | Shopify  | 2.91             |
| March     2020 | Retail   | 97.30            |
| March     2020 | Shopify  | 2.70             |
| April     2020 | Retail   | 96.96            |
| April     2020 | Shopify  | 3.04             |
| May       2020 | Retail   | 96.71            |
| May       2020 | Shopify  | 3.29             |
| June      2020 | Retail   | 96.80            |
| June      2020 | Shopify  | 3.20             |
| July      2020 | Retail   | 96.67            |
| July      2020 | Shopify  | 3.33             |
| August    2020 | Retail   | 96.51            |
| August    2020 | Shopify  | 3.49             |

---
#### 7. What is the percentage of sales by demographic for each year in the dataset?

    WITH total_sales AS (
      SELECT calendar_year,
      	SUM(sales) AS total_sales
      FROM clean_weekly_sales
      GROUP BY calendar_year
      ORDER BY calendar_year)
     
    
    , dem_sales AS (
      SELECT calendar_year,
      	demographic,
      	SUM(sales) AS demographic_sales
      FROM clean_weekly_sales
      GROUP BY calendar_year,
      	demographic
      ORDER BY calendar_year,
      	demographic)
    
    SELECT t.calendar_year,
    	SUM(CASE WHEN d.demographic = 'Couples' 
        	THEN ROUND((d.demographic_sales::numeric / t.total_sales::numeric) * 100, 2) END) AS couples_sales,
        SUM(CASE WHEN d.demographic = 'Families'
            THEN ROUND((d.demographic_sales::numeric / t.total_sales::numeric) * 100, 2) END) AS families_sales,
        SUM(CASE WHEN d.demographic = 'Unknown'
            THEN ROUND((d.demographic_sales::numeric / t.total_sales::numeric) * 100, 2) END) AS unknown_sales
    FROM total_sales AS t
    JOIN dem_sales AS d
    ON t.calendar_year = d.calendar_year
    GROUP BY t.calendar_year;

| calendar_year | couples_sales | families_sales | unknown_sales |
| ------------- | ------------- | -------------- | ------------- |
| 2018          | 26.38         | 31.99          | 41.63         |
| 2019          | 27.28         | 32.47          | 40.25         |
| 2020          | 28.72         | 32.73          | 38.55         |

---

#### 8. Which age_band and demographic values contribute the most to Retail sales?

    SELECT age_band,
    	SUM(sales) AS age_band_sales
    FROM clean_weekly_sales
    WHERE age_band <> 'Unknown' AND 
    	platform = 'Retail'
    GROUP BY age_band
    ORDER BY age_band_sales DESC
    LIMIT 1;

| age_band | age_band_sales |
| -------- | -------------- |
| Retirees | 13005266930    |


    SELECT demographic,
    	SUM(sales) AS dem_sales
    FROM clean_weekly_sales
    WHERE demographic <> 'Unknown' AND
    	platform = 'Retail'
    GROUP BY demographic
    ORDER BY dem_sales DESC
    LIMIT 1;

| demographic | dem_sales   |
| ----------- | ----------- |
| Families    | 12759667763 |

---
#### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

    SELECT calendar_year,
    	platform,
        ROUND(AVG(avg_transaction), 2) AS wrong_avg,
    	ROUND((SUM(sales)::numeric / SUM(transactions)::numeric), 2) AS correct_avg_txn
    FROM clean_weekly_sales
    GROUP BY calendar_year,
    	platform
    ORDER BY calendar_year,
    	platform;

| calendar_year | platform | wrong_avg | correct_avg_txn |
| ------------- | -------- | --------- | --------------- |
| 2018          | Retail   | 42.41     | 36.56           |
| 2018          | Shopify  | 187.80    | 192.48          |
| 2019          | Retail   | 41.47     | 36.83           |
| 2019          | Shopify  | 177.07    | 183.36          |
| 2020          | Retail   | 40.14     | 36.56           |
| 2020          | Shopify  | 174.40    | 179.03          |

---

### C. Before & After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.  
  
Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.  
  
We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before  
  
Using this analysis approach - answer the following questions:

#### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?


    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 4)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 4)
      
    , totals AS (
      SELECT 
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS four_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS four_weeks_after
      FROM clean_weekly_sales)
      
    SELECT ROUND((four_weeks_after::numeric - four_weeks_before::numeric) / four_weeks_before::numeric * 100, 2) AS reduction_rate
    FROM totals;

| reduction_rate |
| -------------- |
| -1.15          |

---

#### 2. What about the entire 12 weeks before and after?

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT 
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales)
      
    SELECT ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS reduction_rate
    FROM totals;

| reduction_rate |
| -------------- |
| -2.14          |

---

#### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

    WITH week_num AS (
      SELECT DISTINCT(week_number)
      FROM clean_weekly_sales
      WHERE week_date = '2020-06-15')
      
    , before AS (
      SELECT DISTINCT week_date
      FROM clean_weekly_sales
      WHERE week_number BETWEEN (SELECT week_number FROM week_num) - 4 AND (SELECT week_number FROM week_num) - 1)
    
    , after AS (
      SELECT DISTINCT week_date
      FROM clean_weekly_sales
      WHERE week_number BETWEEN (SELECT week_number FROM week_num) AND (SELECT week_number FROM week_num) + 3)
        
    , totals AS (
      SELECT calendar_year,
      	SUM(CASE WHEN week_date IN (SELECT * FROM before)
      		THEN sales END) AS four_weeks_before,
      	SUM(CASE WHEN week_date IN (SELECT * FROM after)
      		THEN sales END) AS four_weeks_after
      FROM clean_weekly_sales
      GROUP BY calendar_year
      ORDER BY calendar_year)
    
    SELECT calendar_year,
    	four_weeks_before,
        four_weeks_after,
    	ROUND((four_weeks_after::numeric - four_weeks_before::numeric) / four_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals;

| calendar_year | four_weeks_before | four_weeks_after | growth_rate |
| ------------- | ----------------- | ---------------- | ----------- |
| 2018          | 2119669585        | 2115732898       | -0.19       |
| 2019          | 2249989796        | 2252326390       | 0.10        |
| 2020          | 2345878357        | 2318994169       | -1.15       |

---
**Query #3**

    WITH week_num AS (
      SELECT DISTINCT(week_number)
      FROM clean_weekly_sales
      WHERE week_date = '2020-06-15')
      
    , before AS (
      SELECT DISTINCT week_date
      FROM clean_weekly_sales
      WHERE week_number BETWEEN (SELECT week_number FROM week_num) - 12 AND (SELECT week_number FROM week_num) - 1)
    
    , after AS (
      SELECT DISTINCT week_date
      FROM clean_weekly_sales
      WHERE week_number BETWEEN (SELECT week_number FROM week_num) AND (SELECT week_number FROM week_num) + 11)
        
    , totals AS (
      SELECT calendar_year,
      	SUM(CASE WHEN week_date IN (SELECT * FROM before)
      		THEN sales END) AS twelve_weeks_before,
      	SUM(CASE WHEN week_date IN (SELECT * FROM after)
      		THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY calendar_year
      ORDER BY calendar_year)
    
    SELECT calendar_year,
    	twelve_weeks_before,
        twelve_weeks_after,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals;

| calendar_year | twelve_weeks_before | twelve_weeks_after | growth_rate |
| ------------- | ------------------- | ------------------ | ----------- |
| 2018          | 5863302538          | 6481106927         | 10.54       |
| 2019          | 6883386397          | 6862646103         | -0.30       |
| 2020          | 7126273147          | 6973947753         | -2.14       |

---
### D. Bonus Question
**Query #2**

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT region,
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY region)
      
    SELECT region,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals
    ORDER BY growth_rate;

| region        | growth_rate |
| ------------- | ----------- |
| Asia          | -3.26       |
| Oceania       | -3.03       |
| South America | -2.15       |
| Canada        | -1.92       |
| USA           | -1.60       |
| Africa        | -0.54       |
| Europe        | 4.73        |

---
**Query #3**

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT platform,
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY platform)
      
    SELECT platform,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals
    ORDER BY growth_rate;

| platform | growth_rate |
| -------- | ----------- |
| Retail   | -2.43       |
| Shopify  | 7.18        |

---
**Query #4**

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT age_band,
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY age_band)
      
    SELECT age_band,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals
    ORDER BY growth_rate;

| age_band     | growth_rate |
| ------------ | ----------- |
| Unknown      | -3.34       |
| Middle Aged  | -1.97       |
| Retirees     | -1.23       |
| Young Adults | -0.92       |

---
**Query #5**

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT demographic,
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY demographic)
      
    SELECT demographic,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals
    ORDER BY growth_rate;

| demographic | growth_rate |
| ----------- | ----------- |
| Unknown     | -3.34       |
| Families    | -1.82       |
| Couples     | -0.87       |

---
**Query #6**

    WITH before AS (
    	SELECT DISTINCT(week_date)
    	FROM clean_weekly_sales
    	WHERE week_date < '2020-06-15'
    	ORDER BY week_date DESC
    	LIMIT 12)
    
    , after AS (
      SELECT DISTINCT(week_date)
      FROM clean_weekly_sales
      WHERE week_date >= '2020-6-15'
      ORDER BY week_date
      LIMIT 12)
      
    , totals AS (
      SELECT customer_type,
    	SUM(CASE WHEN week_date IN (SELECT * FROM before)
        	THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date IN (SELECT * FROM after)
        	THEN sales END) AS twelve_weeks_after
      FROM clean_weekly_sales
      GROUP BY customer_type)
      
    SELECT customer_type,
    	ROUND((twelve_weeks_after::numeric - twelve_weeks_before::numeric) / twelve_weeks_before::numeric * 100, 2) AS growth_rate
    FROM totals
    ORDER BY growth_rate;

| customer_type | growth_rate |
| ------------- | ----------- |
| Guest         | -3.00       |
| Existing      | -2.27       |
| New           | 1.01        |

---


