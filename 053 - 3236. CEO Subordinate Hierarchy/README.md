# LeetCode Problem 3236: CEO Subordinate Hierarchy

## Problem Description

You are given a table `Employees` that stores organizational data for a company. Each record represents an employee, including their unique ID, name, salary, and their manager's ID. The CEO is the only person in the table whose `manager_id` is `NULL`. Your task is to identify **all subordinates of the CEO**, whether they report directly or indirectly, and for each subordinate, return the following:

- Their employee ID and name
- Their **hierarchy level** (1 if they report directly to the CEO, 2 if they report to someone who reports to the CEO, etc.)
- The **difference in salary** between them and the CEO

The output should be sorted in ascending order first by hierarchy level and then by employee ID.

### Tables:

#### Employees Table:

| employee_id | employee_name | manager_id | salary  |
|-------------|----------------|------------|---------|
| 101         | Ava            | NULL       | 200000  |
| 102         | Ben            | 101        | 170000  |
| 103         | Clara          | 101        | 160000  |
| 104         | Dan            | 102        | 150000  |
| 105         | Elsa           | 102        | 140000  |
| 106         | Finn           | 103        | 135000  |
| 107         | Gina           | 103        | 138000  |
| 108         | Hank           | 105        | 130000  |

### Expected Output:

| subordinate_id | subordinate_name | hierarchy_level | salary_difference |
|----------------|------------------|------------------|-------------------|
| 102            | Ben              | 1                | -30000            |
| 103            | Clara            | 1                | -40000            |
| 104            | Dan              | 2                | -50000            |
| 105            | Elsa             | 2                | -60000            |
| 106            | Finn             | 2                | -65000            |
| 107            | Gina             | 2                | -62000            |
| 108            | Hank             | 3                | -70000            |

### Problem Constraints:
- There is exactly one CEO in the organization (only one row with `manager_id IS NULL`).
- All employees belong to a single connected hierarchy rooted at the CEO.
- Output must be sorted by `hierarchy_level ASC`, then `subordinate_id ASC`.

---

## Solution Approaches

### Approach 1: Recursive Hierarchical Traversal using CTE

This method uses a **recursive common table expression (CTE)** to explore the employee hierarchy starting from the CEO. It traverses all levels of subordinates while keeping track of how deep in the hierarchy each employee is.

#### SQL Query:
```sql
SET @ceo_sal = (SELECT salary FROM Employees WHERE manager_id IS NULL);

WITH RECURSIVE EmployeeHierarchy AS (
    SELECT employee_id, employee_name, 0 AS hierarchy_level, salary 
    FROM Employees 
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.employee_name, eh.hierarchy_level + 1, e.salary
    FROM EmployeeHierarchy eh 
    JOIN Employees e ON eh.employee_id = e.manager_id)

SELECT employee_id AS subordinate_id, 
       employee_name AS subordinate_name, 
       hierarchy_level, 
       (salary - @ceo_sal) AS salary_difference
FROM EmployeeHierarchy 
WHERE hierarchy_level > 0 ORDER BY hierarchy_level, subordinate_id;
```

#### Explanation:
- We begin by identifying the CEO (the employee with a `NULL` manager_id).
- Using a recursive CTE named `EmployeeHierarchy`, we build the structure layer by layer:
  - Start from the CEO (level 0),
  - Recursively join the CTE with the `Employees` table to find all direct reports of the previous level.
- During recursion, the `hierarchy_level` is incremented.
- Once the full structure is built, we filter out the CEO and calculate the salary difference of each subordinate relative to the CEO using a variable holding the CEO's salary.
- The result is ordered by `hierarchy_level` and `subordinate_id`.

---

## Performance Analysis

### Method 1 (Recursive Hierarchical Traversal using CTE):

- **Time complexity**: O(N), where N is the number of employees. Each employee is visited once in the recursive process.
- **Space complexity**: O(N), due to the recursive CTE and the result set held in memory.

---

## Conclusion

- **Method 1**: This method efficiently models the organizational tree and is both readable and scalable for large datasets. It makes good use of SQL's recursive capabilities, though performance may vary slightly depending on the SQL engine's optimization of recursive queries.

### Final Conclusion:
- The recursive CTE approach is the optimal and natural choice for this problem as it mirrors the hierarchical nature of the employee data structure, while maintaining clear logic and acceptable performance.
