# LeetCode Problem: 601. Human Traffic of Stadium

## Problem Description

Write a solution to display the records with three or more rows with consecutive `id`s, where the number of people is greater than or equal to 100 for each row.

Return the result table ordered by `visit_date` in ascending order.

### Tables:

#### Stadium Table:

| id  | visit_date | people |
|-----|------------|--------|
| 1   | 2017-01-01 | 10     |
| 2   | 2017-01-02 | 109    |
| 3   | 2017-01-03 | 150    |
| 4   | 2017-01-04 | 99     |
| 5   | 2017-01-05 | 145    |
| 6   | 2017-01-06 | 1455   |
| 7   | 2017-01-07 | 199    |
| 8   | 2017-01-09 | 188    |

### Expected Output:

| id  | visit_date | people |
|-----|------------|--------|
| 5   | 2017-01-05 | 145    |
| 6   | 2017-01-06 | 1455   |
| 7   | 2017-01-07 | 199    |
| 8   | 2017-01-09 | 188    |

### Problem Constraints:
- `id` is always increasing in the table.
- `visit_date` is unique for each entry.
- The number of people attending (`people`) must be at least 100 for a row to be considered.
- Only sequences of at least three consecutive `id`s with `people >= 100` should be included.

---

## Solution Approaches

### Approach 1: Gap Grouping with Window Functions
Gap grouping is a technique where we assign a group identifier to consecutive rows by computing the difference between the id and the ROW_NUMBER() within a partition, 
effectively grouping adjacent records with similar properties.

#### SQL Query:
```sql
WITH ClassStadium AS (SELECT *, CASE WHEN people < 100 THEN 'Less' ELSE 'Enough' END AS bool_people FROM Stadium),
GroupedRows AS (SELECT *, id - ROW_NUMBER() OVER (PARTITION BY bool_people ORDER BY id) AS group_id FROM ClassStadium),
ConsecutiveGroups AS (SELECT *, COUNT(*) OVER (PARTITION BY group_id, bool_people) AS cnt
                      FROM GroupedRows WHERE bool_people = 'Enough')
SELECT id, visit_date, people FROM ConsecutiveGroups WHERE cnt > 2 ORDER BY visit_date;
```

#### Explanation:
- Since `id` increases with `visit_date`, we can use it to determine consecutive records.
- We first classify each row based on whether the `people` count is at least 100 (`Enough`) or not (`Less`).
- Using the `ROW_NUMBER()` function, we assign row numbers to each partition based on `Enough` and order them by `id`.
- By subtracting `ROW_NUMBER()` from `id`, we get a `group_id` that identifies groups of consecutive rows where `people >= 100`.
- Finally, we count the number of rows in each `(group_id, bool_people)` partition and filter those where the count is at least 3.
- The final result is ordered by `visit_date`.

---

## Performance Analysis

### Method 1 (Gap Grouping with Window Functions):

- **Time complexity**: `O(N)`, since we are using window functions and filtering in linear time.
- **Space complexity**: `O(N)`, due to temporary tables used for classification and grouping.

---

## Conclusion

- **Method 1**: Efficient for structured datasets where `id` is sequential. Uses window functions to classify and group consecutive records effectively.

### Final Conclusion:
- The gap grouping approach with window functions provides an optimal solution, ensuring correct filtering and ordering with minimal complexity.
