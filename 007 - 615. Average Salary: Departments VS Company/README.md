# LeetCode Problem 615: Average Salary: Departments VS Company

## Problem Description

Given two tables, write a query to display the comparison result (higher/lower/same) of the average salary of employees in a department to the company's average salary.

### Tables:

#### salary Table:

| id | employee_id | amount | pay_date   |
|----|------------|--------|------------|
| 1  | 4          | 8500   | 2022-05-31 |
| 2  | 5          | 7500   | 2022-05-31 |
| 3  | 6          | 9200   | 2022-05-31 |
| 4  | 4          | 7200   | 2022-04-30 |
| 5  | 5          | 6900   | 2022-04-30 |
| 6  | 6          | 8700   | 2022-04-30 |

#### employee Table:

| employee_id | department_id |
|-------------|--------------|
| 4           | 3            |
| 5           | 1            |
| 6           | 1            |

### Expected Output:

| pay_month | department_id | comparison |
|-----------|--------------|------------|
| 2022-05   | 3            | lower      |
| 2022-05   | 1            | higher     |
| 2022-04   | 3            | same       |
| 2022-04   | 1            | same       |

### Problem Constraints:
- The `salary` table records employee salary details with timestamps.
- The `employee` table maps employees to departments.
- The query should determine the department’s average salary compared to the company's average salary for each month.
- The expected output should be sorted by department and pay month.

---

## Solution Approaches

### Approach 1: Using Common Table Expressions (CTE)

#### SQL Query:
```sql
WITH CompanyAvg AS (SELECT DATE_FORMAT(pay_date, '%Y-%m') AS pay_month, AVG(amount) AS company_avg 
                    FROM salary GROUP BY pay_month),
DepartmentAvg AS (SELECT department_id, DATE_FORMAT(pay_date, '%Y-%m') AS pay_month, AVG(amount) AS department_avg 
                  FROM salary JOIN employee USING(employee_id) GROUP BY pay_month, department_id)
SELECT pay_month, department_id, CASE 
WHEN department_avg > company_avg THEN 'higher' WHEN department_avg < company_avg THEN 'lower' ELSE 'same' 
END AS comparison FROM CompanyAvg JOIN DepartmentAvg USING(pay_month) ORDER BY department_id, pay_month;
```

#### Explanation:
- Find the company's average salary by month using a CTE.
- Find the department's average salary by month using another CTE.
- Join them using `pay_month`, which is derived using `DATE_FORMAT(pay_date, '%Y-%m')`.
- Use a `CASE` clause to classify the results as "higher," "lower," or "same."

### Approach 2: Using Window Functions and Distinct

#### SQL Query:
```sql
WITH SalaryAverages as (SELECT DATE_FORMAT(pay_date, '%Y-%m') AS pay_month, department_id,
                        AVG(amount) OVER (PARTITION BY DATE_FORMAT(pay_date, '%Y-%m'), department_id) AS department_avg,
                        AVG(amount) OVER (PARTITION BY DATE_FORMAT(pay_date, '%Y-%m')) AS company_avg
                        FROM salary join employee using(employee_id))
SELECT DISTINCT pay_month, department_id, 
CASE WHEN department_avg > company_avg THEN 'higher' WHEN department_avg < company_avg THEN 'lower' ELSE 'same'
END AS comparison FROM SalaryAverages ORDER BY department_id;
```

#### Explanation:
- Calculate the department’s and company’s average salary using window functions.
- Use `PARTITION BY` to segment salaries by department and by company-wide month.
- Remove duplicate rows using `DISTINCT`.
- Apply a `CASE` clause to determine the comparison result.

---

## Performance Analysis

### Method 1 (CTE-Based):

- **Time complexity**: O(N log N) (due to grouping and joins)
- **Space complexity**: O(N) (for storing intermediate results in CTEs)

### Method 2 (Window Functions):

- **Time complexity**: O(N) (since window functions operate efficiently without additional grouping)
- **Space complexity**: O(1) (no additional storage except for output rows)

---

## Conclusion

- **Method 1**: Easier to understand but involves additional joins, which can slow performance.
- **Method 2**: More optimized by using window functions, reducing the need for explicit joins.

### Final Conclusion:
- Method 2 is generally preferable for performance reasons as it avoids unnecessary joins while still producing correct results.
