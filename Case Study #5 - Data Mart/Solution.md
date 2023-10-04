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
