# LeetCode Problem 3384: Team Dominance by Pass Success

## Problem Description

You are given two tables: `Teams` and `Passes`. The task is to compute a **dominance score** for each team based on their performance in both halves of a football (soccer) match.

> ⚠️ **Note**: We do **not** care about the **consecutiveness** or **sequence** of passes — only whether each pass was **successful** or **intercepted** based on the team of the receiving player.

A match is split into:
- **First Half**: 00:00 to 45:00 (inclusive)
- **Second Half**: 45:01 to 90:00

### Dominance Score Rules:
- If a pass goes **to a teammate**, it's considered successful: **+1 point**
- If a pass goes **to an opponent**, it's considered intercepted: **-1 point**

You need to compute each team's **total dominance score per half**, regardless of pass sequence or continuity.

### Tables:

#### Teams Table:

| player_id | team_name |
|-----------|-----------|
| 1         | Arsenal   |
| 2         | Arsenal   |
| 3         | Arsenal   |
| 4         | Chelsea   |
| 5         | Chelsea   |
| 6         | Chelsea   |

#### Passes Table:

| pass_from | time_stamp | pass_to |
|-----------|------------|---------|
| 1         | 00:15      | 2       |
| 2         | 00:45      | 3       |
| 3         | 01:15      | 1       |
| 4         | 00:30      | 1       |
| 2         | 46:00      | 3       |
| 3         | 46:15      | 4       |
| 1         | 46:45      | 2       |
| 5         | 46:30      | 6       |

---

### Expected Output:

| team_name | half_number | dominance |
|-----------|-------------|-----------|
| Arsenal   | 1           | 3         |
| Arsenal   | 2           | 1         |
| Chelsea   | 1           | -1        |
| Chelsea   | 2           | 1         |

### Explanation:

- **First Half**:
  - Arsenal made 3 successful passes: 1→2, 2→3, 3→1 (+3)
  - Chelsea made 1 intercepted pass: 4→1 (-1)
- **Second Half**:
  - Arsenal: 2→3, 3→4 (intercepted), 1→2 → Total: +1 (2 successes, 1 interception)
  - Chelsea: 5→6 (+1)

---

## Solution Approaches

### Approach 1: Score Calculation with Team Joins

#### SQL Query:

```sql
WITH Scores AS (
    SELECT 
        t1.team_name AS team_name, 
        time_stamp, 
        CASE 
            WHEN time_stamp <= '45:00' THEN 1 
            ELSE 2 
        END AS half_number,
        CASE 
            WHEN t1.team_name = t2.team_name THEN 1 
            ELSE -1 
        END AS score 
    FROM Passes p
    JOIN Teams t1 ON p.pass_from = t1.player_id
    JOIN Teams t2 ON p.pass_to = t2.player_id)

SELECT 
    team_name, 
    half_number, 
    SUM(score) AS dominance 
FROM Scores 
GROUP BY team_name, half_number
ORDER BY team_name, half_number;
```

This solution uses a **CTE** to:
- Join the `Passes` table with the `Teams` table twice:
  - Once to find the **passing team (from)**
  - Once to find the **receiving team (to)**
- Based on whether both players belong to the same team, assign a score of **+1** or **-1**
- Determine which half the pass occurred in using the `time_stamp`
- Group by team and half number to compute the total dominance score

---

## Performance Analysis

### Method 1 (Join and Score):
- **Time complexity**: `O(n)` — Efficient linear scan with joins and group-by.
- **Space complexity**: `O(n)` — Temporary storage for computed scores and half numbers.

---

## Conclusion

- **Method 1** is a clean and efficient approach for this problem.
- It handles team identification, time partitioning, and scoring all within one readable and performant query.
- The use of conditional logic within SQL ensures clarity and precision in mapping real-world rules to database operations.

### Final Takeaway:
This approach provides a reliable way to evaluate team dominance without needing to track sequence or continuity of passes—just their success or failure based on team membership.
