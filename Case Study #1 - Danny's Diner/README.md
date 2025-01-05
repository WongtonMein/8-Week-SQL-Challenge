# 🍜 Case Study #1 - Danny's Diner

![Alt text](https://github.com/WongtonMein/Images/blob/main/Wk1%20-%20Danny's%20Diner%2050%25.png?raw=true)

# 📚 Table of Contents
- [Business Tasks](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#business-tasks)
- [Entity Relationsihp Diagram](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/WongtonMein/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/README.md#questions-and-solutions)

# 📋 Business Tasks
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

# Entity Relationship Diagram

![Alt text](https://github.com/WongtonMein/Images/blob/main/Wk1%20-%20Entity%20Relationship%20Diagram.png)

# Questions and Solutions

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
- Select the customer_id and SUM the total amount spent per customer
- Combine the Sales and Menu tables using a JOIN on their respective product_id values
- The SUM (or aggregated) results are then grouped by customer_id using the GROUP BY function

Results:
| customer_id | total_amount_spent |
|---|---|
| A | 76 |
| B | 74 |
| C | 36 |
