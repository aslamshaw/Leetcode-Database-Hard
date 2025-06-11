# LeetCode Problem 3390: Longest Team Pass Streak

## Problem Description

The problem is to find the longest consecutive successful pass streak for each team during a match. A successful pass streak is defined as consecutive passes where both the pass_from and pass_to players belong to the same team. The streak ends when a pass is intercepted by a player from another team. The result should be ordered by the team name in ascending order.

### Tables:

#### Teams Table:

| player_id | team_name |
|-----------|-----------|
| 1         | Arsenal   |
| 2         | Arsenal   |
| 3         | Arsenal   |
| 4         | Arsenal   |
| 5         | Chelsea   |
| 6         | Chelsea   |
| 7         | Chelsea   |
| 8         | Chelsea   |

#### Passes Table:

| pass_from | time_stamp | pass_to |
|-----------|------------|---------|
| 1         | 00:05      | 2       |
| 2         | 00:07      | 3       |
| 3         | 00:08      | 4       |
| 4         | 00:10      | 5       |
| 6         | 00:15      | 7       |
| 7         | 00:17      | 8       |
| 8         | 00:20      | 6       |
| 6         | 00:22      | 5       |
| 1         | 00:25      | 2       |
| 2         | 00:27      | 3       |

#### Output:

| team_name | longest_streak |
|-----------|----------------|
| Arsenal   | 3              |
| Chelsea   | 4              |

### Problem Constraints:
- The solution should handle up to 10^5 rows in both the `Teams` and `Passes` tables efficiently.
- Time complexity and space complexity should be optimized for large datasets.

---

## Solution Approaches

### Approach 1: SQL Query with Window Functions and Grouping

This approach utilizes SQL window functions to group the passes by consecutive streaks and calculates the longest successful pass streak for each team.

#### SQL Query:
```sql
WITH TeamPasses AS (
    SELECT 
        t1.team_name AS team_from, 
        p.time_stamp, 
        t2.team_name AS team_to
    FROM Passes p 
    JOIN Teams t1 ON p.pass_from = t1.player_id 
    JOIN Teams t2 ON p.pass_to = t2.player_id
    WHERE t1.team_name = t2.team_name
),

GroupedPasses AS (
    SELECT 
        *, 
        ROW_NUMBER() OVER (ORDER BY time_stamp) - 
        ROW_NUMBER() OVER (PARTITION BY team_from ORDER BY time_stamp) AS group_id
    FROM TeamPasses
)

SELECT 
    team_name, 
    MAX(streak) AS longest_streak
FROM (
    SELECT 
        team_from AS team_name, 
        COUNT(*) AS streak
    FROM GroupedPasses 
    GROUP BY team_from, group_id
) s
GROUP BY team_name
ORDER BY team_name;

```

#### Explanation:
- **TeamPasses CTE**: First, we join the `Passes` table with the `Teams` table twice: once for the `pass_from` player and once for the `pass_to` player. We filter out only the passes where both players belong to the same team.
- **GroupedPasses CTE**: We use the `ROW_NUMBER()` function to generate a unique identifier for each consecutive streak of passes. This is achieved by partitioning the rows by team and sorting by timestamp. The difference between the `ROW_NUMBER()` values gives us a unique `group_id` for each streak.
- **Final SELECT**: We calculate the streak length by counting the number of consecutive passes per group. We then find the maximum streak for each team by using the `ROW_NUMBER()` function again and grouping by `team_name` to get the longest streak per team.

---

## Performance Analysis

### Method 1 (SQL Query with Window Functions and Grouping):

- **Time complexity**: The query uses window functions and grouping, which generally run in O(n log n) time complexity due to sorting operations. The sorting of data within partitions adds a log factor.
- **Space complexity**: The space complexity is O(n) as we are creating intermediate tables (`TeamPasses`, `GroupedPasses`), storing results for each pass and team.

---

## Conclusion

- **Method 1**: This approach is efficient because it leverages SQL window functions and grouping, which are designed for handling large datasets. The use of CTEs (Common Table Expressions) allows for better readability and modularity in the query.

### Final Conclusion:
- The provided SQL query approach is optimal for solving the problem, as it efficiently handles the grouping and streak calculation while minimizing the complexity of the solution.
