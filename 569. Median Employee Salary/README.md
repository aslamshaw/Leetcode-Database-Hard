# LeetCode Problem 569: Median Employee Salary

## Problem Description

The Employee table holds all employees. The table has three columns: Employee Id, Company Name, and Salary.

Your task is to find the median salary for each company. Return the median salary for each company, rounded to the nearest integer.

### Tables:

#### Employee Table:

| Id   | Company    | Salary |
|------|------------|--------|
| 1    | A          | 2341   |
| 2    | A          | 341    |
| 3    | A          | 15     |
| 4    | A          | 15314  |
| 5    | A          | 451    |
| 6    | A          | 513    |
| 7    | B          | 15     |
| 8    | B          | 13     |
| 9    | B          | 1154   |
| 10   | B          | 1345   |
| 11   | B          | 1221   |
| 12   | B          | 234    |
| 13   | C          | 2345   |
| 14   | C          | 2645   |
| 15   | C          | 2645   |
| 16   | C          | 2652   |
| 17   | C          | 65     |

### Expected Output:

| Id   | Company    | Salary |
|------|------------|--------|
| 5    | A          | 451    |
| 6    | A          | 513    |
| 12   | B          | 234    |
| 9    | B          | 1154   |
| 14   | C          | 2645   |

### Problem Constraints:
- The Employee table has at most 1000 rows.
- The salary is an integer, and each employee has a unique ID.

---

## Solution Approach

### Approach 1: Using Row Number and Count

To find the median salary for each company, the idea is to first rank the salaries within each company, then calculate the count of employees per company, and finally filter the results to find the middle salary.

#### SQL Query:
```sql
WITH RankAndCount AS (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY company ORDER BY salary ASC) AS rk,
           COUNT(id) OVER (PARTITION BY company) AS n
    FROM Employee
)
SELECT id, company, salary
FROM RankAndCount
WHERE rk >= n / 2 AND rk <= n / 2 + 1;
```

#### Explanation:
- **Step 1**: Use the `ROW_NUMBER()` function to assign a rank to each salary within each company in ascending order.
- **Step 2**: Use the `COUNT()` function to calculate the total number of employees per company.
- **Step 3**: Filter out the rows where the rank is between `n / 2` and `n / 2 + 1`. This will give us the median salary for each company.

Since SQL performs integer division, this approach works for both odd and even counts of employees. 

The SQL query works as follows:
1. It assigns a row number (`rk`) to each salary in ascending order for each company.
2. It calculates the total number of employees (`n`) in each company.
3. It filters out the rows where the rank is between `n / 2` and `n / 2 + 1` to get the median salaries for each company.

---

## Performance Analysis

### Method 1 (Row Number and Count):

- **Time complexity**: O(n log n) due to the sorting operation inside the `ROW_NUMBER()` function.
- **Space complexity**: O(n), where n is the number of employees, since we are storing the row numbers and counts.

---

## Conclusion

- **Method 1**: This approach using `ROW_NUMBER()` and `COUNT()` is flexible and works well for both even and odd numbers of employees. It provides precise control over how the median is calculated.
  
### Final Conclusion:
- Method 1 is the preferred solution for this problem as it offers a robust and flexible approach for calculating the median salary for each company, handling both even and odd numbers of employees effectively.
