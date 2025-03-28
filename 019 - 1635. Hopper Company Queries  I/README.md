# LeetCode Problem 1635: Hopper Company Queries I

## Problem Description

Given three tables: `Drivers`, `Rides`, and `AcceptedRides`, the task is to report the following statistics for each month of 2020:
1. The number of drivers currently with the Hopper company by the end of the month (`active_drivers`).
2. The number of accepted rides in that month (`accepted_rides`).

The result should be ordered by month in ascending order (January as 1, February as 2, etc.).

### Tables:

#### Drivers Table:

| driver_id | join_date  |
|----------|------------|
| 15       | 2019-11-22  |
| 11       | 2020-1-10   |
| 9        | 2020-2-18   |
| 12       | 2020-3-14   |
| 16       | 2020-4-20   |
| 3        | 2020-10-5   |
| 8        | 2021-1-9    |

#### Rides Table:

| ride_id | user_id | requested_at |
|--------|--------|--------------|
| 1      | 22     | 2019-11-20    |
| 3      | 34     | 2020-2-6      |
| 7      | 19     | 2020-3-12     |
| 9      | 48     | 2020-4-18     |
| 14     | 21     | 2020-6-8      |
| 20     | 32     | 2020-7-11     |
| 25     | 44     | 2020-8-25     |
| 30     | 27     | 2020-9-10     |
| 35     | 36     | 2020-11-7     |
| 40     | 50     | 2020-12-15    |
| 45     | 18     | 2021-1-3      |
| 50     | 60     | 2021-1-20     |

#### AcceptedRides Table:

| ride_id | driver_id | ride_distance | ride_duration |
|--------|----------|---------------|---------------|
| 7      | 15       | 55            | 30            |
| 9      | 11       | 65            | 42            |
| 14     | 11       | 75            | 60            |
| 20     | 12       | 82            | 70            |
| 25     | 16       | 90            | 45            |
| 30     | 3        | 100           | 55            |
| 35     | 12       | 110           | 68            |
| 40     | 15       | 120           | 72            |
| 45     | 8        | 98            | 50            |
| 50     | 8        | 88            | 40            |

### Expected Output:

| month | active_drivers | accepted_rides |
|------|----------------|----------------|
| 1    | 2              | 0              |
| 2    | 3              | 0              |
| 3    | 4              | 1              |
| 4    | 5              | 1              |
| 5    | 5              | 0              |
| 6    | 5              | 1              |
| 7    | 5              | 1              |
| 8    | 5              | 1              |
| 9    | 6              | 1              |
| 10   | 7              | 0              |
| 11   | 7              | 1              |
| 12   | 7              | 1              |

### Problem Constraints:
- The number of drivers is limited to reasonable values for efficient aggregation.
- The number of rides and accepted rides is also limited to manageable sizes.
- Year 2020 is specifically targeted, and months are numbered from 1 to 12.
- We need to account for months with no rides or driver additions.

---

## Solution Approaches

### Approach 1: Cumulative Count with Window Functions

#### SQL Query:
```sql
WITH RECURSIVE Months AS (SELECT 1 AS month UNION SELECT month+1 FROM Months WHERE month < 12),
CountActiveDrivers AS (SELECT MONTH(join_date) AS month, ad 
                       FROM (SELECT *, COUNT(*) OVER (ORDER BY join_date) AS ad FROM Drivers) s 
                       WHERE YEAR(join_date) = 2020),
CountAcceptedRides AS (SELECT MONTH(requested_at) AS month, COUNT(ride_id) AS ar 
                       FROM Rides JOIN AcceptedRides USING(ride_id) WHERE YEAR(requested_at) = 2020 GROUP BY month),
NotFilled AS (SELECT month, ad, IFNULL(ar, 0) AS accepted_rides, 
              SUM(CASE WHEN ad IS NOT NULL THEN 1 ELSE 0 END) OVER (ORDER BY month) AS group_id
              FROM Months LEFT JOIN CountActiveDrivers USING(month) LEFT JOIN CountAcceptedRides USING(month))
SELECT month, MAX(ad) OVER (PARTITION BY group_id) AS active_drivers, accepted_rides FROM NotFilled;
```

#### Explanation:
1. **Counting Active Drivers:**
   - Count how many drivers have joined the company by the end of each month.
   - Use a cumulative count to track active drivers as each month progresses.
   - Consider only the year 2020 to match the requirements.

2. **Counting Accepted Rides:**
   - Join `Rides` and `AcceptedRides` tables to find all accepted rides.
   - Aggregate the count of accepted rides for each month.

3. **Generating Missing Months:**
   - Use a recursive CTE to generate numbers from 1 to 12, representing the months.
   - Left join this list with the aggregated data to ensure all months are present.

4. **Handling Null Values:**
   - Fill missing ride counts with `0`.
   - Use forward filling for the `active_drivers` count to propagate the most recent non-null value.

---

## Performance Analysis

### Method 1 (Cumulative Count with Window Functions):

- **Time complexity**: O(n)  
  - Cumulative counting of drivers.
  - Aggregation of rides.
  - Efficient use of window functions to manage forward filling.

- **Space complexity**: O(n)  
  - Storing intermediate data during the aggregation and forward filling.

---

## Conclusion

- **Method 1**: Efficient and optimal for the given problem size, leveraging SQL window functions and recursive CTEs.
- This approach is preferable as it handles both the aggregation and the forward filling seamlessly, maintaining performance and correctness.

### Final Conclusion:
- The problem is best solved using cumulative counts and window functions due to their efficiency and ability to handle forward filling for months with no new drivers or accepted rides.
