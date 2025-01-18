# A. Pizza Metrics

**1. How many pizzas were ordered?**

```sql
SELECT
  COUNT(pizza_id) AS ordered_pizzas
FROM customer_orders_temp;
```
Steps Taken:
- **COUNT** all pizza_id values from our temporary table, customer_orders_temp

| ordered_pizzas |
|---|
| 14 |

- 14 pizzas were ordered total

***

**2. How many unique customer orders were made?**

```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders_temp;
```
Steps Taken:
- **COUNT** all order_id values from our temporary table, customer_orders_temp
- We add the **DISTINCT** keyword to return unique values

| unique_customer_orders |
|---|
| 10 |

- There were 10 unique customer orders

***

**3. How many successful orders were delivered by each runner?**

```sql
SELECT
  runner_id,
  COUNT(order_id) FILTER(WHERE pickup_time IS NOT NULL) AS successful_orders_delivered
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id;
```
Steps Taken:
- SELECT the runner_id and **COUNT** all order_id values from our temporary table, customer_orders_temp
- Additionally, we will **FILTER** our order_id count WHERE the pickup_time is not NULL to ensure we query successful pickups
- We then GROUP BY and ORDER BY the runner_id

| runner_id | successful_orders_delivered |
|---|---|
| 1 | 4 |
| 2 | 4 |
| 3 | 1 |

- runner_id 1 successfully delivered four orders
- runner_id 2 successfully delivered three orders
- runner_id 3 successfully delivered one order

***

**4. How many of each type of pizza was delivered?**

```sql
SELECT
  c.pizza_id,
  pizza_name,
  COUNT(c.pizza_id) AS pizzas_delivered
FROM customer_orders_temp c
JOIN runner_orders_temp r
  ON c.order_id = r.order_id
JOIN pizza_names p
  ON c.pizza_id = p.pizza_id
WHERE pickup_time IS NOT NULL
GROUP BY c.pizza_id, p.pizza_name
ORDER BY c.pizza_id;
```
Steps Taken:
- SELECT the pizza_id, pizza_name, and **COUNT** all pizza_id values from our temporary table, customer_orders_temp
- We then **JOIN** customer_orders_temp with runner_orders_temp on their order_id values
- The pizza_names table is joined on customer_orders_temp on their pizza_id values
- Similar to the previous question, we filter pickup_time WHERE it is not NULL to ensure we query successful pickups
- We then GROUP BY pizza_id and pizza_name and ORDER BY pizza_id

| pizza_id | pizza_name | pizzas_delivered |
|---|---|---|
| 1 | Meatlovers | 9 |
| 2 | Vegetarian | 3 |

- Nine Meatlovers pizzas were delivered
- Three Vegetarian pizzas were delivered
***

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT
  customer_id,
  COUNT(pizza_id) FILTER(WHERE pizza_id = 1) AS Meatlovers,
  COUNT(pizza_id) FILTER(WHERE pizza_id = 2) AS Vegetarian
FROM customer_orders_temp
GROUP BY customer_id
ORDER BY customer_id;
```
Steps Taken:
- SELECT the customer_id and two lines where we **COUNT** all pizza_id values from our temporary table, customer_orders_temp
- On the first COUNT, we apply a FILTER WHERE the pizza_id = 1
  - This is aliased as "Meatlovers"
- On the second COUNT, we apply a FILTER WEHRE the pizza-Id = 2
  - This is aliased as "vegetarian"
- We then GROUP BY and ORDER BY customer_id

| customer_id | meatlovers | vegetarian |
|---|---|---|
| 101 | 2 | 1 |
| 102 | 2 | 1 |
| 103 | 3 | 1 |
| 104 | 3 | 0 |
| 105 | 0 | 1 |

- customer_id 101 ordered two Meatlovers pizzas and one Vegetarian pizza
- customer_id 102 ordered two Meatlovers pizzas and one Vegetarian pizza
- customer_id 103 ordered three Meatlovers pizzas and one Vegetarian pizza
- customer_id 104 ordered three Meatlovers pizzas
- customer_id 105 ordered one Vegetarian pizza

***

**6. What was the maximum number of pizzas delivered in a single order?**

```sql
SELECT
  r.order_id,
  COUNT(pizza_id) AS max_pizzas_delivered
FROM customer_orders_temp c
JOIN runner_orders_temp r
  ON c.order_id = r.order_id
WHERE pickup_time IS NOT NULL
GROUP BY r.order_id
ORDER BY COUNT(pizza_id) DESC
LIMIT 1;
```
Steps Taken:
- SELECT the order_id and **COUNT** the pizza_id values from our temporary table, customer_orders_temp
- We then **JOIN** customer_orders_temp with runner_orders_temp on their order_id values
- In our WHERE clause, we filter WHERE pickup_time IS NOT NULL
- We then GROUP BY order_id and ORDER BY the pizza_id count in descending order
- Lastly, we limit our results to one to show the maximum numbers of pizzas delievered in a single order

| order_id | meatlovers |
|---|---|
| 4 | 3 |

- The maximum number of pizzas delievered in a single order was three pizzas on order_id 4

***

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

***

**8. How many pizzas were delivered that had both exclusions and extras?**

***

**9. What was the total volume of pizzas ordered for each hour of the day?**

***

**10. What was the volume of orders for each day of the week?**

