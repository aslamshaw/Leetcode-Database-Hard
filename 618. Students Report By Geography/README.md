# LeetCode Problem 618: Students Report By Geography

## Problem Description

Given a table `Student`, write a query to pivot the `continent` column so that each student's name is sorted alphabetically and displayed under its corresponding continent. The output headers should be `America`, `Asia`, and `Europe`, respectively.

### Tables:

#### Student Table:

| name   | continent |
|--------|-----------|
| Alice  | America   |
| Bob    | Europe    |
| Chen   | Asia      |
| David  | America   |
| Ethan  | Asia      |
| Frank  | Europe    |

### Expected Output:

| America | Asia  | Europe |
|---------|------|--------|
| Alice   | Chen | Bob    |
| David   | Ethan | Frank  |

### Problem Constraints:
- The table may contain duplicate rows.
- The output should display student names alphabetically under their respective continents.
- The test cases ensure that America has at least as many students as either Asia or Europe.
- If the number of students is unknown for each continent, the solution should still work dynamically.

---

## Solution Approaches

### Approach 1: Using Row Number and Aggregation

#### SQL Query:
```sql
WITH RankedRows AS (SELECT ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name) AS rk, continent, name FROM Student)
SELECT MAX(CASE WHEN continent = 'America' THEN name ELSE NULL END) AS America,
       MAX(CASE WHEN continent = 'Asia' THEN name ELSE NULL END) AS Asia,
       MAX(CASE WHEN continent = 'Europe' THEN name ELSE NULL END) AS Europe
FROM RankedRows GROUP BY rk;
```

#### Explanation:
- Since there is no natural pivot column, create a row number (`rk`) for each continent using `ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name)`.
- Use this `rk` as the pivot column to group data.
- Aggregate student names using `MAX()` to ensure names appear in their respective columns.
- The `ORDER BY name` ensures names are sorted alphabetically within each continent.

### Follow-up Consideration:
- If the number of students in each continent is unknown, this approach still works because `ROW_NUMBER()` dynamically assigns row numbers for each partition.

---

## Performance Analysis

### Method 1 (Row Number and Aggregation):

- **Time complexity**: O(N log N) (due to sorting in `ROW_NUMBER()`)
- **Space complexity**: O(N) (storing row numbers and grouped results)

---

## Conclusion

- **Method 1**: Efficient and flexible, as it dynamically handles varying student counts.
- **Final Recommendation**: This method is optimal for pivoting categorical data without a predefined pivot column.
