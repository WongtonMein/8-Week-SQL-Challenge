# 🍕 [Case Study #2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20Pizza%20Runner%2050%25.png)


## 📚 Table of Contents
- [Objectives](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#-objectives)
- [Entity Relationsihp Diagram](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#questions-and-solutions)

## 📋 Objectives
Danny has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

## Entity Relationship Diagram

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20Entity%20Relationship%20Diagram.png)

## Data Cleanup and Transformation

### Table 2: customer_orders

Looking at Table 2: customer_orders, we can see the following:
 - In the `exclusions` column, there are several `null` values and missing or blank spaces
 - In the `extras` column, there are several `null` and `NaN` values as well as missing or blank spaces
These will need to cleaned up before we can begin answering the case study questions

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20customer_orders_table.png)

Steps Taken:
- Remove any `null` values from the `exclusions` column and replace them with a blank space or ' '
- Remove any `null` or `NaN` values from the `extras` column and replace them with a blank space or ' '

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE
    WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
    ELSE exclusions
    END AS exclusions,
  CASE
    WHEN extras IS NULL or extras LIKE 'null' THEN ''
    ELSE extras
    END AS extras,
  order_time
FROM pizza_runner.customer_orders;
```

> insert temp_table here





### Table 3: runner_orders

```sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT
  order_id,
  runner_id,
  CASE
    WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ''
    ELSE pickup_time
    END AS pickup_time,
  CASE
    WHEN distance IS NULL OR distance LIKE 'null' THEN ''
    WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
    ELSE distance
    END AS distance,
  CASE
    WHEN duration IS NULL or duration LIKE 'null' THEN ''
    WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
    ELSE duration
    END AS duration,
  CASE
    WHEN cancellation IS NULL OR cancellation LIKE 'null' or cancellation LIKE 'NaN' THEN ''
    ELSE cancellation
    END AS cancellation
FROM pizza_runner.runner_orders;
```

```sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE timestamp,
ALTER COLUMN distance TYPE FLOAT,
ALTER COLUMN duration TYPE INT;
```


## Questions and Solutions

Feel free to join me in writing and executing queries using PostgreSQL in [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

[**A. Pizza Metrics**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics.md)

[**B. Runner and Customer Experience**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md)

[**C. Ingredient Optimisation**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/C.%20Ingredient%20Optimisation.md)

[**D. Pricing and Ratings**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/D.%20Pricing%20and%20Ratings.md)

[**E. Bonus Questions**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/E.%20Bonus%20Questions.md)
