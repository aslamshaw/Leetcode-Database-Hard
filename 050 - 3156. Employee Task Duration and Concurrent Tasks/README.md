# LeetCode Problem 3156: Employee Task Duration and Concurrent Tasks

## Problem Description
You are given a table named **Tasks** that contains information about employee tasks, including the task identifier, employee identifier, start time, and end time. The goal is to calculate the total duration of tasks for each employee and the maximum number of concurrent tasks an employee handled at any point in time. The total duration should be rounded down to the nearest number of full hours.

### Tables:

#### Tasks Table:
| task_id | employee_id | start_time          | end_time            |
|--------|-------------|---------------------|---------------------|
| 1      | 1001        | 2023-05-01 08:00:00 | 2023-05-01 09:00:00 |
| 2      | 1001        | 2023-05-01 08:30:00 | 2023-05-01 10:30:00 |
| 3      | 1001        | 2023-05-01 11:00:00 | 2023-05-01 12:00:00 |
| 7      | 1001        | 2023-05-01 13:00:00 | 2023-05-01 15:30:00 |
| 4      | 1002        | 2023-05-01 09:00:00 | 2023-05-01 10:00:00 |
| 5      | 1002        | 2023-05-01 09:30:00 | 2023-05-01 11:30:00 |
| 6      | 1003        | 2023-05-01 14:00:00 | 2023-05-01 16:00:00 |

### Expected Output:
| employee_id | total_task_hours | max_concurrent_tasks |
|------------|------------------|----------------------|
| 1001       | 6                | 2                    |
| 1002       | 2                | 2                    |
| 1003       | 2                | 1                    |

### Problem Constraints:
- Task durations must be rounded down to the nearest full hour.
- Tasks may overlap for the same employee.
- The result must be sorted by **employee_id** in ascending order.

---

## Solution Approaches

### Approach 1: Event-based Interval Counting
This approach treats task start and end times as events. We record each event as either a task start (`+1`) or task end (`-1`). Then we calculate the duration of concurrent tasks and the maximum concurrency for each employee.

#### SQL Query:
```sql
WITH TaskEvents AS (
    SELECT employee_id, start_time AS event_time, 1 AS event_type FROM Tasks
    UNION ALL
    SELECT employee_id, end_time AS event_time, -1 AS event_type FROM Tasks
),
ConcurrentTasks AS (
    SELECT employee_id, event_time, event_type,
           LEAD(event_time) OVER (PARTITION BY employee_id ORDER BY event_time, event_type DESC) AS next_time,
           SUM(event_type) OVER (PARTITION BY employee_id ORDER BY event_time, event_type DESC) AS running_tasks
    FROM TaskEvents
)
SELECT employee_id, 
      FLOOR(SUM(CASE WHEN running_tasks > 0 THEN TIMESTAMPDIFF(MINUTE, event_time, next_time)
              ELSE 0 END) / 60) AS total_task_hours,
      MAX(running_tasks) AS max_concurrent_tasks
FROM ConcurrentTasks GROUP BY employee_id
```

#### Explanation:
1. **Task Events:** Gather all start and end times, marking them with `+1` or `-1` respectively.
2. **Concurrent Tasks Calculation:** Use `LEAD()` to calculate the duration between consecutive events and `SUM()` to calculate active tasks.
3. **Aggregate Results:** Group the results by `employee_id` to calculate the total task hours and the peak concurrency.

---

## Performance Analysis

### Method 1 (Event-based Interval Counting):
- **Time complexity:** `O(N log N)` - Due to sorting of events.
- **Space complexity:** `O(N)` - Storing events and intermediate calculations.

---

## Conclusion
- **Method 1:** Efficient for handling overlapping tasks and calculating both the total duration and peak concurrency. The use of window functions and aggregation helps maintain performance.

### Final Conclusion:
The event-based interval counting method is effective in calculating both the total task duration and the maximum concurrency for each employee, leveraging SQL window functions efficiently.
