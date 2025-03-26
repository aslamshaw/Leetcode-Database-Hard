# LeetCode Problem 1336: Number of Transactions per Visit

## Problem Description

A bank wants to analyze the number of transactions made by customers during a single visit. The goal is to determine how many visits had 0, 1, or more transactions and how many customers correspond to each transaction count.

### Tables:

#### **Visits Table:**
This table stores customer visits to the bank.

| user_id | visit_date |
|---------|------------|
| 3       | 2023-06-01 |
| 5       | 2023-06-02 |
| 14      | 2023-06-01 |
| 21      | 2023-06-04 |
| 3       | 2023-06-02 |
| 5       | 2023-06-04 |
| 3       | 2023-06-05 |
| 11      | 2023-06-10 |
| 18      | 2023-06-15 |
| 17      | 2023-06-20 |

#### **Transactions Table:**
This table records transactions made by customers.

| user_id | transaction_date | amount |
|---------|------------------|--------|
| 3       | 2023-06-02       | 200    |
| 5       | 2023-06-04       | 50     |
| 11      | 2023-06-10       | 300    |
| 3       | 2023-06-05       | 25     |
| 18      | 2023-06-15       | 60     |
| 18      | 2023-06-15       | 75     |
| 17      | 2023-06-20       | 20     |
| 18      | 2023-06-15       | 100    |

### Expected Output:

| transactions_count | visits_count |
|--------------------|-------------|
| 0                 | 3           |
| 1                 | 4           |
| 2                 | 0           |
| 3                 | 1           |

### Problem Constraints:
- Every transaction is tied to a valid visit in the `Visits` table.
- The `Visits` table may contain customers who did not make any transactions.
- The result should include all possible `transactions_count` values up to the maximum transactions done by any customer in one visit, even if no visits had that transaction count.

---

## Solution Approaches

### Approach 1: Using Transaction Grouping

#### SQL Query:
```sql
WITH RECURSIVE 
ZeroVisitCount AS (
    SELECT 0 AS transactions_count, COUNT(*) AS visits_count
    FROM Visits v LEFT JOIN Transactions t 
    ON v.user_id = t.user_id AND v.visit_date = t.transaction_date WHERE t.user_id IS NULL),
SameDayTransactions AS (
    SELECT user_id, transaction_date, COUNT(*) AS transactions_count 
    FROM Transactions GROUP BY user_id, transaction_date),
NoIntermediateTC AS (
    SELECT * FROM ZeroVisitCount 
    UNION ALL
    SELECT transactions_count, COUNT(*) AS visits_count FROM SameDayTransactions GROUP BY transactions_count),
IntermediateTC AS (
    SELECT 0 AS transactions_count, (SELECT MAX(transactions_count) FROM NoIntermediateTC) AS mtc
    UNION ALL
    SELECT transactions_count + 1, mtc FROM IntermediateTC WHERE transactions_count < mtc)
SELECT transactions_count, IFNULL(visits_count, 0) AS visits_count 
FROM IntermediateTC LEFT JOIN NoIntermediateTC USING (transactions_count) ORDER BY transactions_count;
```

#### Explanation:
1. Identify all visits that did not result in any transactions.
2. Count how many transactions were made per visit by grouping transactions by `user_id` and `transaction_date`.
3. Combine both datasets and count occurrences for each transaction count.
4. Generate missing transaction counts if there are gaps in the sequence to ensure continuity in reporting.
5. Order results by `transactions_count` for proper representation.

---

## Performance Analysis

### Method 1 (Transaction Grouping):
- **Time complexity**: O(N log N) (due to sorting and grouping operations)
- **Space complexity**: O(N) (to store intermediate results)

---

## Conclusion

- **Method 1**: Efficiently counts transactions per visit while ensuring all possible transaction counts are represented.
- Ensures missing transaction counts are included in the output, even if no visits had that exact count.
- Uses grouping and aggregation techniques to derive results.

### Final Conclusion:
This approach effectively provides insight into customer transaction behavior per visit while maintaining completeness in the output.
