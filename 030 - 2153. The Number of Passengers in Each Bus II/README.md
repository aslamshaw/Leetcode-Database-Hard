# LeetCode Problem 2153: The Number of Passengers in Each Bus II

## Problem Description

At a bus station, buses and passengers arrive at different times. Each bus has a specific **arrival_time** and a limited **capacity** of empty seats. Passengers can board a bus if they arrive **at or before** the bus’s arrival time and if there is still available capacity on the bus.

If there are more waiting passengers than the available seats when a bus arrives, only the **capacity** number of passengers will be able to board the bus. The goal is to determine the number of passengers that board each bus and return the result ordered by **bus_id** in ascending order.

### Tables:

#### Buses Table:

| bus_id | arrival_time | capacity |
|--------|--------------|----------|
| 1      | 3            | 2        |
| 2      | 5            | 8        |
| 3      | 9            | 4        |

#### Passengers Table:

| passenger_id | arrival_time |
|-------------|--------------|
| 21          | 1            |
| 22          | 2            |
| 23          | 4            |
| 24          | 7            |
| 25          | 8            |

### Expected Output:

| bus_id | passengers_cnt |
|--------|----------------|
| 1      | 2              |
| 2      | 2              |
| 3      | 2              |

### Problem Constraints:
- Each bus has a unique **bus_id**.
- No two buses arrive at the same time.
- All bus capacities are positive integers.
- Each passenger has a unique **passenger_id**.
- The number of passengers and buses is within reasonable constraints for SQL queries.

---

## Solution Approaches

### Approach: Event-Based Processing Using Variables

#### SQL Query:
```sql
SET @passenger_waiting = 0;

WITH Events AS (
    SELECT NULL AS bus_id, arrival_time AS event_time, NULL AS capacity FROM Passengers
    UNION ALL
    SELECT * FROM Buses),

RunningTotals AS (
    SELECT *, 
           ROW_NUMBER() OVER (ORDER BY event_time) AS rn, 
           CASE
               WHEN bus_id IS NULL AND @passenger_waiting >= 0 THEN @passenger_waiting := @passenger_waiting + 1
               WHEN bus_id IS NOT NULL THEN @passenger_waiting := @passenger_waiting - capacity
               WHEN bus_id IS NULL AND @passenger_waiting < 0 THEN @passenger_waiting := 1 
           END AS cumulative_passengers_waiting
    FROM Events)

SELECT bus_id, 
       CASE 
           WHEN cumulative_passengers_waiting > 0 THEN capacity 
           ELSE capacity + cumulative_passengers_waiting 
       END AS passengers_cnt
FROM RunningTotals WHERE bus_id IS NOT NULL ORDER BY bus_id;
```

#### Explanation:
- Since SQL does not support loops, we simulate the boarding process using a running variable **@passenger_waiting**.
- We **merge both buses and passengers into a single timeline** ordered by event time.
- Passengers **accumulate** as they arrive at the station.
- When a bus arrives:
  - It boards waiting passengers **up to its capacity**.
  - The remaining waiting passengers continue to wait for the next bus.
- We track passengers using a cumulative counter that adjusts as buses take on passengers.

#### Step-by-Step Breakdown:
1. **Combine Events**: Use `UNION ALL` to merge the bus and passenger events.
2. **Order Events**: Sort the timeline using `ORDER BY event_time`.
3. **Use a Running Variable**:
   - Increase **@passenger_waiting** when a passenger arrives.
   - Decrease **@passenger_waiting** when a bus arrives, accounting for its capacity.
   - Reset **@passenger_waiting** if it goes negative (indicating empty seats available).
4. **Compute Boarded Passengers**:
   - If **remaining passengers waiting is positive**, the bus is full.
   - If **remaining passengers waiting is negative**, compute the boarded passengers using available seats.

---

## Performance Analysis

### Event-Based Processing Using Variables:
- **Time Complexity**: `O(N log N)` due to sorting events.
- **Space Complexity**: `O(1)` since we use a running variable instead of extra tables or joins.

---

## Conclusion

- **Strengths**:
  - Efficient and avoids complex joins.
  - Uses a running variable to track passengers dynamically.
  - Works well for large datasets since sorting is `O(N log N)`.
- **Weaknesses**:
  - Requires variable management, which might not be supported in all SQL versions.

### Final Conclusion:
- This approach is optimal for solving the problem using SQL as it efficiently processes passengers and buses in a single pass over sorted events.
