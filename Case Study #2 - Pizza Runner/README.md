# 🍕 [Case Study #2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20Pizza%20Runner%2050%25.png)


## 📚 Table of Contents
- [Objectives](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#-objectives)
- [Entity Relationsihp Diagram](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/WongtonMein/8-Week-SQL-Challenge/edit/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#questions-and-solutions)
- [Data Cleanup and Transformation](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/README.md#data-cleanup-and-transformation)

## 📋 Objectives
Danny has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

## Entity Relationship Diagram

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20Entity%20Relationship%20Diagram.png)

## Questions and Solutions

Feel free to join me in writing and executing queries using PostgreSQL in [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65).

[**A. Pizza Metrics**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics.md)

[**B. Runner and Customer Experience**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md)

[**C. Ingredient Optimisation**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/C.%20Ingredient%20Optimisation.md)

[**D. Pricing and Ratings**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/D.%20Pricing%20and%20Ratings.md)

[**E. Bonus Questions**](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/E.%20Bonus%20Questions.md)

## Data Cleanup and Transformation

### Table 2: customer_orders

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20customer_orders_table.png)

Looking at Table 2: customer_orders, we can see the following:
 - In the `exclusions` column, there are several `null` values and missing or blank spaces
 - In the `extras` column, there are several `null` and `NaN` values as well as missing or blank spaces

These will need to cleaned up before we can begin answering the case study questions

Steps Taken:
- Remove any '' values from the `exclusions` column and replace them with a `null` value
- Remove any '' values from the `extras` column and replace them with a `null` value

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE
    WHEN exclusions = '' OR exclusions LIKE 'null' THEN NULL
    ELSE exclusions
    END AS exclusions,
  CASE
    WHEN extras = '' or extras LIKE 'null' THEN NULL
    ELSE extras
    END AS extras,
  order_time::timestamp
FROM pizza_runner.customer_orders;
```

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20customer_orders_temp_table.png)

***

### Table 3: runner_orders

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20runner_orders_table.png)

Looking at Table 3: runner_orders, we can see the following:
 - In the `pickup_time` column, there are a few `null` values
 - In the `distance` column, there are a few `null` values and distances with units of measure
 - In the `duration` column, there are a few `null` values and durations with units of measure
 - In the `cancellation` column, there are a few `null` values and missing or blank spaces

These will need to cleaned up before we can begin answering the case study questions

Steps Taken:
- Remove any potential '' values from the `exclusions` column and replace them with a `null` value
- Remove any '' values from the `distance` column and replace them with a `null` value. Additionally, we **TRIM** 'km' off values in the `distance` column and **CAST** them as a **decimal** data type
- Remove any '' values from the `duration` column and replace them with a `null` value. Additionally, we **TRIM** 'mins', 'minute', and 'minutes' off values in the `duration` column and **CAST** them as an **integer** data type
- Remove any potential '' values from the `cancellation` column and replace them with a `null` value

```sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT
  order_id,
  runner_id,
  CASE
    WHEN pickup_time = '' OR pickup_time LIKE 'null' THEN NULL
    ELSE pickup_time::timestamp
    END AS pickup_time,
  CASE
    WHEN distance = '' OR distance LIKE 'null' THEN NULL
    WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)::decimal
    ELSE distance::decimal
    END AS distance,
  CASE
    WHEN duration = '' or duration LIKE 'null' THEN NULL
    WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)::integer
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)::integer
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)::integer
    ELSE duration::integer
    END AS duration,
  CASE
    WHEN cancellation = '' OR cancellation LIKE 'null' or cancellation LIKE 'NaN' THEN NULL
    ELSE cancellation
    END AS cancellation
FROM pizza_runner.runner_orders;
```

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Wk2%20-%20runner_orders_temp_table.png)

***

Click [here](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics.md) to view my solutions to A. Pizza Metrics.
