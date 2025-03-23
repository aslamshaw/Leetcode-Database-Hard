# LeetCode Problem 1097: Game Play Analysis V

## Problem Description

Given a table `Activity`, write a solution to calculate for each `install date`, the number of players that installed the game on that day and their day one retention.

### Tables:

#### Activity Table:

| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|
| 1         | 3         | 2018-02-15 | 4            |
| 1         | 3         | 2018-02-16 | 7            |
| 2         | 2         | 2019-07-10 | 2            |
| 3         | 1         | 2018-02-15 | 0            |
| 3         | 4         | 2018-03-05 | 5            |

### Expected Output:

| install_dt | installs | Day1_retention |
|------------|----------|----------------|
| 2018-02-15 | 2        | 0.50           |
| 2019-07-10 | 1        | 0.00           |

### Problem Constraints:
- `(player_id, event_date)` is the primary key of this table.
- `install_date` refers to the first day a player logged in.
- The day one retention is calculated by the ratio of players who logged back in the next day after their `install_date` divided by the number of players who installed on that day.

---

## Solution Approaches

### Approach 1: Using `ROW_NUMBER()` and `DATEDIFF()` to Calculate Day 1 Retention

#### SQL Query:
```sql
WITH Installs AS (SELECT *, MIN(event_date) OVER (PARTITION BY player_id) AS first_install, 
                  CASE WHEN DATEDIFF(event_date, MIN(event_date) OVER (PARTITION BY player_id)) = 1 THEN 1 
                       ELSE 0 END AS is_next_day_install FROM Activity)
SELECT first_install, COUNT(DISTINCT player_id) AS installs,
ROUND(SUM(is_next_day_install) / COUNT(DISTINCT player_id), 2) AS Day1_retention FROM Installs GROUP BY first_install;
```

#### Explanation:
- First, we need to determine each player's `install_date`, which is the first day they logged in. We can achieve this by using `MIN(event_date) OVER (PARTITION BY player_id)` to get the earliest `event_date` for each player.
- After identifying the `install_date`, we can check if the player logged in the next day by calculating the difference between their `event_date` and their `first_install` date using `DATEDIFF()`. If the difference equals `1`, then this player has logged back in the day after their installation, contributing to day 1 retention.
- The day 1 retention is then calculated as the ratio of players who logged in the next day to the total number of players who installed the game on that day.
- To avoid counting duplicate players, we use `DISTINCT` to count unique players only.

---

## Performance Analysis

### Method 1 (Using `ROW_NUMBER()` and `DATEDIFF()`):

- **Time complexity**: O(N) (due to window function `MIN()` and `DATEDIFF()` calculations)
- **Space complexity**: O(N) (for storing intermediate results with `ROW_NUMBER()` and calculated retention flags)

---

## Conclusion

- **Method 1**: Efficient for calculating day 1 retention, with good handling of duplicates using `DISTINCT`.
- **Final Recommendation**: This method is optimal for calculating day 1 retention while considering varying numbers of installs per day.
