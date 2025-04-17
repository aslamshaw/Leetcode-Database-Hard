# LeetCode Problem 3057: Employees Project Allocation

## Problem Description

You are given two tables: `Project` and `Employees`. The `Project` table contains information about which employees are working on which projects and their corresponding workload. The `Employees` table contains details about each employee, including their `employee_id`, name, and the team they belong to.

You need to identify the employees who are working on projects with a workload that exceeds the average workload of all employees within their respective teams.

Return a result table that includes the employee's `employee_id`, `project_id`, `employee_name`, and `project_workload`. The result should be ordered by `employee_id` and `project_id` in ascending order.

### Tables:

#### Project Table:

| project_id | employee_id | workload |
|------------|-------------|----------|
| 1          | 1           | 45       |
| 1          | 2           | 90       |
| 2          | 3           | 12       |
| 2          | 4           | 68       |

#### Employees Table:

| employee_id | name   | team |
|-------------|--------|------|
| 1           | Khaled | A    |
| 2           | Ali    | B    |
| 3           | John   | B    |
| 4           | Doe    | A    |

### Expected Output:

| employee_id | project_id | employee_name | project_workload |
|-------------|------------|---------------|------------------|
| 2           | 1          | Ali           | 90               |
| 4           | 2          | Doe           | 68               |

### Problem Constraints:
- The `employee_id` is the primary key for both the `Project` and `Employees` tables.
- The `employee_id` in the `Project` table is a foreign key that references the `Employees` table.
- The input tables may contain up to 100,000 rows.

---

## Solution Approaches

### Approach 1: Using Window Functions and CTE

This approach utilizes a common table expression (CTE) with a window function to calculate the average workload for each team. The `AVG(workload) OVER (PARTITION BY team)` window function allows us to calculate the average workload for each team without needing a separate subquery. Once the average team workload is calculated, we filter out employees whose project workloads exceed the team's average.

#### SQL Query:
```sql
WITH AvgTeamWorkload AS (
    SELECT *, 
           AVG(workload) OVER (PARTITION BY team) AS avg_team_workload
    FROM Project
    JOIN Employees USING(employee_id))

SELECT employee_id, 
       project_id, 
       name AS employee_name, 
       workload AS project_workload
FROM AvgTeamWorkload
WHERE workload > avg_team_workload
ORDER BY employee_id, project_id; 
```

#### Explanation:
1. We join the `Project` and `Employees` tables on the `employee_id` field.
2. We use a `WITH` clause to create a CTE (`AvgTeamWorkload`), which calculates the average workload for each team using the window function `AVG(workload) OVER (PARTITION BY team)`.
3. We filter the records in the `AvgTeamWorkload` CTE to include only those rows where the `workload` exceeds the `avg_team_workload` for that employee's team.
4. Finally, we select the required columns (`employee_id`, `project_id`, `name`, and `workload`) and order the result by `employee_id` and `project_id` in ascending order.

---

## Performance Analysis

### Method 1 (Using Window Functions and CTE):

- **Time complexity**: O(n), where n is the number of rows in the `Project` table. The window function operates in linear time since it calculates the average workload for each team in a single pass over the data.
- **Space complexity**: O(n), as the CTE stores intermediate results for each row in the `Project` table.

---

## Conclusion

- **Method 1**: The first approach using window functions is efficient and straightforward. It allows the calculation of the average workload for each team in a single pass over the data, making it a preferred solution for large datasets.

### Final Conclusion:
- **Method 1** (using window functions) is the preferred approach due to its efficiency and simplicity in calculating the required average workload within each team.
