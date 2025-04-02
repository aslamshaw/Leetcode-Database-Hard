# LeetCode Problem 1972: First and Last Call On the Same Day

## Problem Description
Given a `Calls` table that stores information about calls made between users, find the IDs of users who had the **first and last call with the same person on any given day**.

### Tables:

#### Calls Table:

| caller_id | recipient_id | call_time           |
|----------|--------------|---------------------|
| 3        | 6            | 2021-09-15 08:15:00  |
| 6        | 3            | 2021-09-15 22:45:13  |
| 9        | 1            | 2021-09-12 10:28:44  |
| 3        | 7            | 2021-09-14 03:05:22  |
| 5        | 9            | 2021-09-14 15:17:09  |
| 3        | 5            | 2021-09-14 20:30:05  |

### Expected Output:

| user_id |
|--------|
| 1      |
| 3      |
| 6      |
| 9      |

### Problem Constraints:
- The `(caller_id, recipient_id, call_time)` combination is a **primary key**.
- The call times are given in the `datetime` format.
- The result can be returned in any order.

---

## Solution Approaches

### Approach 1: Bidirectional Relationship and Ranking
This approach ensures that both perspectives of each call (caller and recipient) are taken into account by creating a **bidirectional relationship (BDR)** table. This table includes both `(caller, recipient)` and `(recipient, caller)` pairs.

#### SQL Query:
```sql
WITH BDR AS (SELECT caller_id, recipient_id, call_time FROM Calls 
             UNION ALL SELECT recipient_id, caller_id, call_time FROM Calls),
RankedCalls AS (SELECT caller_id, recipient_id, DATE(call_time) AS call_date,
                RANK() OVER (PARTITION BY caller_id, DATE(call_time) ORDER BY call_time) AS first_call_rk,
                RANK() OVER (PARTITION BY caller_id, DATE(call_time) ORDER BY call_time DESC) AS last_call_rk FROM BDR)
SELECT caller_id AS user_id FROM RankedCalls WHERE first_call_rk = 1 OR last_call_rk = 1 
GROUP BY caller_id, call_date HAVING COUNT(DISTINCT recipient_id) = 1;
```

#### Explanation:
- Create a BDR table with both perspectives of each call.
- Use `RANK()` to find the **first and last call times** for each person on a given day.
- Group the results and check if the **first and last call were made to the same recipient**.
- Only include users whose first and last calls on a specific day were with the **same person**.

---

## Performance Analysis

### Method 1 (Bidirectional Relationship and Ranking):
- **Time complexity**: O(n log n), due to sorting calls per user per day.
- **Space complexity**: O(n), for storing intermediate ranked results.

---

## Conclusion
- **Method 1**: Efficient for handling both perspectives of a call (caller and recipient) by leveraging bidirectional relationships and ranking.
- Final Recommendation: This approach is optimal since it guarantees covering all scenarios by treating the relationship as bidirectional.
