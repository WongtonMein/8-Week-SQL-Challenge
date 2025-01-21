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
