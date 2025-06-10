# LeetCode Problem 2752: Customers with Maximum Number of Transactions on Consecutive Days

## Problem Description

You are given a `Transactions` table containing purchase records made by customers. Each transaction has a unique identifier and records the date and amount of the transaction.

The task is to identify all `customer_id`s who have made the **maximum number of transactions on consecutive days**. The result should include all customers who have the highest streak of daily transactions without missing a day in between.

### Tables:

#### Transactions Table:

| transaction_id | customer_id | transaction_date | amount |
|----------------|-------------|------------------|--------|
| 10             | 200         | 2023-07-01       | 120    |
| 11             | 200         | 2023-07-02       | 180    |
| 12             | 200         | 2023-07-03       | 240    |
| 13             | 201         | 2023-07-01       | 70     |
| 14             | 201         | 2023-07-03       | 110    |
| 15             | 201         | 2023-07-04       | 210    |
| 16             | 204         | 2023-07-01       | 150    |
| 17             | 204         | 2023-07-02       | 190    |
| 18             | 204         | 2023-07-03       | 220    |

### Expected Output:

| customer_id |
|-------------|
| 200         |
| 204         |

### Problem Constraints:
- A transaction streak is considered only if the dates are consecutive without gaps.
- Multiple transactions on the same day by a customer are not possible (unique `customer_id, transaction_date` pairs).
- Return all customers who match the **maximum** streak count.
- Output should be ordered by `customer_id` in ascending order.

---

## Solution Approaches

### Approach 1: Grouping Consecutive Transactions

This approach uses window functions to detect breaks in consecutive transaction days and then groups sequences of such transactions to find the maximum streaks per customer.

#### SQL Query:
```sql
WITH GroupedTransactions AS (
    SELECT *, 
           SUM(CASE WHEN prev_date IS NULL OR DATEDIFF(transaction_date, prev_date) > 1 THEN 1 
           ELSE 0 END) 
           OVER (PARTITION BY customer_id ORDER BY transaction_date) AS group_id
    FROM (
        SELECT *, 
               LAG(transaction_date) OVER (PARTITION BY customer_id ORDER BY transaction_date) AS prev_date
        FROM Transactions) s),

RankedCount AS (
    SELECT customer_id, 
           DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS rk
    FROM GroupedTransactions 
    GROUP BY customer_id, group_id)

SELECT customer_id FROM RankedCount WHERE rk = 1 ORDER BY customer_id;
```

#### Explanation:
- **Step 1**: Use `LAG` to get the previous transaction date for each customer based on chronological order.
- **Step 2**: Compare the difference between the current and previous transaction date. If it’s not exactly 1 day, a new group starts.
- **Step 3**: Apply a running total (`SUM`) to generate a `group_id` for each sequence.
- **Step 4**: Group by `customer_id` and `group_id` and count how many consecutive days are in each group.
- **Step 5**: Return customers whose streak length is equal to the global maximum.

---

## Performance Analysis

### Method 1 (Grouping Consecutive Transactions):

- **Time complexity**: O(N log N), due to sorting within the `OVER (PARTITION BY ...)` clause.
- **Space complexity**: O(N), to store intermediate results and group information.

---

## Conclusion

- **Method 1**: Efficient and scalable for analyzing streaks of consecutive records using window functions and aggregation. It handles partitioning and grouping seamlessly to detect transaction sequences.

### Final Conclusion:
- This method is robust and ideal for detecting longest consecutive patterns in time-series data, particularly for customer behavior analytics.
