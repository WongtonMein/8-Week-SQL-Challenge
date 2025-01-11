# 🍜 [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/Wk1%20-%20Danny's%20Diner%2050%25.png)


## 📚 Table of Contents
- [Objectives](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#objectives)
- [Entity Relationsihp Diagram](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#questions-and-solutions)

## 📋 Objectives
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

## Entity Relationship Diagram

![Alt text](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/Wk1%20-%20Entity%20Relationship%20Diagram.png)

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

***

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

***

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

***

**Which item was purchased just before the customer became a member?**
```sql
WITH member_orders AS (
  SELECT
    sales.customer_id,
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date DESC) AS rank
    FROM dannys_diner.sales
    JOIN dannys_diner.menu
      ON sales.product_id = menu.product_id
    JOIN dannys_diner.members
      ON sales.customer_id = members.customer_id
  WHERE members.join_date > sales.order_date
  GROUP BY sales.customer_id, menu.product_name, sales.order_date
)

SELECT
  customer_id,
  product_name
FROM member_orders
WHERE rank = 1;
```

Steps Taken:
 - This question is very similar to the one prior with some changes to account for the customer's item purchase prior to becoming a member
 - We will build a CTE again with customer_id, product_name, and the **DENSE_RANK** window function
 - The DENSE_RANK() window function will be **PARTITION BY** the customer_id and ORDER BY** the order_date in descending order
   - By ordering in descending order, this will rank the most recent order dates first
- We then JOIN the Sales and Menu tables via their respective product_id values and the Sales and Members tables via their respective customer_id values
- We then apply a filter **WHERE** the join_date value is less than or equal to the order_date so only orders that were placed the same day as a customer's membership join date or later are included
- Results are then grouped by the customer_id, product_name, and order_date
- In our SELECT statement, we select the customer_id and product_name from our CTE and filter **WHERE rank = 1**

Results:
| customer_id | product_name |
|---|---|
| A | curry |
| A | sushi |
| B | sushi |

- Customer A ordered curry and sushi before they became a member
- Customer B ordered sushi before they became a member

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT
    sales.customer_id,
    COUNT(sales.product_id) AS total_items_ordered,
    SUM(menu.price) AS total_amount_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
WHERE members.join_date > sales.order_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id
```

Steps Taken:
- We need to JOIN all three tables together now
  - Sales with Menu on their product_id values
  - Sales with Members on their customer_id values
- We then SELECT the customer_id, run a COUNT on the sales.product_id column to determine the total_items_ordered, and run a SUM on the menu.price for the total_amount_spent
- We then apply filter WHERE members.join_date is greater than sales.order_date
  - This ensures all orders are after customers have become a member
- We then GROUP BY and ORDER BY the customer_id

Results: 
| customer_id | total_items_ordered | total_amount_spent |
|---|---|---|
| A | 2 | 25 |
| B | 3 | 40 |

- Customer A ordered two items and spent $25 before becoming a member
- Customer B ordered three items and spent $40 before becoming a member

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

*This question assumes customers became a member before making any purchases

```sql
SELECT
    sales.customer_id,
    SUM(CASE
	  WHEN menu.product_id = 1 THEN price * 20
	  ELSE price * 10
	END) AS total_points_earned
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

Steps Taken:
- We JOIN Sales with Menu on their product_id values
- This problem uses a SUM function with a relatively simple **CASE WHEN** expression inside
  - WHEN (or if) the product_id is 1, then the price is multiplied by 20
  - ELSE the price is multiplied by 10
- The summed values are then GROUP BY and ORDER BY customer_id

Results:
| customer_id | total_points_earned |
|---|---|
| A | 860 |
| B | 940 |
| C | 360 |

- Customer A would have earned 860 points
- Customer B would have earned 940 points
- Customer C would have earned 360 points

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
SELECT
    sales.customer_id,
    SUM(CASE
        WHEN (sales.order_date >= members.join_date) 
        AND (sales.order_date <= members.join_date + INTERVAL '6 days') 
        OR (menu.product_id = 1) THEN price * 20
        ELSE price * 10
    END) AS total_points_earned
FROM dannys_diner.sales
JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
WHERE 
    EXTRACT(MONTH FROM sales.order_date) < 2
    AND members.join_date <= sales.order_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

Steps Taken:
- This question is an extension of the previous question with additional filtering in the **CASE WHEN** statement
- In addition to sushi (product_id = 1), all items are worth double the points for the first week a customer joins the restaurant's membership program
- In our CASE WHEN, we state that:
  - WHEN (or if) the order_date is greater than or equal to the join_date and is less than or equal to six days after the join_date OR if the product_id = 1 (for sushi), the price is multipled by 20
  - ELSE the price is multiplied by 10
- Additionally, the question is asking for customer A and B's points by the end of January. In other words, only orders from their respective membership join dates through the end of January
- We then apply two filters in our **WHERE** clause
  - The first is the month must be less than 2
  - The second being the join_date must be less than or equal to the order_date

Results:
| customer_id | total_points_earned |
|---|---|
| A | 1020 |
| B | 320 |

- Customer A earned 1020 points by the end of January since they joined the program
- Customer B earned 320 points by the end of January since they joined the program

***

## BONUS QUESTIONS

**Join All The Things**

```sql
SELECT
  sales.customer_id,
  sales.order_date,
  menu.product_name,
  menu.price,
  CASE WHEN members.join_date <= sales.order_date THEN 'Y'
ELSE 'N' END AS member
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
FULL JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
ORDER BY sales.customer_id, sales.order_date, menu.product_name;
```

Steps Taken:
- This question is asking us to JOIN all three tables together and show all the relevant information
- Additionally, it is asking us to state whether a customer was a member or not when they had placed an order
- We'll go ahead and start by selecting the customer_id, order_date, product_name, and price from their respective tables
- We'll then write a **CASE WHEN** statement where if the join_date is less than or equal to the order_date, then we will list 'Y' that the customer was a member when this order was placed. Otherwise, we will say 'N' that they were not a a member at the time
- We will then JOIN Sales with Menu on their product_id values
- We then need to use a **FULL JOIN** for Sales and Members on their customer_id values as this will include Customer C who has not signed up for the restaurant's membership program

Results:
| customer_id | order_date | product_name | price | member |
|---|---|---|---|---|
| A | 2021-01-01 | curry | 15 | N |
| A | 2021-01-01 | sushi | 10 | N |
| A | 2021-01-07 | curry | 15 | Y |
| A | 2021-01-10 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| B | 2021-01-01 | curry | 15 | N |
| B | 2021-01-02 | curry | 15 | N |
| B | 2021-01-04 | sushi | 10 | N |
| B | 2021-01-11 | sushi | 10 | Y |
| B | 2021-01-16 | ramen | 12 | Y |
| B | 2021-02-01 | ramen | 12 | Y |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-07 | ramen | 12 | N |

***

**Rank All The Things**

```sql
WITH customer_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    menu.price,
    CASE WHEN members.join_date <= sales.order_date THEN 'Y'
	ELSE 'N' END AS member
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
FULL JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
ORDER BY sales.customer_id, sales.order_date, menu.product_name
)

SELECT
    customer_id,
    order_date,
    product_name,
    price,
    member,
    CASE WHEN member = 'Y' THEN
	RANK() OVER(
	    PARTITION BY customer_id, member
	    ORDER BY order_date)
	ELSE NULL END AS ranking
FROM customer_cte;
```

Steps Taken:
- Conveniently enough, we can use our solution from "Join All The Things" as a CTE for this question
- Outside of the CTE in our SELECT statement, we select the customer_id, order_date, product_name, price, and membership stats "member"
- We then write a **CASE WHEN** statement where if the customer's status as a member is 'Y', we then have a window function where we **RANK()** the order a customer purchased menu items
- We first **PARTITION BY** the customer_id and then member status
- We then **ORDER BY** order_date to ensure we are ranking item orders chronologically
- To close our CASE WHEN statement, if the customer's status as a member is not 'Y' when the order was placed, we set the value to NULL

Results:
| customer_id | order_date | product_name | price | member | ranking |
|---|---|---|---|---|---|
| A | 2021-01-01 | curry | 15 | N | null |
| A | 2021-01-01 | sushi | 10 | N | null |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | null |
| B | 2021-01-02 | curry | 15 | N | null |
| B | 2021-01-04 | sushi | 10 | N | null |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-07 | ramen | 12 | N | null |
