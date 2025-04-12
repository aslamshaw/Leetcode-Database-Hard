# LeetCode Problem 3401: Find Circular Gift Exchange Chains

## Problem Description

You're given a table that records gift exchanges between employees during a Secret Santa event. Each row documents which employee gave a gift to whom, along with the value of that gift. The goal is to identify **circular chains** of exchanges, where:

- Every employee gives a gift to exactly one other employee.
- Every employee receives a gift from exactly one other employee.
- The exchange forms a closed loop or cycle.

For each circular chain detected, you must calculate:
- The total number of people involved in the chain (`chain_length`)
- The cumulative value of the gifts in that chain (`total_gift_value`)

The final result should present these chains in descending order by:
1. Chain length
2. Total gift value (if lengths are equal)

### Tables:

#### SecretSanta Table:

| giver_id | receiver_id | gift_value |
|----------|-------------|------------|
| 10       | 12          | 50         |
| 12       | 14          | 30         |
| 14       | 10          | 40         |
| 20       | 25          | 35         |
| 25       | 20          | 25         |

### Expected Output:

| chain_id | chain_length | total_gift_value |
|----------|--------------|------------------|
| 1        | 3            | 120              |
| 2        | 2            | 60               |

### Problem Constraints:
- Each `giver_id` gives a gift to only one `receiver_id`.
- Every person in a circular chain must be part of both a giving and receiving relationship.
- `(giver_id, receiver_id)` is a unique pair in the table.
- The dataset is guaranteed to form one or more valid circular chains.
- Output must be sorted by chain length and total gift value, both in descending order.

---

## Solution Approaches

### Approach 1: Recursive Chain Traversal

This approach uses a **recursive common table expression (CTE)** to trace potential gift exchange paths, starting from each giver and following the chain of receivers. The goal is to identify when the path loops back to the original giver—thus forming a circular chain.

#### SQL Query:
```sql
WITH RECURSIVE Chains AS (
    SELECT *, giver_id AS start_id 
    FROM SecretSanta
    UNION ALL
    SELECT sc.giver_id, sc.receiver_id, sc.gift_value, c.start_id
    FROM Chains c 
    JOIN SecretSanta sc 
        ON c.receiver_id = sc.giver_id 
        AND c.start_id != sc.giver_id),

RankedChains AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY giver_id, receiver_id ORDER BY start_id) AS rk
    FROM Chains),

AggregatedChains AS (
    SELECT COUNT(*) AS chain_length, SUM(gift_value) AS total_gift_value
    FROM (SELECT * 
          FROM RankedChains 
          WHERE rk = 1) s 
    GROUP BY start_id)

SELECT ROW_NUMBER() OVER (ORDER BY chain_length DESC, total_gift_value DESC) AS chain_id, 
       chain_length, 
       total_gift_value 
FROM AggregatedChains;
```

#### Explanation:
- A recursive CTE `Chains` starts with each gift exchange and follows the path of gift giving.
- The recursion continues by matching the `receiver_id` of one exchange to the `giver_id` of the next.
- It halts paths that would loop back before completing the cycle or revisit nodes prematurely.
- To eliminate duplicates caused by multiple paths detecting the same loop, each giver-receiver pair is ranked and filtered using `ROW_NUMBER()`.
- Aggregation is performed per unique cycle starting point (`start_id`) to compute:
  - The total number of people involved (`COUNT(*)`)
  - The sum of all gift values (`SUM(gift_value)`)
- Finally, the result is assigned a unique `chain_id` and sorted accordingly.

---

## Performance Analysis

### Method 1 (Recursive Chain Traversal):

- **Time complexity**: `O(n)` in the average case, where `n` is the number of rows; each path is traversed once per chain. In the worst case (if data isn't cyclic or is extremely sparse), it could approach `O(n^2)` due to recursion.
- **Space complexity**: `O(n)` due to CTE recursion stack and temporary storage for intermediate result sets.

---

## Conclusion

- **Method 1**: Efficient and scalable for moderate datasets. The use of recursive CTEs makes the logic elegant and avoids excessive joins or procedural logic. However, recursion depth and duplicate detection require careful handling.

### Final Conclusion:
The recursive CTE method is effective and concise for identifying and aggregating circular gift chains. It's the preferred approach when dealing with datasets that naturally lend themselves to path traversal and cyclic pattern detection.
