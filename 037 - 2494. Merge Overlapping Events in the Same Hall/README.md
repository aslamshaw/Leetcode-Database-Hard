# LeetCode Problem 2494: Merge Overlapping Events in the Same Hall

## Problem Description

You are given a table named `HallEvents`, where each row represents an event held in a specific hall with a `start_day` and an `end_day`. The events may overlap, and your task is to merge those overlapping events in the same hall. Two events overlap if they have at least one common day.

### Tables:

#### HallEvents Table:

| hall_id | start_day   | end_day     |
|---------|-------------|-------------|
| 1       | 2023-01-13  | 2023-01-14  |
| 1       | 2023-01-14  | 2023-01-17  |
| 1       | 2023-01-18  | 2023-01-25  |
| 2       | 2022-12-09  | 2022-12-23  |
| 2       | 2022-12-13  | 2022-12-17  |
| 3       | 2022-12-01  | 2023-01-30  |

### Expected Output:

| hall_id | start_day   | end_day     |
|---------|-------------|-------------|
| 1       | 2023-01-13  | 2023-01-17  |
| 1       | 2023-01-18  | 2023-01-25  |
| 2       | 2022-12-09  | 2022-12-23  |
| 3       | 2022-12-01  | 2023-01-30  |

### Problem Constraints:
- The table does not have a primary key, and it may contain duplicates.
- The result should contain merged events for each hall, where the `start_day` is the earliest and the `end_day` is the latest in each overlapping group.

---

## Solution Approaches

### Approach 1: Using LAG and Grouping

This approach involves detecting overlapping events by checking the end date of the previous event and comparing it to the start date of the current event. If the current event starts before or when the previous one ends, we treat them as overlapping and group them together.

#### SQL Query:
```sql
WITH SortedEvents AS (
  SELECT *, 
         LAG(end_day) OVER (PARTITION BY hall_id ORDER BY start_day) AS prev_end
  FROM HallEvents
),
GroupedEvents AS (
  SELECT *, 
         SUM(CASE WHEN start_day > prev_end OR prev_end IS NULL THEN 1 ELSE 0 END) 
         OVER (PARTITION BY hall_id ORDER BY start_day) AS group_id
  FROM SortedEvents
)
SELECT hall_id, 
       MIN(start_day) AS start_day, 
       MAX(end_day) AS end_day
FROM GroupedEvents 
GROUP BY hall_id, group_id;
```

#### Explanation:
- The `LAG` window function is used to get the `end_day` of the previous event for each hall, allowing us to check if two consecutive events overlap.
- If the `start_day` of the current event is greater than the `end_day` of the previous event, we mark it as the start of a new group.
- The `SUM` function is then used to create a running group identifier that tracks consecutive overlapping events.
- Finally, we group the events by `hall_id` and the generated `group_id` to calculate the earliest `start_day` and the latest `end_day` for each group.

---

### Importance of Using `SUM()` with `OVER()` for `group_id`

The use of `SUM()` with `OVER()` is critical for correctly identifying and grouping overlapping events. Here's why it's important:

- **Grouping Without `SUM()`**: If you use only a `CASE` statement with `PARTITION BY hall_id` and `GROUP BY`, you would attempt to group rows based only on whether the `start_day` of an event is after the `end_day` of the previous event. However, this could lead to incorrect groupings, especially when there are multiple consecutive overlapping events. For example, even if three consecutive events overlap, you may accidentally treat them as separate groups due to breaks between them.

- **Using `SUM()` with `OVER()`**: The cumulative sum (`SUM()`) helps to "accumulate" the group identifier, ensuring that all consecutive overlapping events are assigned the same group ID. When we detect a gap (i.e., when the `start_day` of an event is later than the `end_day` of the previous event), we increment the group ID. This ensures that all overlapping events are grouped together correctly. The cumulative sum keeps track of these breaks and accumulates the correct group ID for each row, preventing erroneous separations of overlapping events.

In short, `SUM()` with `OVER()` provides an efficient and reliable way to generate a `group_id` that correctly identifies groups of overlapping events, which is crucial for merging those events into a single time interval.

---

## Performance Analysis

### Method 1: Using LAG and Grouping

- **Time complexity**: O(n), where n is the number of events in the table. The solution processes each row once, and the window functions (`LAG`, `SUM`) work in linear time.
- **Space complexity**: O(n), as we store intermediate results and generate a running group identifier for each row.

---

## Conclusion

- **Approach 1**: This method efficiently handles overlapping events using window functions and grouping. It is scalable and handles duplicates effectively by merging them into appropriate groups.

### Final Conclusion:
- This approach is highly effective for merging overlapping events, providing both an efficient solution and clarity in implementation.
