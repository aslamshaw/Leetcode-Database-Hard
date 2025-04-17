# LeetCode Problem 3060: User Activities within Time Bounds

## Problem Description

You're given a `Sessions` table that tracks user activity sessions on a platform. Each session has a `user_id`, `session_start`, `session_end`, `session_id`, and a `session_type`, which can be either `'Viewer'` or `'Streamer'`.

The task is to identify all users who have had **at least one pair of consecutive sessions of the same session type**, where the **gap between the end of one session and the start of the next is no more than 12 hours**.

Return the list of such `user_id`s sorted in ascending order.

### Tables:

#### Sessions Table:

| user_id | session_start       | session_end         | session_id | session_type |
|---------|---------------------|---------------------|------------|--------------|
| 201     | 2023-10-01 07:00:00 | 2023-10-01 08:00:00 | 101        | Streamer     |
| 201     | 2023-10-01 09:00:00 | 2023-10-01 10:00:00 | 102        | Viewer       |
| 202     | 2023-10-01 12:00:00 | 2023-10-01 13:00:00 | 103        | Viewer       |
| 202     | 2023-10-01 14:00:00 | 2023-10-01 15:00:00 | 104        | Viewer       |
| 203     | 2023-10-02 08:00:00 | 2023-10-02 10:00:00 | 105        | Streamer     |
| 203     | 2023-10-02 20:00:00 | 2023-10-02 22:00:00 | 106        | Streamer     |
| 204     | 2023-10-03 07:00:00 | 2023-10-03 08:00:00 | 107        | Viewer       |
| 204     | 2023-10-04 07:00:00 | 2023-10-04 08:00:00 | 108        | Viewer       |

### Expected Output:

| user_id |
|---------|
| 202     |
| 203     |

### Problem Constraints:
- The time gap between sessions is calculated in hours.
- Only sessions of the **same type** count toward the consecutive requirement.
- Consecutive sessions are considered based on `session_start` time (and `session_id` for tie-breaker).
- Result should be ordered by `user_id` ascending.
- `session_id` is unique.

---

## Solution Approaches

### Approach 1: Using ROW_NUMBER and Self-Join

This method uses a Common Table Expression (CTE) to assign a row number (`ROW_NUMBER`) to each session per user, ordered by session start time. It then performs a self-join on the same user where the second session immediately follows the first (i.e., row numbers differ by 1), and both sessions share the same `session_type` with a time gap of no more than 12 hours between them.

#### SQL Query:
```sql
WITH RankedSessions AS (
  SELECT *,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY session_start, session_id) AS rk
  FROM Sessions)
SELECT DISTINCT s1.user_id
FROM RankedSessions s1
JOIN RankedSessions s2
  ON s1.user_id = s2.user_id
AND s1.rk = s2.rk - 1
AND s1.session_type = s2.session_type
AND TIMESTAMPDIFF(HOUR, s1.session_end, s2.session_start) <= 12;
```

#### Explanation:
- The `RankedSessions` CTE generates ordered session rows per user.
- A self-join compares each session to its immediate next session.
- The `TIMESTAMPDIFF` function calculates the hour gap between `session_end` of the earlier session and `session_start` of the next.
- If the time gap is less than or equal to 12 and the session types match, the user qualifies.

### Approach 2: Using LAG and Aggregation

This approach leverages the `LAG()` window function to access the previous session's end time and type for each session. Then it groups by `user_id` and uses a `HAVING` clause to count how many times a user had a session where the current and previous types matched, and the gap was 12 hours or less.

#### SQL Query:
```sql
WITH PrevSessions AS (
  SELECT *,
         LAG(session_type) OVER (PARTITION BY user_id ORDER BY session_start, session_id) AS prev_type,
         LAG(session_end) OVER (PARTITION BY user_id ORDER BY session_start, session_id) AS prev_end
  FROM Sessions)
  
SELECT user_id
FROM PrevSessions 
GROUP BY user_id 
HAVING SUM(
    CASE 
      WHEN TIMESTAMPDIFF(HOUR, prev_end, session_start) <= 12 
           AND session_type = prev_type 
      THEN 1 ELSE 0 
    END) > 0;
```

#### Explanation:
- The `PrevSessions` CTE adds `prev_type` and `prev_end` for each session using `LAG()`.
- In the main query, we group by `user_id` and evaluate how many qualifying consecutive same-type sessions exist.
- If at least one exists, the user is included in the result.

---

## Performance Analysis

### Method 1 (Using ROW_NUMBER and Self-Join):

- **Time complexity**: O(n log n) — due to window function and join operation.
- **Space complexity**: O(n) — for the intermediate CTE and join buffer.

### Method 2 (Using LAG and Aggregation):

- **Time complexity**: O(n log n) — window function and grouping.
- **Space complexity**: O(n) — intermediate storage for LAG values and aggregates.

---

## Conclusion

- **Method 1**: Offers a clear and direct mapping of session order with exact control over session sequencing using `ROW_NUMBER`. The self-join is easy to reason about for pairs of sessions.
- **Method 2**: More concise and efficient in some scenarios since it avoids a join and leverages window functions along with a group aggregate to identify qualifying users.

### Final Conclusion:
- **Method 2** is generally preferable due to its simplicity and better performance in avoiding the join, though both approaches are valid and effective for solving the problem.
