# LeetCode Problem 2173: Longest Winning Streak

## Problem Description

Given a table containing match outcomes for various players, the goal is to determine the longest sequence of consecutive wins ("Win") for each player. A streak is broken by either a "Draw" or a "Lose" result.

### Tables:

#### Matches Table:

| player_id | match_day  | result |
|-----------|------------|--------|
| 8         | 2022-03-01 | Win    |
| 8         | 2022-03-02 | Win    |
| 8         | 2022-03-05 | Win    |
| 8         | 2022-03-06 | Draw   |
| 8         | 2022-03-10 | Win    |
| 10        | 2022-03-03 | Lose   |
| 10        | 2022-03-05 | Lose   |
| 12        | 2022-04-01 | Win    |

### Expected Output:

| player_id | longest_streak |
|-----------|----------------|
| 8         | 3              |
| 10        | 0              |
| 12        | 1              |

### Problem Constraints:
- Each player may have multiple match entries across different dates.
- A streak of wins is interrupted by a loss or a draw.
- Output must include players with zero wins.
- `result` is an ENUM type: ('Win', 'Draw', 'Lose').

---

## Solution Approaches

### Approach 1: Win Streak Grouping via Row Number Difference

This approach utilizes a row-number based technique to group consecutive wins together efficiently.

#### SQL Query:
```sql
WITH GroupedMatches AS (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY player_id, match_day)
           - ROW_NUMBER() OVER (PARTITION BY player_id, result ORDER BY match_day) AS group_id
    FROM Matches
    ORDER BY player_id, match_day)
SELECT player_id, MAX(cnt) AS longest_streak
FROM (
    SELECT player_id, SUM(result = 'Win') AS cnt
    FROM GroupedMatches
    GROUP BY player_id, group_id) s
GROUP BY player_id;
```

#### Explanation:
- First, generate a row number over the full dataset ordered by `player_id` and `match_day` to provide a sequence for all matches per player.
- Then, generate another row number but partition by both `player_id` and the `result`, and again order by `match_day`.
- Subtracting these two row numbers gives a unique `group_id` that helps segment sequences of identical results (i.e., all consecutive "Win" results will share the same `group_id`).
- Group the results by both `player_id` and `group_id`, and use `SUM(result = 'Win')` to count only the "Win" entries within each group.
- Use `MAX(cnt)` grouped by `player_id` to find the longest streak of wins for each player.
- Using `SUM(result = 'Win')` instead of filtering for 'Win' using a `WHERE` clause ensures that even players with no wins are included in the final result with a streak of 0.
- The entire row-numbering and grouping logic is handled within a single CTE for simplicity and performance.

---

## Performance Analysis

### Method 1 (Win Streak Grouping via Row Number Difference):
- **Time complexity**: O(N log N), where N is the number of match records (due to window function sorting).
- **Space complexity**: O(N), used for intermediate storage in the CTE.

---

## Conclusion

- **Method 1**: Efficient and elegant way to group consecutive wins using window functions and simple arithmetic to simulate groupings.
  - **Strengths**: Includes all players even with no wins, minimal SQL logic, and good performance.
  - **Weaknesses**: Might be slightly less intuitive due to the windowing technique.

### Final Conclusion:
- The window function approach using `ROW_NUMBER` and group-wise aggregation provides a reliable and scalable solution to determine the longest winning streak per player. It is the preferred method for this problem due to its simplicity and ability to include all necessary cases.
