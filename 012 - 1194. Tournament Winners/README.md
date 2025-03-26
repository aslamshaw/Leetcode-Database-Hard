# LeetCode Problem 1194: Tournament Winners

## Problem Description

In a tournament, players are grouped, and each match records the scores of two competing players. The objective is to determine the winner in each group based on the highest accumulated points. If multiple players have the same highest score, the player with the lowest `player_id` is considered the winner.

### Tables:

#### Players Table:

| player_id | group_id |
|-----------|----------|
| 12        | 1        |
| 22        | 1        |
| 32        | 1        |
| 42        | 1        |
| 11        | 2        |
| 33        | 2        |
| 55        | 2        |
| 21        | 3        |
| 41        | 3        |

#### Matches Table:

| match_id | first_player | second_player | first_score | second_score |
|----------|-------------|---------------|-------------|--------------|
| 1        | 12          | 42            | 4           | 1            |
| 2        | 32          | 22            | 2           | 3            |
| 3        | 32          | 12            | 3           | 1            |
| 4        | 41          | 21            | 6           | 4            |
| 5        | 33          | 55            | 2           | 2            |

### Expected Output:

| group_id | player_id |
|----------|----------|
| 1        | 12       |
| 2        | 33       |
| 3        | 41       |

### Problem Constraints:
- Each player belongs to only one group.
- Players only compete within their respective groups.
- Scores are non-negative integers.
- There are no duplicate match records.

---

## Solution Approaches

### Approach 1: Aggregation and Ranking

This approach involves calculating the total score for each player, associating them with their groups, and ranking them accordingly.

#### SQL Query:
```sql
WITH UniPlayers AS (SELECT first_player AS player_id, first_score AS score FROM Matches 
                    UNION ALL SELECT second_player, second_score FROM Matches),
PlayerScore AS (SELECT player_id, SUM(score) AS total_score FROM UniPlayers GROUP BY player_id),
RankedPlayers AS (SELECT group_id, player_id, total_score, 
                  DENSE_RANK() OVER (PARTITION BY group_id ORDER BY total_score DESC, player_id) AS rk
                  FROM PlayerScore JOIN Players USING(player_id))
SELECT group_id, player_id FROM RankedPlayers WHERE rk = 1;
```

#### Explanation:
1. **Combine Player Scores**: Use `UNION ALL` to aggregate scores from both `first_player` and `second_player` columns.
2. **Summarize Total Points**: Group by `player_id` to compute the total score.
3. **Join with Player Groups**: Merge results with the `Players` table to retrieve `group_id`.
4. **Rank Players Within Groups**: Utilize `DENSE_RANK()` to rank players based on descending total score and ascending `player_id`.
5. **Select the Top Player**: Extract players with rank `1` in each group.

---

## Performance Analysis

### Method 1 (Aggregation and Ranking):

- **Time Complexity**: `O(N log N)` (Sorting for ranking, with N being the number of matches)
- **Space Complexity**: `O(N)` (Storage for intermediate results and ranking computation)

---

## Conclusion

- **Method 1**: Efficiently determines the tournament winners with straightforward aggregation and ranking.
- **Final Conclusion**: This approach is optimal given the constraints and ensures accurate results by handling ties through sorting by `player_id`.
