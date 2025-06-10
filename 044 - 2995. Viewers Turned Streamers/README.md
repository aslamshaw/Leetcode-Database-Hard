# LeetCode Problem 2995: Viewers Turned Streamers

## Problem Description

You are given a table called `Sessions` containing user activity data, where each row represents a session, including details such as the user ID, the session's start and end time, session ID, and session type (Viewer or Streamer). 

The task is to find the **number of streaming sessions** for each user, but only for those whose **first session was as a viewer**. 

The output should be ordered by the number of streaming sessions in descending order, and for users with the same number of sessions, by user ID in descending order.

### Tables:

#### Sessions Table:

| user_id | session_start       | session_end         | session_id | session_type |
|---------|---------------------|---------------------|------------|--------------|
| 101     | 2023-11-06 13:53:42 | 2023-11-06 14:05:42 | 375        | Viewer       |
| 101     | 2023-11-22 16:45:21 | 2023-11-22 20:39:21 | 594        | Streamer     |
| 102     | 2023-11-16 13:23:09 | 2023-11-16 16:10:09 | 777        | Streamer     |
| 102     | 2023-11-17 13:23:09 | 2023-11-17 16:10:09 | 778        | Streamer     |
| 101     | 2023-11-20 07:16:06 | 2023-11-20 08:33:06 | 315        | Streamer     |
| 104     | 2023-11-27 03:10:49 | 2023-11-27 03:30:49 | 797        | Viewer       |
| 103     | 2023-11-27 03:10:49 | 2023-11-27 03:30:49 | 798        | Streamer     |

### Expected Output:

| user_id | sessions_count |
|---------|----------------|
| 101     | 2              |

### Problem Constraints:
- `user_id` and `session_id` are unique for each session.
- The `session_type` is either `'Viewer'` or `'Streamer'`.
- A user must have their first session as a `'Viewer'` for it to be considered in the result.
- Output should be sorted by the number of streaming sessions, then by `user_id` in descending order.

---

## Solution Approaches

### Approach 1: Ranking and Filtering First Session as Viewer

This approach ranks the sessions for each user and then filters for users whose first session was as a viewer, followed by counting their streaming sessions.

#### SQL Query:
```sql
WITH FirstViewers AS (
    SELECT user_id
    FROM (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY session_start) AS rk
        FROM Sessions
    ) s 
    WHERE rk = 1 AND session_type = 'Viewer'
)
SELECT user_id, SUM(session_type = 'Streamer') AS sessions_count
FROM Sessions JOIN FirstViewers USING(user_id)
GROUP BY user_id
HAVING SUM(session_type = 'Streamer') > 0
ORDER BY sessions_count, user_id DESC;
```

#### Explanation:
- **Step 1:** Use `ROW_NUMBER()` to rank sessions for each user based on the session start time, ensuring that the first session for each user can be identified.
- **Step 2:** Pick `rk = 1` i.e. first session for each user and check if `session_type = 'Viewer'` i.e. whose first session is also as Viewer.
- **Step 3:** Count the number of `'Streamer'` sessions for users whose first session was a viewer.
- **Step 4:** Return the results, ordered by the number of streaming sessions and `user_id` in descending order.

---

## Performance Analysis

### Method 1 (Ranking and Filtering First Session as Viewer):

- **Time complexity**: `O(n log n)` — Sorting is done as part of the `ROW_NUMBER()` window function, which requires sorting the data by `session_start`.
- **Space complexity**: `O(n)` — Temporary storage for the ranked sessions.

---

## Conclusion

- **Method 1**: Efficiently handles the problem by utilizing window functions to rank the sessions and conditional aggregation to filter and count streaming sessions. This approach ensures that only users who started as viewers are included in the count, while also handling ordering and edge cases like no streaming sessions.

### Final Conclusion:
- This approach is optimal for this problem, ensuring the correct sessions are counted and users are ranked as required, while adhering to the constraints and expected output format.
