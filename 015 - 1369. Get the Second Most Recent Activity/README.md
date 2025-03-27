# LeetCode Problem 1369: Get the Second Most Recent Activity

## Problem Description

Given a table `UserActivity`, which stores the activities performed by users along with their start and end dates, retrieve the second most recent activity for each user. If a user has only one activity, return that activity instead.

A user cannot perform more than one activity at the same time.

### Tables:

#### UserActivity Table:

| username  | activity     | startDate  | endDate    |
|-----------|-------------|------------|------------|
| John      | Running     | 2023-05-10 | 2023-05-12 |
| John      | Swimming    | 2023-05-15 | 2023-05-18 |
| John      | Cycling     | 2023-05-20 | 2023-05-22 |
| Sarah     | Yoga        | 2023-04-05 | 2023-04-07 |

### Expected Output:

| username  | activity  | startDate  | endDate    |
|-----------|----------|------------|------------|
| John      | Swimming | 2023-05-15 | 2023-05-18 |
| Sarah     | Yoga     | 2023-04-05 | 2023-04-07 |

### Problem Constraints:
- Each user can have multiple activities.
- Activities are uniquely identified by their `startDate`.
- If a user has only one activity, return that one.
- The output can be returned in any order.

---

## Solution Approaches

### Approach 1: Ranking with `DENSE_RANK()`

This approach uses the `DENSE_RANK()` function to assign ranks to activities for each user based on `startDate` in descending order. Additionally, the `COUNT()` function is used to determine the total number of activities for each user.

#### SQL Query:
```sql
SELECT username, activity, startDate,endDate
FROM (SELECT *, DENSE_RANK() OVER (PARTITION BY username ORDER BY startDate DESC) AS rk,
      COUNT(username) OVER (PARTITION BY username) AS cnt FROM UserActivity) AS s
WHERE rk = 2 OR cnt = 1;
```

#### Explanation:
- Assign a rank (`rk`) to each activity of a user, ordering by `startDate` in descending order.
- Count the total number of activities (`cnt`) for each user.
- Select activities where `rk = 2` (i.e., second most recent activity) or `cnt = 1` (user has only one activity).

---

## Performance Analysis

### Method 1 (Ranking with `DENSE_RANK()`):

- **Time complexity**: `O(N log N)`, where `N` is the number of records, due to the window functions.
- **Space complexity**: `O(N)`, as we store ranking information in a subquery.

---

## Conclusion

- **Method 1**: Efficient and simple to implement using SQL window functions. It ensures that the second most recent activity is selected while handling cases where only one activity exists.

### Final Conclusion:
- This method effectively retrieves the desired results with minimal complexity and is optimal for solving this problem.
