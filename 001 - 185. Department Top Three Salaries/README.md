# LeetCode Problem 185: Department Top Three Salaries

## Problem Description

Given two tables `Employee` and `Department`, we need to find the top three highest salaries in each department. We must return the department name, employee name, and salary for the employees who are among the top three highest earners in their respective departments.

### Tables:

#### Employee Table:

| id  | name   | salary | deptId |
|-----|--------|--------|--------|
| 1   | Joe    | 85000  | 1      |
| 2   | Henry  | 80000  | 2      |
| 3   | Sam    | 60000  | 2      |
| 4   | Max    | 90000  | 1      |
| 5   | Janet  | 69000  | 1      |
| 6   | Randy  | 85000  | 1      |
| 7   | Will   | 70000  | 1      |

#### Department Table:

| id  | name   |
|-----|--------|
| 1   | IT     |
| 2   | Sales  |

### Expected Output:

| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

### Problem Constraints:
- The number of employees and departments is not specified, so solutions should be efficient for large datasets.

---

## Solution Approaches

### Approach 1: Using `DENSE_RANK()` Window Function

We can use the `DENSE_RANK()` window function to assign a rank to each employee’s salary within their department, ordered by salary in descending order. Then, we select all employees who have a rank less than 4 (i.e., the top 3 highest salaries).

#### SQL Query:
```sql
WITH CTE AS (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY deptId ORDER BY salary DESC) AS ranking
    FROM employee
)
SELECT d.name AS Department, cte.name AS Employee, cte.salary
FROM CTE
JOIN department AS d ON cte.deptId = d.id
WHERE ranking <= 3;
```

#### Explanation:
- **Common Table Expression (CTE)**: We use a CTE to calculate the dense rank of employees within each department based on their salary in descending order.
- **`DENSE_RANK()`**: This window function assigns ranks to the employees in each department. It ensures that if two employees have the same salary, they receive the same rank.
- **Filter for top 3 ranks**: We filter the results to only return employees who have a rank of 3 or less, meaning they are among the top three earners in their department.

### Approach 2: Using a Correlated Subquery

A correlated subquery is a subquery that references columns from the outer query. In this case, the outer query selects the department and employee, while the subquery counts how many employees in the same department have a higher salary than the current employee. If fewer than three employees have a higher salary, the current employee qualifies as a top earner.

#### SQL Query:
```sql
SELECT 
    d.name AS 'Department', 
    e1.name AS 'Employee', 
    e1.salary AS 'Salary'
FROM Employee e1
JOIN Department d ON e1.deptId = d.id
WHERE (
    SELECT COUNT(DISTINCT e2.salary) 
    FROM Employee e2 
    WHERE e2.salary > e1.salary AND e1.deptId = e2.deptId
) < 3;
```

#### Explanation:
- **Join between Employee and Department tables**: We first join the `Employee` and `Department` tables based on the `deptId` field.
- **Correlated Subquery**: The subquery counts how many employees in the same department have a salary greater than the current employee's salary. This count is restricted to those with higher salaries within the same department.
- **Top 3 condition**: The outer query filters for employees where the subquery count is less than 3, meaning the employee has a salary in the top three salaries for their department.

---

## Performance Analysis

### Method 1 (Using `DENSE_RANK()`):

- **Time complexity**: O(n log n), where `n` is the number of employees. The `DENSE_RANK()` function sorts employees by salary, which takes O(n log n) time.
- **Space complexity**: O(n), as we store the ranks and salaries in the CTE.

### Method 2 (Using Correlated Subquery):

- **Time complexity**: O(n^2), because for each employee, the subquery iterates through all employees to count those with a higher salary. This makes it less efficient than Method 1.
- **Space complexity**: O(1), as we do not store intermediate results except for the query execution.

---

## Conclusion

- **Method 1** is more efficient, as it leverages the window function `DENSE_RANK()` to calculate ranks in a single pass over the data.
- **Method 2** can work well for small datasets but may suffer from performance issues on larger datasets due to the correlated subquery.

### Final Conclusion:
Both methods offer valid solutions to the problem, but using `DENSE_RANK()` is generally preferred for larger datasets due to its better performance. This problem highlights the importance of efficient SQL queries when working with large datasets in relational databases.
