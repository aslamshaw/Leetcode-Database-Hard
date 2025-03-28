# LeetCode Problem 1645: Hopper Company Queries II

## Problem Description

Given three tables: **Drivers**, **Rides**, and **AcceptedRides**, the task is to calculate the percentage of working drivers (`working_percentage`) for each month of the year 2020.

### Tables:

#### Drivers Table:

| driver_id | join_date  |
|----------|------------|
| 1        | 2020-01-05 |
| 2        | 2020-02-15 |
| 3        | 2020-03-20 |
| 4        | 2020-04-25 |
| 5        | 2020-05-30 |
| 6        | 2020-06-10 |
| 7        | 2020-07-22 |

#### Rides Table:

| ride_id | user_id | requested_at |
|--------|--------|--------------|
| 101    | 15     | 2020-01-20    |
| 102    | 18     | 2020-02-17    |
| 103    | 21     | 2020-03-05    |
| 104    | 25     | 2020-04-15    |
| 105    | 30     | 2020-05-20    |
| 106    | 35     | 2020-06-25    |
| 107    | 40     | 2020-07-29    |
| 108    | 45     | 2020-08-15    |
| 109    | 50     | 2020-09-12    |
| 110    | 55     | 2020-10-28    |
| 111    | 60     | 2020-11-06    |
| 112    | 65     | 2020-12-18    |

#### AcceptedRides Table:

| ride_id | driver_id | ride_distance | ride_duration |
|--------|----------|---------------|---------------|
| 101    | 1        | 15            | 30            |
| 103    | 2        | 20            | 40            |
| 105    | 3        | 25            | 50            |
| 106    | 4        | 30            | 60            |
| 107    | 5        | 35            | 70            |
| 108    | 6        | 40            | 80            |
| 111    | 7        | 45            | 90            |

### Expected Output:

| month | working_percentage |
|------|--------------------|
| 1    | 100.00              |
| 2    | 50.00               |
| 3    | 33.33               |
| 4    | 0.00                |
| 5    | 33.33               |
| 6    | 50.00               |
| 7    | 33.33               |
| 8    | 50.00               |
| 9    | 0.00                |
| 10   | 0.00                |
| 11   | 50.00               |
| 12   | 0.00                |

### Problem Constraints:
- If the number of available drivers during a month is zero, the `working_percentage` is considered 0.
- The result table should be ordered by month in ascending order.
- Round `working_percentage` to the nearest two decimal places.

---

## Solution Approaches

### Approach 1: Common Table Expressions (CTE)

#### SQL Query:
```sql
WITH RECURSIVE Months AS (SELECT 1 AS month UNION SELECT month+1 FROM Months WHERE month < 12),
CountActiveDrivers AS (SELECT MONTH(join_date) AS month, ad 
                       FROM (SELECT *, COUNT(*) OVER (ORDER BY join_date) AS ad FROM Drivers) s 
                       WHERE YEAR(join_date) = 2020),
CountAcceptedRides AS (SELECT MONTH(requested_at) AS month, COUNT(DISTINCT driver_id) AS ar 
                       FROM Rides JOIN AcceptedRides USING(ride_id) WHERE YEAR(requested_at) = 2020 GROUP BY month),
NotFilled AS (SELECT month, ad, IFNULL(ar, 0) AS accepted_rides, 
              SUM(CASE WHEN ad IS NOT NULL THEN 1 ELSE 0 END) OVER (ORDER BY month) AS group_id
              FROM Months LEFT JOIN CountActiveDrivers USING(month) LEFT JOIN CountAcceptedRides USING(month))
SELECT month, ROUND(accepted_rides/MAX(ad) OVER (PARTITION BY group_id ORDER BY month)*100, 2) 
AS working_percentage FROM NotFilled;
```

**Explanation:**
1. Use CTEs to calculate the number of active drivers and accepted rides for each month.
2. Calculate the percentage of working drivers by dividing the number of drivers who accepted rides by the number of available drivers.
3. Round the percentage to two decimal places.
4. Return the result sorted by month in ascending order.

---

## Performance Analysis

### Method 1 (CTE):

- **Time complexity**: O(n * m), where n is the number of rides and m is the number of drivers.
- **Space complexity**: O(n + m) for storing intermediate results.

---

## Conclusion

- **Method 1**: Uses CTEs to calculate working percentages efficiently. Suitable for large datasets due to aggregation and filtering.
- **Final Conclusion**: CTE-based approach is preferable for calculating monthly working percentages as it efficiently handles complex queries.
