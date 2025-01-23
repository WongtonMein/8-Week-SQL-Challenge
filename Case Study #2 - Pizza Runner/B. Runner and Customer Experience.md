# B. Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
```sql
SELECT
  EXTRACT(WEEK FROM registration_date) AS week_number,
  COUNT(runner_id) AS registration_count
FROM pizza_runner.runners
GROUP BY week_number;
```

Steps Taken:
 - **EXTRACT** the week number from the `registration_date` column and alias as week_number
 - COUNT runner_id values and alias as registration_count
 - GROUP BY the week_number

Results:
| week_number | registration_count |
|---|---|
| 53 | 2 |
| 1 | 1 |
| 2 | 1 |

Note, these results are based on the **ISOYEAR** which means the year can start anywhere between December 29 and January 1

- Two runners signed up in week 53 of the previous year
- One runner signed up in week 1 of the current year
- One runner singed up in week 2 of the current year

To respond to the question more accurately, we need to define week 1 starting on 2021-01-01

```sql
SELECT
  ((registration_date - '2021-01-01') / 7) + 1 AS week_number,
  registration_date - ((registration_date - '2021-01-01') % 7) AS week_start_date,
  COUNT(runner_id) AS registration_count
FROM pizza_runner.runners
GROUP BY week_number, week_start_date
ORDER BY week_number;
```

Steps Taken:
 - Define the week number
   - Subtract the registration_date by '2021-01-01' resulting in an integer
   - Divide the resulting integer by 7, which truncates any decimal value
   - Add 1
 - Define the week_start_date
   - Subtract the registration_date by '2021-01-01' resulting in an integer
   - Use the modulus (%) function to return the remainder of the previous integer when divided by 7
   - Subtract this new integer from the registration_date to return the week_start_date
 - COUNT runner_id values and alias as registration_count

Results:
| week_number | week_start_date | registration_count |
|---|---|---|
| 1 | 2021-01-01 | 2 |
| 2 | 2021-01-08 | 1 |
| 3 | 2021-01-15 | 1 |

- Two runners signed up in week 1 which started on 2021-01-01
- One runner signed up in week 2 which started on 2021-01-08
- One runner signed up in week 3 which started on 2021-01-15

***

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT
  runner_id,
  EXTRACT(MINUTE FROM DATE_TRUNC('second', AVG(r.pickup_time - c.order_time))) +
  ROUND((EXTRACT(SECOND FROM DATE_TRUNC('second', AVG(r.pickup_time - c.order_time))) / 60)::decimal, 2) AS "avg_arrival_time_(min)"
FROM customer_orders_temp c
JOIN runner_orders_temp r
  ON c.order_id = r.order_id
GROUP BY runner_id
ORDER BY runner_id;
```

Steps Taken:
 - We will SELECT the runner_id and calculate the average time in minutes for the runner to arrive to pickup the order
 - Average pickup time calculation
   - Take the average time of the difference between pickup_time and order_time
   - I truncated the timestamp at the seconds mark, otherwise it goes down the milliseconds
   - The calculated timestamp value has its MINUTE value extracted
     - This can also be done using DATE_PART
   - The calculated timestamp value then has its SECOND value extracted and rounded to two decimal places
   - The extracted MINUTE and SECOND values are then added together
- We JOIN the customer_orders_temp and the runner_orders_temp tables on their order_id values
- We then GROUP BY and ORDER BY the runner_id

Results:
| runner_id | avg_arrival_time_(min) |
|---|---|
| 1 | 15.67 |
| 2 | 23.72 |
| 3 | 10.47 |

- runner_id 1 arrived at the store in 15.67 minutes on average
- runner_id 2 arrived at the store in 23.72 minutes on average
- runner_id 3 arrived at the store in 10.47 minutes on average

Alternatively, you can also use a CTE to produce the same results.

```sql
WITH avg_arrival_cte AS (
  SELECT
    runner_id,
    DATE_TRUNC('second', AVG(r.pickup_time - c.order_time)) AS avg_time_trunc
  FROM customer_orders_temp c
  JOIN runner_orders_temp r
    ON c.order_id = r.order_id
  GROUP BY runner_id
  ORDER BY runner_id
)

SELECT
  runner_id,
  EXTRACT(MINUTE FROM avg_time_trunc) +
  ROUND(((EXTRACT(SECOND FROM avg_time_trunc) / 60)::decimal), 2) AS "avg_arrival_time_(min)"
FROM avg_arrival_cte
GROUP BY runner_id, avg_time_trunc
ORDER BY runner_id;
```

***

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
WITH pizza_prep_cte AS (
  SELECT
    c.order_id,
    COUNT(c.order_id) AS pizza_count,
    EXTRACT(MINUTE FROM DATE_TRUNC('second', (r.pickup_time - c.order_time))) +
    ROUND((EXTRACT(SECOND FROM DATE_TRUNC('second', (r.pickup_time - c.order_time))) / 60)::decimal, 2) AS pizza_prep_time
  FROM customer_orders_temp c
  JOIN runner_orders_temp r
    ON c.order_id = r.order_id
  GROUP BY c.order_id, r.pickup_time, c.order_time
  ORDER BY order_id
)

SELECT
  pizza_count,
  ROUND(AVG(pizza_prep_time)::integer,2) AS avg_pizza_prep_time
FROM pizza_prep_cte
GROUP BY pizza_count
ORDER BY pizza_count
```

Steps Taken:
 - We will build a CTE with the order_id, a COUNT of the order_id values, and a similar query to the previous question to determine the pizza preparation time in minutes (pizza_prep_time)
- We JOIN the customer_orders_temp and the runner_orders_temp tables on their order_id values
 - We then GROUP BY the order_id, pickup_time, and order_time and ORDER BY the order_id
 - In our main SELECT statement, we will SELECT the pizza_count and take the average of the pizza_prep_time rounded to two decimals
 - We GROUP BY and ORDER BY the pizza_count

Results:
| pizza_count | avg_pizza_prep_time |
|---|---|
| 1 | 12.00 |
| 2 | 18.00 |
| 3 | 29.00 |

 - Preparing one pizza takes 12.00 minute on average
 - Preparing two pizzas takes 18.00 minutes on average
 - Preparing three pizzas takes 29.00 minutes on average

***

**4. What was the average distance travelled for each customer?**

```sql
SELECT
  customer_id,
  ROUND(AVG(distance), 2) AS avg_distance
FROM customer_orders_temp c
JOIN runner_orders_temp r
  ON c.order_id = r.order_id
GROUP BY customer_id
ORDER BY customer_id;
```

Steps Taken:
 - We SELECT the customer_id and take the average distance travelled rounded to two decimals
 - We JOIN the customer_orders_temp and the runner_orders_temp tables on their order_id values
 - We then GROUP BY and ORDER BY the customer_id

Results:
| customer_id | avg_distance |
|---|---|
| 101 | 20.00 |
| 102 | 16.73 |
| 103 | 23.40 |
| 104 | 10.00 |
| 105 | 15.00 |

- The average distance travelled to customer_id 101 is 20.00 kilometers
- The average distance travelled to customer_id 102 is 16.73 kilometers
- The average distance travelled to customer_id 103 is 23.40 kilometers
- The average distance travelled to customer_id 104 is 10.00 kilometers
- The average distance travelled to customer_id 105 is 25.00 kilometers

***

**5. What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT
  MIN(duration) AS min_delivery_time,
  MAX(duration) AS max_delivery_time,
  MAX(duration) - MIN(duration) AS diff_btwn_max_and_min_duration
FROM runner_orders_temp
```

Steps Taken:
 - We query the MIN() and MAX() values of the `duration` column and the difference between MAX() and MIN()

| min_delivery_time | max_delivery_time | diff_btwn_max_and_min_duration |
|---|---|---|
| 10 | 40 | 30 |

- The difference between the longest and shortest delivery times for all orders is 30 minutes

***

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```sql
SELECT
  runner_id,
  distance,
  duration,
  ROUND(distance / (duration / 60.0), 2) AS km_per_hr
FROM runner_orders_temp
WHERE duration IS NOT NULL
ORDER BY runner_id, order_id;
```

Steps Taken:
 - SELECT the runner_id, distance, and duration columns
 - The average speed each runner_id delivered each order is then calculated
 - A WHERE clause is added to exclude any orders with a NULL duration value
 - We then ORDER BY the runner_id as well as the order_id

Results:
| runner_id | distance | duration | km_per_hr |
|---|---|---|---|
| 1 | 20 | 32 | 37.50 |
| 1 | 20 | 27 | 44.44 |
| 1 | 13.4 | 20 | 40.20 |
| 1 | 10 | 10 | 60.00 |
| 2 | 23.4 | 40 | 35.10 |
| 2 | 25 | 25 | 60.00 |
| 2 | 23.4 | 15 | 93.60 |
| 3 | 10 | 15 | 40.00 |

 - runner_id 1's average speed ranged from 37.50 km/hr to 60.00 km/hr
 - runner_id 2's average speed ranged from 35.10 km/hr to 93.60 km/hr
 - runner_id 3 has only delivered one order so their average speed is 40.00 km/hr

***

**7. What is the successful delivery percentage for each runner?**

```sql
WITH order_success_cte AS (
  SELECT
    runner_id,
    COUNT(order_id) AS all_orders,
    COUNT(order_id) FILTER(WHERE duration IS NOT NULL) AS successful_orders
  FROM runner_orders_temp
  GROUP BY runner_id
  ORDER BY runner_id
)

SELECT
  runner_id,
  ROUND(successful_orders * 100.0 / all_orders, 2) || '%' AS percent_successful_orders
FROM order_success_cte
GROUP BY runner_id, successful_orders, all_orders
ORDER BY runner_id;
```

Steps Taken:
 - We will build a CTE with the runner_id, a COUNT of the order_id aliased as all_orders, and a COUNT of the order_id where we utilize a FILTER to exclude any rows where duration is NULL
 - In our main SELECT statement, we SELECT the runner_id, successful_orders, all_orders and calculate the percentage of successful deliveries by dividing the successful_orders by all_orders
  - Note, I concatenated a '%' symbol, but this is not necessary
  - Otherwise remove the "|| '%'" and change the "* 100.0" to "* 1.0"
- WE THEN GROUP BY the runner_id, successful_orders, and all_orders and ORDER BY the runner_id

Results:
| runner_id | succesful_orders | all_orders | percent_successful_orders |
|---|---|---|---|
| 1 | 4 | 4 | 100.00% |
| 2 | 3 | 4 | 75.00% |
| 3 | 1 | 2 | 50.00% |

- runner_id 1 has a 100.00% successful delivery rate
- runner_id 2 has a 75.00% successful delivery rate
- runner_id 3 has a 50.00% successful delivery rate

***

Click [here](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/C.%20Ingredient%20Optimisation.md) to view my solutions to C. Ingredient Optimisation.
