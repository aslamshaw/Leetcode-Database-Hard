# LeetCode Problem 579: Calculate Cumulative Salary of Employees

## Problem Description

You are given a table `Employee` that contains the salary data of employees for various months. Your task is to write a SQL query that calculates the cumulative salary for each employee over a period of three months, excluding the most recent month for each employee.

The results should be displayed in ascending order of `Id` and, within the same `Id`, in descending order of the `Month`.

### Tables:

#### Employee Table:

| Id | Month | Salary |
|----|-------|--------|
| 1  | 1     | 20     |
| 2  | 1     | 25     |
| 1  | 2     | 30     |
| 2  | 2     | 35     |
| 3  | 2     | 40     |
| 1  | 3     | 40     |
| 3  | 3     | 60     |
| 1  | 4     | 50     |
| 3  | 4     | 70     |

### Expected Output:

| Id | Month | Salary |
|----|-------|--------|
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 25     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |

### Problem Constraints:
- The query should exclude the most recent month’s salary for each employee.
- The `Salary` should be the cumulative sum over the previous 3 months, excluding the most recent month for each employee.
- You must order the results by `Id` ascending and `Month` descending.

---

## Solution Approaches

### Approach 1: Using `MAX()` to Identify the Latest Month and `ORDER BY Month ASC` for Calculating Windowed Cumulative Sum

In this approach, we use the `MAX()` function to identify the most recent month for each employee. Then, we calculate the cumulative sum of the salary for the previous 3 months (excluding the most recent one) using a windowed sum.

#### SQL Query:
```sql
WITH EmployeeMaxMonth AS (SELECT *, MAX(Month) OVER (PARTITION BY Id) AS max_month FROM Employee)  
SELECT Id, Month, SUM(Salary) OVER (PARTITION BY Id ORDER BY Month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS Salary  
FROM EmployeeMaxMonth  WHERE Month != max_month  ORDER BY Id, Month DESC;
```

#### Explanation:
- First, we identify the most recent month for each employee using the `MAX()` function and partition by `Id`.
- Next, we calculate the cumulative sum of salaries using a window function. We use the `SUM()` function with a window of `2 PRECEDING AND CURRENT ROW` to get the sum over the last 3 months.
- Finally, we exclude the most recent month by filtering out rows where the `Month` equals the most recent month.

### Approach 2: Using `DENSE_RANK()` to Exclude the Latest Month and `ORDER BY Month DESC` for Windowed Cumulative Sum

In this approach, we use the `DENSE_RANK()` function to assign a ranking to the months in descending order. We exclude the most recent month by selecting only those rows with a rank greater than 1. Then, we use a windowed sum to calculate the cumulative salary for the remaining months.

#### SQL Query:
```sql
SELECT Id, Month, 
SUM(Salary) OVER (PARTITION BY Id ORDER BY Month DESC ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING) AS Salary
FROM (SELECT *, DENSE_RANK() OVER (PARTITION BY Id ORDER BY Month DESC) AS rk FROM Employee) s WHERE rk != 1;
```

#### Explanation:
- First, we use `DENSE_RANK()` to assign a rank to each row, ordered by `Month DESC`, for each employee.
- We exclude the most recent month by filtering out the rows where the rank equals 1.
- Then, we use a window function to calculate the cumulative salary over the previous 3 months, but we adjust the window to only include the relevant months.

---

## Performance Analysis

### Method 1 (Using `MAX()`):

- **Time complexity**: O(n) where `n` is the number of rows in the `Employee` table, since we compute the maximum month for each employee and the windowed cumulative sum in one pass.
- **Space complexity**: O(n), as we store the result of the `MAX()` function and the windowed sum in memory.

### Method 2 (Using `DENSE_RANK()`):

- **Time complexity**: O(n log n) where `n` is the number of rows in the `Employee` table, because of the sorting step when applying the `DENSE_RANK()` function.
- **Space complexity**: O(n), as we store the ranking result and the windowed sum in memory.

---

## Conclusion

- **Method 1**: This approach is more straightforward and uses `MAX()` to find the latest month and computes the cumulative sum using window functions. It is efficient in terms of time complexity and well-suited for this type of problem.
- **Method 2**: While this approach uses `DENSE_RANK()` to exclude the latest month, it has a slightly higher time complexity due to the sorting step. However, it still works efficiently in practice for smaller datasets.

### Final Conclusion:
- **Method 1** is generally the preferable approach due to its simpler logic and better time complexity. However, **Method 2** can also be useful if you are already familiar with ranking functions and want to explore an alternative solution.
