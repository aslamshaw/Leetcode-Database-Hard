# LeetCode Problem 2991: Top Three Wineries

## Problem Description

You are given a table named `Wineries` containing data about different wineries, including the country they are located in, their name, and their score (points). The goal is to determine the **top three wineries in each country** based on their **total accumulated points**.

The rules for ranking are:
- Wineries are ranked by **total points** in descending order.
- If two or more wineries have the same total points, they are ordered by **winery name in ascending order**.
- If a country has fewer than three wineries, output `'No second winery'` or `'No third winery'` accordingly.

Return the final result with one row per country, showing the top three wineries, ordered alphabetically by country name.

---

### Tables:

#### Wineries Table:

| id  | country   | points | winery         |
|-----|-----------|--------|----------------|
| 200 | France    | 90     | ChateauLyon    |
| 301 | France    | 70     | BordeauxPrime  |
| 117 | France    | 70     | ChateauLyon    |
| 410 | Brazil    | 50     | RioReserve     |
| 125 | Brazil    | 50     | SaoVino        |
| 330 | Brazil    | 60     | AmazonVintners |
| 518 | Canada    | 82     | MapleCellars   |
| 666 | Canada    | 55     | SnowPeak       |
| 999 | Chile     | 100    | AndesSelect    |

### Expected Output:

| country | top_winery           | second_winery     | third_winery        |
|---------|----------------------|-------------------|---------------------|
| Brazil  | AmazonVintners (60)  | RioReserve (50)   | SaoVino (50)        |
| Canada  | MapleCellars (82)    | SnowPeak (55)     | No third winery     |
| Chile   | AndesSelect (100)    | No second winery  | No third winery     |
| France  | ChateauLyon (160)    | BordeauxPrime (70)| No third winery     |

### Problem Constraints:
- `id` is a unique identifier for each record.
- The same winery may appear multiple times and must have their `points` **summed up per country**.
- Output exactly one row per country.
- The output should always include three columns for winery rankings, even if some are placeholders.

---

## Solution Approaches

### Approach 1: Aggregation and Ranking with DENSE_RANK

This approach first computes the total points for each winery in each country, then assigns a rank based on the score.

#### SQL Query:
```sql
WITH WineryPoints AS (
    SELECT country, winery, SUM(points) AS points
    FROM Wineries
    GROUP BY country, winery),

RankedWineries AS (
    SELECT country, 
           CONCAT(winery, ' (', points, ')') AS winery,
           DENSE_RANK() OVER (PARTITION BY country ORDER BY points DESC, winery) AS rk
    FROM WineryPoints)

SELECT country,
       MAX(CASE WHEN rk = 1 THEN winery ELSE NULL END) AS top_winery,
       IFNULL(MAX(CASE WHEN rk = 2 THEN winery ELSE NULL END), 'No second winery') AS second_winery,
       IFNULL(MAX(CASE WHEN rk = 3 THEN winery ELSE NULL END), 'No third winery') AS third_winery
FROM RankedWineries
GROUP BY country
ORDER BY country;
```

#### Explanation:
- **Step 1:** Use a `GROUP BY` to calculate the sum of points for each `winery` within each `country`.
- **Step 2:** Apply `DENSE_RANK()` window function to assign ranks within each country, ordering by `points DESC` and `winery ASC`.
- **Step 3:** Extract top 1st, 2nd, and 3rd ranked wineries for each country using conditional aggregation.
- **Step 4:** Use `IFNULL()` to fill in `'No second winery'` or `'No third winery'` if the country has fewer than three wineries.

This method ensures correct tie-breaking and formatting of output names with scores using `CONCAT()`.

---

## Performance Analysis

### Method 1 (Aggregation and Ranking with DENSE_RANK):

- **Time complexity**: `O(n log n)` — mainly due to sorting within the window function per country.
- **Space complexity**: `O(n)` — for temporary storage of grouped and ranked results.

---

## Conclusion

- **Method 1**: Provides a robust and scalable way to compute the top wineries per country using SQL's grouping and windowing capabilities. Handles edge cases where countries have fewer than three wineries gracefully.

### Final Conclusion:
- The aggregation and ranking approach effectively solves the problem and handles ordering, ties, and placeholder logic in a clean and efficient manner.
