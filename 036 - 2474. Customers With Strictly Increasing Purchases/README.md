# LeetCode Problem 2474: Customers With Strictly Increasing Purchases

## Problem Description

You are given a table `Orders` that tracks individual customer purchases. Each row contains an `order_id`, the `customer_id` who made the purchase, the `order_date`, and the `price` of the order.

Your task is to identify the customers whose **total yearly purchases are strictly increasing** over the range of years in which they placed at least one order.

For each customer:
- Calculate the **total purchase amount** for each year.
- The **first year** is the year of the customer's first order.
- The **last year** is the year of their last order.
- If the customer did **not place any order** in a certain year within this range, their total for that year is considered **0**.
- Only include customers whose **yearly totals are strictly increasing** from the first to the last year.

### Tables:

#### Orders Table:

| order_id | customer_id | order_date | price |
|----------|-------------|------------|-------|
| 101      | 10          | 2018-06-15 | 1300  |
| 102      | 10          | 2018-12-30 | 1700  |
| 103      | 10          | 2019-02-01 | 2900  |
| 104      | 10          | 2020-03-12 | 3600  |
| 105      | 10          | 2021-11-20 | 5100  |
| 106      | 11          | 2016-04-09 | 800   |
| 107      | 11          | 2018-09-17 | 1100  |
| 108      | 12          | 2019-01-03 | 950   |
| 109      | 12          | 2020-10-21 | 950   |

### Expected Output:

| customer_id |
|-------------|
| 10          |

### Problem Constraints:
- The `order_id` is unique and serves as the primary key.
- Customers may skip years between purchases; skipped years count as 0 in total.
- A customer's total purchases must increase strictly (i.e., each year must be greater than the last).
- Final output should include only the `customer_id` values that meet this criteria.

---

## Solution Approaches

### Approach 1: Year-Purchase Alignment via Dense Ranking

This method evaluates the change in purchase totals over the years and checks if the increase is **strictly linear** by leveraging the mathematical property of lines with slope 1.

#### SQL Query:
```sql
WITH AnnualPrices AS (
    SELECT customer_id, YEAR(order_date) AS order_year, SUM(price) AS price
    FROM Orders
    GROUP BY customer_id, order_year)

SELECT customer_id
FROM (
    SELECT *, order_year - DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY price) AS group_id
    FROM AnnualPrices) s
GROUP BY customer_id
HAVING COUNT(DISTINCT group_id) = 1;
```

#### Explanation:
- A Common Table Expression (CTE) calculates the **total purchase amount per customer per year**.
- For each customer, assign a **dense rank** to the total price values in ascending order.
- Compute a `group_id` as the **difference between the order year and its price rank**.
- If a customer's purchases are strictly increasing each year:
  - The year and its rank will both increase by one.
  - Therefore, the difference (`year - rank`) remains **constant**.
- Group by `customer_id` and check that **only one unique group_id** exists.
- Customers with exactly one group_id have strictly increasing purchases and are included in the result.

### Approach 2: Grouping with Lag Functions

This method uses **window functions** (like `LAG()`) to compare the total purchases of each customer from one year to the next.

#### SQL Query:
```sql
WITH AnnualPrices AS (
    SELECT customer_id, YEAR(order_date) AS order_year, SUM(price) AS total_spent
    FROM Orders
    GROUP BY customer_id, order_year
),
SortedOrders AS (
    SELECT *, 
    LAG(order_year) OVER (PARTITION BY customer_id ORDER BY order_year) AS prev_year,
    LAG(total_spent) OVER (PARTITION BY customer_id ORDER BY order_year) AS prev_spent
    FROM AnnualPrices
),
GroupedOrders AS (
    SELECT *, 
    SUM(CASE 
        WHEN prev_year IS NULL 
        OR prev_year != order_year - 1 
        OR prev_spent >= total_spent 
        THEN 1 
        ELSE 0 
    END) OVER (PARTITION BY customer_id ORDER BY order_year) AS group_id
    FROM SortedOrders
)
SELECT customer_id 
FROM GroupedOrders 
GROUP BY customer_id 
HAVING COUNT(DISTINCT group_id) = 1;
```

#### Explanation:
- **Step 1**: The query calculates the **total purchase amount per customer per year**.
- **Step 2**: For each customer, the `LAG()` function is used to get the previous year's total purchase (`prev_spent`) and the previous year's year (`prev_year`).
- **Step 3**: A new `group_id` is calculated based on the following conditions:
  - If a customer did not make purchases in a previous year (i.e., a gap), or if the current year’s purchases are less than or equal to the previous year's, it marks a break.
- **Step 4**: By partitioning the data by `customer_id` and ordering by `order_year`, we can track groups of years where purchases are strictly increasing.
- **Step 5**: The final filter ensures that customers whose purchases are strictly increasing from year to year (i.e., they only belong to one group_id) are selected.

---

## Performance Analysis

### Method 1 (Year-Purchase Alignment via Dense Ranking):

- **Time complexity**: `O(n log n)` due to the use of `GROUP BY`, sorting, and `DENSE_RANK()` operations.
- **Space complexity**: `O(n)` for intermediate storage in the CTE and subqueries.

### Method 2 (Grouping with Lag Functions):

- **Time complexity**: `O(n log n)` because of the use of `LAG()` function and ordering operations, with additional time required for grouping.
- **Space complexity**: `O(n)` due to window functions storing previous year's values and intermediate grouping data.

---

## Conclusion

- **Method 1**: Efficient and concise. It cleverly uses dense ranking to avoid explicitly checking adjacent years and handles missing years automatically through rank misalignment.
  
- **Method 2**: Provides a more granular view using the `LAG()` function to track previous year purchases, which can help in more complex scenarios where specific historical comparisons matter. However, it requires additional handling for gaps in the year sequence.

### Final Conclusion:
- Both methods work efficiently for this problem, but **Method 1** (dense ranking) may be slightly more efficient in simpler cases due to fewer operations and a cleaner approach. **Method 2** may be more flexible if additional custom conditions are needed in future scenarios.
