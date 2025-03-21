WITH CTE AS (SELECT *, DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS ranking FROM employee)
SELECT d.name AS Department, cte.name AS Employee, cte.salary FROM CTE 
JOIN department AS d ON cte.departmentId = d.id WHERE ranking < 4;
