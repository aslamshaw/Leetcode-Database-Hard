# LeetCode Problem 3268: Find Overlapping Shifts II

## Problem Description

You are given a table named `EmployeeShifts` containing the work shifts for various employees. Each shift has a `start_time` and an `end_time`, and shifts may overlap in time.

Two shifts are considered **overlapping** if:
- They occur on the **same date**, and
- One shift's `end_time` is **later** than another shift's `start_time`.

Your task is to analyze the shifts **per employee** and return:
1. The **maximum number of overlapping shifts** that occur at any given time.
2. The **total duration (in minutes)** of all overlapping periods.

The result should be sorted in ascending order by `employee_id`.

### Tables:

#### EmployeeShifts Table:

| employee_id | start_time           | end_time             |
|-------------|----------------------|----------------------|
| 1001        | 2023-09-10 08:00:00  | 2023-09-10 16:00:00  |
| 1001        | 2023-09-10 14:00:00  | 2023-09-10 20:00:00  |
| 1001        | 2023-09-10 15:00:00  | 2023-09-10 22:00:00  |
| 1002        | 2023-09-10 09:00:00  | 2023-09-10 17:00:00  |
| 1002        | 2023-09-10 10:30:00  | 2023-09-10 18:30:00  |
| 1003        | 2023-09-10 07:00:00  | 2023-09-10 15:00:00  |

### Expected Output:

| employee_id | max_overlapping_shifts | total_overlap_duration |
|-------------|-------------------------|--------------------------|
| 1001        | 3                       | 540                      |
| 1002        | 2                       | 390                      |
| 1003        | 1                       | 0                        |

### Problem Constraints:
- Shifts are defined by their `start_time` and `end_time`.
- Overlaps are only considered on the **same calendar date** (ignoring multi-day overlaps).
- Each pair of overlapping shifts is only counted once for duration.

---

## Solution Approaches

### Approach 1: Self Join for Duration + Event-Based Tracking for Overlap Count

This solution combines two strategies:
1. **Pairwise Comparison via Self-Join** to determine overlapping durations.
2. **Event-based Sweep Line Algorithm** to find the maximum number of overlapping shifts.

#### SQL Query:
```sql
WITH TotalOverlapDuration AS (
    SELECT 
        es1.employee_id,
        SUM(TIMESTAMPDIFF(MINUTE, es2.start_time, LEAST(es1.end_time, es2.end_time))) AS total
    FROM EmployeeShifts es1
    JOIN EmployeeShifts es2
        ON es1.employee_id = es2.employee_id
        AND DATE(es1.start_time) = DATE(es2.start_time)
        AND es1.end_time > es2.start_time
        AND es1.start_time < es2.start_time
    GROUP BY es1.employee_id),

ShiftEvents AS (
    SELECT employee_id, start_time AS event_time, 1 AS event_type FROM EmployeeShifts
    UNION ALL
    SELECT employee_id, end_time AS event_time, -1 AS event_type FROM EmployeeShifts),

ShiftOverlap AS (
    SELECT 
        employee_id, 
        MAX(running_shifts) AS max_overlapping_shifts
    FROM (
        SELECT 
            employee_id,
            event_time,
            SUM(event_type) OVER (PARTITION BY employee_id ORDER BY event_time) AS running_shifts
        FROM ShiftEvents) sub
    GROUP BY employee_id)

SELECT 
    employee_id,
    max_overlapping_shifts,
    IFNULL(total, 0) AS total_overlap_duration
FROM ShiftOverlap 
LEFT JOIN TotalOverlapDuration USING(employee_id)
ORDER BY employee_id;
```

#### Explanation:

##### Self Join for Overlap Duration:
- Each employee's shifts are compared against one another using a **self-join**.
- Overlapping conditions:
  - Same `employee_id`
  - Same date (`DATE(start_time)`)
  - Shift A (`es1`) ends after Shift B (`es2`) starts
  - Shift A starts before Shift B (to avoid duplicate comparisons)
- Duration is calculated using:
  - `TIMESTAMPDIFF(MINUTE, es2.start_time, LEAST(es1.end_time, es2.end_time))`

##### Event-Based Counting for Max Overlaps:
- Treats shift starts and ends as events:
  - `+1` when a shift starts
  - `-1` when a shift ends
- Events are sorted chronologically.
- Uses a **running sum (window function)** to track current concurrent shifts.
- The maximum value of this sum gives the **peak number of overlapping shifts**.

---

## Performance Analysis

### Method 1 (Self Join + Event-Based Tracking):

- **Time complexity**:
  - Self Join: O(N²) per employee due to pairwise comparisons.
  - Event Tracking: O(N log N) due to sorting of events.
- **Space complexity**: O(N) for intermediate CTEs and event storage.

---

## Conclusion

- **Self Join**: Accurately computes overlap durations but can be expensive for large datasets due to O(N²) comparisons.
- **Event-Based Counting**: Efficient for identifying peak overlaps and scales better due to O(N log N) sorting rather than O(N²) joins.

### Final Conclusion:
- Combining both methods provides a **complete and accurate solution**: Event-based for efficient overlap counting and self-join for exact overlap durations.
