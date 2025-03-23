# LeetCode Problem 262: Trips and Users

## Problem Description

Given two tables `Trips` and `Users`, we need to find the cancellation rate of requests made by unbanned users (both client and driver must not be banned) between Oct 1, 2013, and Oct 3, 2013. The cancellation rate is computed by dividing the number of canceled (by client or driver) requests with unbanned users by the total number of requests with unbanned users on that day.

### Tables:

#### Trips Table:

| id  | client_id | driver_id | city_id | status              | request_at |
|-----|-----------|-----------|---------|---------------------|------------|
| 1   | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2   | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3   | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4   | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5   | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6   | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7   | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8   | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9   | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10  | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |

#### Users Table:

| users_id | banned | role   |
|----------|--------|--------|
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |

### Expected Output:

| Day        | Cancellation Rate |
|------------|-------------------|
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |

### Problem Constraints:
- The cancellation rate is calculated by dividing the number of canceled requests (by client or driver) with unbanned users by the total number of requests with unbanned users on that day.
- Unbanned users are those who have "No" in the `banned` column in the `Users` table.
- The request date should be between "2013-10-01" and "2013-10-03".
  
---

## Solution Approaches

### Approach 1: Using `WHERE` Clause to Filter Data

In this approach, we filter out the trips made by banned clients and drivers by checking the `Users` table. After filtering, we calculate the cancellation rate by dividing the number of canceled trips by the total number of trips for each day.

#### SQL Query:
```sql
SELECT Request_at AS Day, 
       ROUND(SUM(Status != 'completed') / COUNT(Status), 2) AS 'Cancellation Rate' 
FROM Trips 
WHERE Client_Id IN (SELECT Users_Id FROM Users WHERE Banned = 'No' AND Role = 'client') 
  AND Driver_Id IN (SELECT Users_Id FROM Users WHERE Banned = 'No' AND Role = 'driver') 
  AND '2013-10-01' <= Request_at AND Request_at <= '2013-10-03' 
GROUP BY Request_at;
```
#### Explanation:
- **Filtering out banned users**: The WHERE clause filters the trips by checking that both the client_id and driver_id are not banned in the Users table.
- **Cancellation rate**: The cancellation rate is calculated by dividing the number of canceled trips (Status != 'completed') by the total number of trips (COUNT(Status)).
- **Date range**: We ensure that only trips from 2013-10-01 to 2013-10-03 are considered.

### Approach 2: Using JOIN and AVG to Calculate Cancellation Rate
In this approach, we use JOIN to combine the Trips table with the Users table to filter out banned clients and drivers. We then calculate the cancellation rate using the AVG function.

#### SQL Query:
```sql
SELECT T.Request_at AS Day, 
       ROUND(AVG(T.Status != 'completed'), 2) AS 'Cancellation Rate'
FROM Trips T
JOIN Users U1 ON T.Client_Id = U1.Users_Id AND U1.Banned = 'No' AND U1.Role = 'client'
JOIN Users U2 ON T.Driver_Id = U2.Users_Id AND U2.Banned = 'No' AND U2.Role = 'driver'
WHERE T.Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.Request_at;
```
#### Explanation:
- **Using JOIN**: We join the Trips table with the Users table for both clients and drivers to ensure that both are unbanned.
- **Calculating cancellation rate**: The cancellation rate is calculated using AVG(T.Status != 'completed'), which calculates the proportion of trips that are canceled (not completed).
- **Date range**: The WHERE clause ensures that only trips within the date range of 2013-10-01 to 2013-10-03 are included.
---

## Performance Analysis

### Method 1 (Using `WHERE` Clause):

- **Time complexity**: O(n), where `n` is the number of trips. The filtering and counting operations are linear with respect to the number of trips.
- **Space complexity**: O(n), as the filtered trips are stored temporarily during the execution.

### Method 2 (Using `JOIN`):

- **Time complexity**: O(n), where `n` is the number of trips. The `JOIN` operation and filtering are linear, and the cancellation rate calculation uses `AVG`, which also processes the data linearly.
- **Space complexity**: O(n), as intermediate results are stored for the joins.

---

## Conclusion

- **Method 1** is a straightforward approach using filtering in the `WHERE` clause and is easy to understand.
- **Method 2** leverages `JOIN` operations, which are often more efficient in terms of readability and performance in large datasets, especially when using aggregation functions like `AVG`.

### Final Conclusion:
Both methods are valid solutions to the problem, but using `JOIN` operations may be preferable for larger datasets due to its cleaner structure and potentially better optimization by the SQL engine.
