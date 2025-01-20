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
