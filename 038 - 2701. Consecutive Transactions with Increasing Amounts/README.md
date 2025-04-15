# LeetCode Problem 2701: Consecutive Transactions with Increasing Amounts

## Problem Description

You are given a `Transactions` table that records transaction details for customers, including the transaction ID, customer ID, date, and transaction amount. Each transaction is unique by `(customer_id, transaction_date)`.

The task is to identify customers who have made transactions for **at least three consecutive days** where the **amount increased each day**. A customer may have multiple such sequences.

You need to return each customer's ID along with the **start and end dates** of each valid sequence, sorted by `customer_id` in ascending order.

### Tables:

#### Transactions Table:

| transaction_id | customer_id | transaction_date | amount |
|----------------|-------------|------------------|--------|
| 201            | 501         | 2024-06-01       | 120    |
| 202            | 501         | 2024-06-02       | 180    |
| 203            | 501         | 2024-06-03       | 250    |
| 204            | 502         | 2024-06-01       | 90     |
| 205            | 502         | 2024-06-03       | 110    |
| 206            | 502         | 2024-06-04       | 115    |
| 207            | 504         | 2024-06-10       | 100    |
| 208            | 504         | 2024-06-11       | 130    |
| 209            | 504         | 2024-06-12       | 170    |
| 210            | 504         | 2024-06-13       | 210    |
| 211            | 504         | 2024-06-20       | 220    |
| 212            | 504         | 2024-06-21       | 250    |
| 213            | 504         | 2024-06-22       | 270    |

### Expected Output:

| customer_id | consecutive_start | consecutive_end |
|-------------|-------------------|-----------------|
| 501         | 2024-06-01        | 2024-06-03      |
| 504         | 2024-06-10        | 2024-06-13      |
| 504         | 2024-06-20        | 2024-06-22      |

### Problem Constraints:
- Transactions are uniquely identified by the combination of `customer_id` and `transaction_date`.
- Only consider **consecutive days** with **increasing amounts**.
- At least **3 consecutive days** are needed to form a valid result.

---

## Solution Approaches

### Approach 1: Grouping with Window Functions

This approach relies on using SQL window functions to identify transaction sequences that meet the criteria of being on consecutive days with increasing amounts.

#### SQL Query:
```sql
WITH SortedTransactions AS (
    SELECT *, 
    LAG(transaction_date) OVER (PARTITION BY customer_id ORDER BY transaction_date) AS prev_date,
    LAG(amount) OVER (PARTITION BY customer_id ORDER BY transaction_date) AS prev_amount
    FROM Transactions),

GroupedTransactions AS (
    SELECT *, 
    SUM(CASE WHEN prev_date IS NULL 
        OR prev_date != transaction_date - INTERVAL 1 DAY
        OR prev_amount >= amount THEN 1 ELSE 0 END) 
    OVER (PARTITION BY customer_id ORDER BY transaction_date) AS group_id
    FROM SortedTransactions)

SELECT customer_id, MIN(transaction_date) AS consecutive_start, MAX(transaction_date) AS consecutive_end
FROM GroupedTransactions 
GROUP BY customer_id, group_id 
HAVING COUNT(*) > 2;
```

#### Explanation:
- **Step 1**: Use the `LAG()` function to get the previous date and amount for each transaction of a customer.
- **Step 2**: Check if the current transaction continues the sequence (i.e., it’s one day after the previous and the amount is higher). If not, start a new group.
- **Step 3**: Use a `SUM()` window function to assign a group number for each sequence.
- **Step 4**: Group by `customer_id` and this computed group ID to find sequences.
- **Step 5**: Filter those sequences that are at least 3 days long.
- **Step 6**: Return the start and end dates of each valid transaction streak.

---

## Performance Analysis

### Method 1 (Grouping with Window Functions):

- **Time complexity**: O(N log N), primarily due to sorting by `transaction_date` for each customer.
- **Space complexity**: O(N) for intermediate window function storage and grouping.

---

## Conclusion

- **Method 1**: Efficient and readable. Uses modern SQL features like window functions which are optimized in most SQL engines. It clearly separates steps for computing differences and grouping.

### Final Conclusion:
- The window function-based grouping method is robust, scalable, and best suited for this kind of sequence detection. It is the recommended approach for solving this problem cleanly and efficiently.
