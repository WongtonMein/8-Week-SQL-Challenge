# 🍜 Case Study #1 - Danny's Diner

![Alt text](https://github.com/WongtonMein/Images/blob/main/Wk1%20-%20Danny's%20Diner%2050%25.png?raw=true)

## 📚 Table of Contents
- [Objectives](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#objectives)
- [Entity Relationsihp Diagram](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#questions-and-solutions)

## 📋 Objectives
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

## Entity Relationship Diagram

![Alt text](https://github.com/WongtonMein/Images/blob/main/Wk1%20-%20Entity%20Relationship%20Diagram.png)

## Questions and Solutions

Feel free to join me in writing and executing queries using PostgreSQL in [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138). 

**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT
  sales.customer_id,
  SUM(menu.price) AS total_amount_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
Steps Taken:
- Select the customer_id and **SUM** the total amount spent per customer
- Combine the Sales and Menu tables using a JOIN on their respective product_id values
- The **SUM()** (or aggregated) results are then grouped by customer_id using the **GROUP BY** function
- The customer_id values are then arranged in order alphabetically using the **ORDER BY** function

Results:
| customer_id | total_amount_spent |
|---|---|
| A | 76 |
| B | 74 |
| C | 36 |

- Customer A spent $76
- Customer B spent $74
- Customer C spent $36

***

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS total_days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
Steps Taken:
 - Select the customer_id and **COUNT** the days visited per customer
   - **DISTINCT** is included in the COUNT statement to only return unique values as a customer may have ordered multiple items on the same day
 - The **COUNT()** (or aggregated) results are then grouped by customer_id using the **GROUP BY** function
 - The customer_id values are then arranged in order alphabetically using the **ORDER BY** function

Results:
| customer_id | total_days_visited |
|---|---|
| A | 4 |
| B | 6 |
| C | 2 |

- Customer A visited 4 days
- Customer B visited 6 days
- Customer C visited 2 days

***

**3. What was the first item from the menu purchased by each customer?**

```sql
WITH ordered_sales AS (
  SELECT
    sales.customer_id,
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT
  customer_id,
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
```
Steps Taken:
 - We are looking for the item(s) that each customer purchased on their first visit to Danny's Diner so this will involve utilizing both a window function as well as a **Common Table Expression (CTE)**
 - In the CTE, we will select the customer_id from the Sales table, the product_name from the Menu table, and then use the **DENSE_RANK()** window function to ensure that there are no numeric gaps in the ranking
 - In the **DENSE_RANK()** window function, we will **PARTITION BY** the customer_id and **ORDER BY** the order_date to assign each customer a rank in numeric order
   - Our **DENSE_RANK()** window function will be aliased as "rank"
 - The Sales and Menu tables are combined using a JOIN on their respective product_id values
 - In our SELECT statement, we once again select the customer_id and product_name from our CTE, ordered_sales
 - We then filter our query with WHERE rank = 1
 - Lastly, we GROUP BY customer_id and product_name as a customer may have ordered the same product twice on their first visit

Results:
| customer_id | product_name|
|---|---|
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |

- Customer A ordered both curry and sushi
- Customer B ordered curry
- Customer C ordered ramen

Another option is to write this query using a subquery rather than a CTE. However, a CTE only needs to be defined once and can be used recursively whereas a subquery cannot.

```sql
SELECT
  customer_id,
  product_name
FROM (
  SELECT
    sales.customer_id,
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id) ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
```

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT
  menu.product_name,
  COUNT(sales.product_id) AS total_times_ordered
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY total_times_ordered DESC
LIMIT 1
```

Steps Taken:
- We run a **COUNT** aggregation on the product_id column and **ORDER BY** the count in descending order
- A **LIMIT 1** is applied to list the most purchased item

Results:
| product_name | total_times_ordered|
|---|---|
| ramen | 8 |
- The most ordered item is ramen having been ordered eight times.

**5. Which item was the most popular for each customer?**

```sql
WITH customer_orders AS (
  SELECT
    sales.customer_id,
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.product_id) DESC) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT
  customer_id,
  product_name
FROM customer_orders
WHERE rank = 1;
```
*Note that a customer may have ordered multiple items an equal number of times.

Steps Taken:
- Build a CTE with the customer_id, product_name, and a **DENSE_RANK()** window function
- In the DENSE_RANK() window function, we are going to **PARTITION BY** the customer_id and **ORDER BY** the product_id count in descending order so the largest counts are ranked first
- In our SELECT statement, we select the customer_id and product_name from our CTE and filter **WHERE rank = 1**

Results:
| customer_id | product_name |
|---|---|
| A | ramen |
| B | ramen |
| B | curry |
| B | sushi |
| C | ramen |

- Customer A's most popular item was ramen
- Customer B's most popular items were ramen, curry, and sushi equally
- Customer C's most popular item was ramen

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH member_orders AS (
  SELECT
  	sales.customer_id,
  	menu.product_name,
  	DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  	ON sales.product_id = menu.product_id
  JOIN dannys_diner.members
  	ON sales.customer_id = members.customer_id
  WHERE members.join_date <= sales.order_date
  GROUP BY sales.customer_id, menu.product_name, sales.order_date
)

SELECT
	customer_id,
    product_name
FROM member_orders
WHERE rank = 1;
```
*Note that a customer may have ordered multiple items in their first purchase as a member. Addditionally, not all customers are 

Steps Taken:
- Build a CTE with the customer_id, product_name, and a **DENSE_RANK()** window function
- In the DENSE_RANK() window function, we are going to **PARTITION BY** the customer_id and **ORDER BY** the order_date
- We then JOIN the Sales and Menu tables via their respective product_id values and the Sales and Members tables via their respective customer_id values
- We then apply a filter **WHERE** the join_date value is less than or equal to the order_date so only orders that were placed the same day as a customer's membership join date or later are included
- Results are then grouped by the customer_id, product_name, and order_date
- In our SELECT statement, we select the customer_id and product_name from our CTE and filter **WHERE rank = 1**

Results:
| customer_id | product_name |
|---|---|
| A | curry |
| B | sushi |

- Customer A's first item purchased as a member was curry
- Customer B's first item purchased as a member was sushi


