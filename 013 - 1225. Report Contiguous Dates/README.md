# LeetCode Problem 1225: Report Contiguous Dates

## Problem Description

You are tasked with generating a report of the continuous intervals for tasks, where each task either succeeded or failed. The tasks are recorded as daily events in two separate tables: one for failed tasks and one for succeeded tasks. Each table contains the date on which tasks either failed or succeeded. Your goal is to identify and group the contiguous periods of time in which tasks were either all successful or all failed. The report should include the start and end dates for each interval and the state of the task during that period.

### Tables:

#### Failed Table:

| fail_date   |
|-------------|
| 2019-01-01  |
| 2019-01-02  |
| 2019-01-05  |
| 2019-01-06  |

#### Succeeded Table:

| success_date |
|--------------|
| 2019-01-03   |
| 2019-01-04   |
| 2019-01-07   |
| 2019-01-08   |

### Expected Output:

| period_state | start_date | end_date   |
|--------------|------------|------------|
| succeeded    | 2019-01-03 | 2019-01-04 |
| failed       | 2019-01-01 | 2019-01-02 |
| succeeded    | 2019-01-07 | 2019-01-08 |
| failed       | 2019-01-05 | 2019-01-06 |

### Problem Constraints:
- The system operates one task per day.
- The input contains two tables, "Failed" and "Succeeded," representing the days on which tasks failed or succeeded, respectively.
- The report should cover the period from 2019-01-01 to 2019-12-31.
- The output must list each continuous period of either all failed or all succeeded tasks in chronological order.

---

## Solution Approach: Using Gap Grouping

### Problem Breakdown:

1. **Union of Dates**: 
   Combine the `Failed` and `Succeeded` dates into a single list, marking each entry with its corresponding `period_state` (`failed` or `succeeded`).
   
2. **Row Numbering**:
   Assign a unique `ROW_NUMBER()` to each row based on the order of the dates.

3. **Gap Grouping**:
   Using the `ROW_NUMBER()` and the `id`, we identify contiguous groups (or gaps). The idea is to create a `group_id` that groups consecutive dates with the same `period_state`. When the state changes (from `succeeded` to `failed` or vice versa), the group will also change.

4. **Group Identification**:
   The difference between the `id` and the `ROW_NUMBER()` (calculated for each `period_state`) helps us identify contiguous intervals. A new group is formed whenever there is a gap between consecutive dates.

### Approach Details:

1. **UNION** the `Failed` and `Succeeded` dates into a single list.
2. **Assign ROW_NUMBER()** to order the dates and create partitions by `period_state`.
3. **Gap Grouping**: Use `id - ROW_NUMBER()` to create a `group_id` that identifies each group of consecutive dates.
4. **Group Results**: For each `group_id`, calculate the `start_date` (minimum date) and `end_date` (maximum date).

#### SQL Query:
```sql
    WITH Date_Status AS (
        SELECT fail_date AS dates, 'failed' AS period_state FROM Failed
        WHERE '2019-01-01' <= fail_date AND fail_date <= '2019-12-31'
        UNION ALL
        SELECT success_date, 'succeeded' AS period_state FROM Succeeded
        WHERE '2019-01-01' <= success_date AND success_date <= '2019-12-31' order by dates),
    Ranked_Rows AS (SELECT *, ROW_NUMBER() OVER (ORDER BY dates) AS id FROM Date_Status),
    Grouped_Rows AS (SELECT *, id - ROW_NUMBER() OVER (PARTITION BY period_state ORDER BY id) AS group_id 
                     FROM Ranked_Rows)
    SELECT period_state, MIN(dates) AS start_date, MAX(dates) AS end_date FROM Grouped_Rows
    GROUP BY group_id, period_state ORDER BY start_date;
```

### Explanation of the Approach:

1. **Date Status Union**: 
   We combine the dates from both the `Failed` and `Succeeded` tables, while also indicating whether each date corresponds to a `failed` or `succeeded` task.

2. **Row Numbering**: 
   We assign a sequential number to each date in the combined list based on the order of dates.

3. **Identifying Groups**: 
   By calculating the difference between `id` and `ROW_NUMBER()` for each `period_state`, we group consecutive dates together. This allows us to identify gaps when the task status changes (from succeeded to failed or vice versa).

4. **Group Results**: 
   For each group (defined by `group_id`), we calculate the `start_date` (the earliest date) and `end_date` (the latest date) for that period.

### Key Concepts Used:
- **Union** of tables: Combines rows from multiple sources (Failed and Succeeded).
- **ROW_NUMBER()**: Assigns a sequential number to each row.
- **Gap Grouping**: A method used to identify contiguous sequences of data by looking for "gaps" between rows.
- **Grouping Logic**: Uses the difference between `id` and `ROW_NUMBER()` to identify when the state changes and when new groups begin.

---

## Performance Analysis

### Time Complexity:
- **Union Operation**: Combining the two tables requires O(N) time, where N is the total number of rows across both tables.
- **ROW_NUMBER() Assignment**: This operation involves sorting, which takes O(N log N) time.
- **Grouping and Aggregation**: Grouping by `group_id` and calculating the minimum and maximum dates takes O(N) time.

Thus, the overall time complexity is **O(N log N)**, where N is the total number of records in the `Failed` and `Succeeded` tables.

### Space Complexity:
- The space complexity is **O(N)** as we store both the `Failed` and `Succeeded` dates along with their `period_state`, `ROW_NUMBER()`, and `group_id`.

---

## Conclusion

- **Gap Grouping** is an efficient and powerful technique to identify continuous periods of similar states (failed or succeeded) in a series of dates.
- The solution works by combining the dates from both tables and using `ROW_NUMBER()` to identify groups of consecutive dates with the same state.
- This approach ensures that the solution is efficient both in terms of time and space, making it well-suited for large datasets.
