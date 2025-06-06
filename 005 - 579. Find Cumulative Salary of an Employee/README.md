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

### Approach 1: 



#### SQL Query:
```sql
WITH Months AS (
    SELECT 1 AS month UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION
    SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION
    SELECT 9 UNION SELECT 10 UNION SELECT 11 UNION SELECT 12),
IdMonths AS (SELECT id, month FROM Months CROSS JOIN (SELECT DISTINCT id FROM Employee) s),
FilteredEmployees AS (
    SELECT * FROM (SELECT *, ROW_NUMBER() OVER (PARTITION BY id ORDER BY month DESC) AS rk FROM Employee) s
    WHERE rk != 1),
EmployeeLogs AS (
    SELECT id, month, IFNULL(salary, 0) AS salary FROM IdMonths LEFT JOIN FilteredEmployees USING(id, month)),
CumSal AS (
    SELECT id, month,
           SUM(salary) OVER (PARTITION BY id ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS salary
    FROM EmployeeLogs)
SELECT id, month, c.salary FROM CumSal c JOIN FilteredEmployees USING(id, month) ORDER BY id, month desc
```

#### Explanation:
- 
- 
- 

### Approach 2: Using `DENSE_RANK()` to Exclude the Latest Month and `ORDER BY Month DESC` for Windowed Cumulative Sum

In this approach, we use the `DENSE_RANK()` function to assign a ranking to the months in descending order. We exclude the most recent month by selecting only those rows with a rank greater than 1. Then, we use a windowed sum to calculate the cumulative salary for the remaining months.

#### SQL Query:
```sql
SELECT Id, Month, 
SUM(Salary) OVER (PARTITION BY Id ORDER BY month RANGE 2 PRECEDING) AS Salary
FROM (SELECT *, DENSE_RANK() OVER (PARTITION BY Id ORDER BY Month DESC) AS rk FROM Employee) s WHERE rk != 1;
```

#### Explanation:
- First, we use `DENSE_RANK()` to assign a rank to each row, ordered by `Month DESC`, for each employee.
- We exclude the most recent month by filtering out the rows where the rank equals 1.
- Then, we use a window function to calculate the cumulative salary over the previous 3 months using RANGE 2 PRECEDING instead of ROWS BETEEN ... AND ..., thus adjusting the window to only include the relevant months.

---

## Performance Analysis

### Method 1 (Using `UNION`):

- **Time complexity**: 
- **Space complexity**: 

### Method 2 (Using `DENSE_RANK()`):

- **Time complexity**: O(n log n) where `n` is the number of rows in the `Employee` table, because of the sorting step when applying the `DENSE_RANK()` function.
- **Space complexity**: O(n), as we store the ranking result and the windowed sum in memory.

---

## Conclusion

- **Method 1**: 
- **Method 2**: While this approach uses `DENSE_RANK()` to exclude the latest month, it has a slightly higher time complexity due to the sorting step. However, it still works efficiently in practice for smaller datasets.

### Final Conclusion:
- **Method 1** is generally the preferable approach due to its simpler logic and better time complexity. However, **Method 2** can also be useful if you are already familiar with ranking functions and want to explore an alternative solution.
