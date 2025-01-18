# A. Pizza Metrics


```sql
SELECT
  COUNT(pizza_id) AS ordered_pizzas
FROM customer_orders_temp;
```

```sql
SELECT
  COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders_temp;
```

```sql
SELECT
  runner_id,
  COUNT(order_id) FILTER(WHERE pickup_time IS NOT NULL) AS successful_orders_delivered
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id;
```

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

```sql
SELECT
  customer_id,
  COUNT(pizza_id) FILTER(WHERE pizza_id = 1) AS Meatlovers,
  COUNT(pizza_id) FILTER(WHERE pizza_id = 2) AS Vegetarian
FROM customer_orders_temp
GROUP BY customer_id
ORDER BY customer_id;
```

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
