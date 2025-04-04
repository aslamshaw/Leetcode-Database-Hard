# LeetCode Problem 3482: Analyze Organization Hierarchy

## Problem Description

Given a dataset of employees and their reporting relationships, determine each employee’s hierarchical level, the size of the team they manage (if any), and the total salary budget they are responsible for. An employee may have direct and indirect reports, and those all contribute to the team size and budget.

The top-level manager (i.e., the CEO) has no manager and is considered level 1. Every subsequent level increases the hierarchy depth by 1. An employee’s team includes all subordinates reporting to them directly or indirectly. The budget is the sum of salaries for the employee and their entire team.

The goal is to compute, for every employee:
- Their level in the organization
- The number of team members they manage
- The total salary budget (including their own salary and all of their team’s salaries)

The output must be sorted by:
1. Level (ascending)
2. Budget (descending)
3. Employee name (alphabetically ascending)

### Tables:

#### Employees Table:

| employee_id | employee_name | manager_id | salary | department     |
|-------------|----------------|------------|--------|----------------|
| 101         | Emma           | null       | 13000  | Executive      |
| 102         | Liam           | 101        | 11000  | Finance        |
| 103         | Olivia         | 101        | 11000  | Operations     |
| 104         | Noah           | 102        | 8000   | Finance        |
| 105         | Ava            | 102        | 8200   | Finance        |
| 106         | Mason          | 103        | 9500   | Operations     |
| 107         | Sophia         | 103        | 9100   | Operations     |
| 108         | Lucas          | 104        | 6200   | Finance        |
| 109         | Isabella       | 106        | 7200   | Operations     |
| 110         | Mia            | 106        | 7200   | Operations     |

### Expected Output:

| employee_id | employee_name | level | team_size | budget |
|-------------|----------------|-------|-----------|--------|
| 101         | Emma           | 1     | 9         | 92500  |
| 103         | Olivia         | 2     | 4         | 44800  |
| 102         | Liam           | 2     | 3         | 33400  |
| 106         | Mason          | 3     | 2         | 23900  |
| 104         | Noah           | 3     | 1         | 14200  |
| 107         | Sophia         | 3     | 0         | 9100   |
| 105         | Ava            | 3     | 0         | 8200   |
| 109         | Isabella       | 4     | 0         | 7200   |
| 110         | Mia            | 4     | 0         | 7200   |
| 108         | Lucas          | 4     | 0         | 6200   |

### Problem Constraints:
- `employee_id` is unique for each employee.
- Each employee may have zero or one manager.
- Each manager can have multiple direct or indirect reports.
- Salaries are positive integers.
- The hierarchy is a tree with a single root (the CEO).

---

## Solution Approaches

### Approach 1: Recursive CTE for Hierarchy and Aggregates

This approach uses two recursive common table expressions (CTEs) to calculate both the level of each employee in the hierarchy and their managerial span in terms of team size and salary budget.

#### SQL Query:
```sql
WITH RECURSIVE EmployeeHierarchy AS (
SELECT employee_id, employee_name, 1 AS level FROM Employees WHERE manager_id IS NULL
UNION ALL
SELECT e.employee_id, e.employee_name, level+1
FROM EmployeeHierarchy eh JOIN Employees e ON e.manager_id = eh.employee_id),

Subordinates AS (
SELECT employee_id AS subordinate_id, manager_id AS team_lead, salary AS subordinate_salary 
FROM Employees WHERE manager_id IS NOT NULL
UNION ALL
SELECT e.employee_id, team_lead, salary 
FROM Subordinates s JOIN Employees e ON e.manager_id = s.subordinate_id),

Teams AS (
SELECT team_lead AS employee_id, COUNT(*)-1 AS team_size, SUM(subordinate_salary) AS budget
FROM (SELECT * FROM Subordinates 
      UNION ALL 
      SELECT employee_id, employee_id, salary FROM Employees) s
GROUP BY team_lead)

SELECT * FROM EmployeeHierarchy JOIN Teams USING(employee_id) ORDER BY level, budget DESC, employee_name;
```

#### Explanation:
- **Hierarchy Calculation (Level):**
  - We start by identifying the CEO, who has a `NULL` as `manager_id`, and assign them level 1.
  - Recursively, each employee reporting to another inherits their manager’s level + 1.
  - This builds a full hierarchy with depth levels.

- **Team Aggregation (Team Size & Budget):**
  - The second recursive CTE builds a relationship between each manager (referred to as `team_lead`) and their direct or indirect subordinates.
  - In the base case, we pull all employees with non-null managers and treat the manager as the initial `team_lead`.
  - In the recursive step, we continue passing down the original `team_lead` as we discover deeper subordinates.
  - This captures the full team under each manager, including indirect relationships.
  
- **Final Calculations:**
  - To determine the `team_size`, we count all subordinates associated with each manager and subtract one to exclude the manager themself.
  - For `budget`, we sum all salaries related to the team (including the manager's own salary).
  - Finally, we join the hierarchy and team summaries and order the results by level, then descending budget, then ascending name.

---

## Performance Analysis

### Method 1 (Recursive CTE for Hierarchy and Aggregates):

- **Time complexity**: O(N²) in the worst-case scenario (e.g., linear reporting structure), due to the recursive joins over the employees table.
- **Space complexity**: O(N), where N is the number of employees stored in intermediate CTEs and results.

---

## Conclusion

- **Method 1**: 
  - **Strengths**: Cleanly separates concerns of hierarchy traversal and aggregation. It scales reasonably well for moderately sized datasets and is easy to interpret using recursive logic.
  - **Weaknesses**: Recursive CTEs can be inefficient on large hierarchies due to repeated joins and may lead to performance issues on very deep or large trees.

### Final Conclusion:
- The recursive CTE approach is effective and intuitive for problems involving organizational trees. While performance can degrade with large input sizes, this method remains one of the most structured and maintainable solutions for computing both hierarchical depth and managerial metrics in SQL.
