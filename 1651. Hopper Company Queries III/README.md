# LeetCode Problem 1651: Hopper Company Queries III

## Problem Description

We are given three tables: **Drivers**, **Rides**, and **AcceptedRides**. The goal is to compute the average ride distance and average ride duration of every 3-month window from January 2020 to October 2020. The average is calculated by summing up the total ride distances and durations over each 3-month period and then dividing by 3. Round both the average ride distance and the average ride duration to two decimal places.

### Tables:

#### Drivers Table:

| driver_id | join_date  |
|-----------|------------|
| 10        | 2019-12-10 |
| 8         | 2020-1-13  |
| 5         | 2020-2-16  |
| 7         | 2020-3-8   |
| 4         | 2020-5-17  |
| 1         | 2020-10-24 |
| 6         | 2021-1-5   |

#### Rides Table:

| ride_id | user_id | requested_at |
|---------|---------|--------------|
| 6       | 75      | 2019-12-9    |
| 1       | 54      | 2020-2-9     |
| 10      | 63      | 2020-3-4     |
| 19      | 39      | 2020-4-6     |
| 3       | 41      | 2020-6-3     |
| 13      | 52      | 2020-6-22    |
| 7       | 69      | 2020-7-16    |
| 17      | 70      | 2020-8-25    |
| 20      | 81      | 2020-11-2    |
| 5       | 57      | 2020-11-9    |
| 2       | 42      | 2020-12-9    |
| 11      | 68      | 2021-1-11    |
| 15      | 32      | 2021-1-17    |
| 12      | 11      | 2021-1-19    |
| 14      | 18      | 2021-1-27    |

#### AcceptedRides Table:

| ride_id | driver_id | ride_distance | ride_duration |
|---------|-----------|---------------|---------------|
| 10      | 10        | 63            | 38            |
| 13      | 10        | 73            | 96            |
| 7       | 8         | 100           | 28            |
| 17      | 7         | 119           | 68            |
| 20      | 1         | 121           | 92            |
| 5       | 7         | 42            | 101           |
| 2       | 4         | 6             | 38            |
| 11      | 8         | 37            | 43            |
| 15      | 8         | 108           | 82            |
| 12      | 8         | 38            | 34            |
| 14      | 1         | 90            | 74            |

### Expected Output:

| month | average_ride_distance | average_ride_duration |
|-------|-----------------------|-----------------------|
| 1     | 21.00                 | 12.67                 |
| 2     | 21.00                 | 12.67                 |
| 3     | 21.00                 | 12.67                 |
| 4     | 24.33                 | 32.00                 |
| 5     | 57.67                 | 41.33                 |
| 6     | 97.33                 | 64.00                 |
| 7     | 73.00                 | 32.00                 |
| 8     | 39.67                 | 22.67                 |
| 9     | 54.33                 | 64.33                 |
| 10    | 56.33                 | 77.00                 |

### Problem Constraints:
- The problem involves working with tables that contain ride data for the year 2020, and the goal is to aggregate this data over 3-month windows.

---

## Solution Approaches

### Approach 1: Window Calculation Using Recursive CTE

The solution involves calculating the total ride distance and ride duration for each month from January 2020 to October 2020, and then using a rolling window of 3 months to calculate the average. We join the tables to get the relevant ride data, aggregate it, and use the recursive approach to ensure that we cover every 3-month window.

#### SQL Query:
```sql
WITH RECURSIVE M AS (SELECT 1 AS month UNION ALL SELECT month + 1 FROM M WHERE month < 12),
AcceptedRidesSum AS (SELECT MONTH(requested_at) AS month, 
                     SUM(ride_distance) AS ride_distance, SUM(ride_duration) AS ride_duration
                     FROM Rides JOIN AcceptedRides USING(ride_id) WHERE YEAR(requested_at) = 2020 GROUP BY month),
Filled AS (SELECT month, IFNULL(ride_distance, 0) AS ride_distance, IFNULL(ride_duration, 0) AS ride_duration 
FROM M LEFT JOIN AcceptedRidesSum USING(month))
SELECT month, 
ROUND(AVG(ride_distance) OVER (ORDER BY month ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING), 2) AS average_ride_distance,    
ROUND(AVG(ride_duration) OVER (ORDER BY month ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING), 2) AS average_ride_duration 
FROM Filled LIMIT 10;
```

#### Explanation:
- A recursive CTE is used to create a list of months from 1 to 12.
- A left join is performed between the months and the ride data to ensure we capture all months.
- A window function is used to calculate the rolling average for each 3-month period.
- The result is rounded to two decimal places as required.

---

## Performance Analysis

### Method 1: Window Calculation Using Recursive CTE:

- **Time complexity**: O(n), where n is the number of rows in the Rides and AcceptedRides tables, and the recursive CTE runs for 12 months.
- **Space complexity**: O(n), where n is the space required to store the intermediate results for each month and rolling window calculation.

---

## Conclusion

- **Method 1**: This approach efficiently calculates the required 3-month rolling average using window functions and recursive CTEs. It ensures that we cover all the months in the year 2020 and handle the edge cases correctly. The performance is optimal for this type of query.
