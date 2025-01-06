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

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

TBD
