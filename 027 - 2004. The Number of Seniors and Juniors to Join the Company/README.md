# LeetCode Problem 2004: The Number of Seniors and Juniors to Join the Company

## Problem Description
A company wants to hire new employees from a list of candidates with a limited budget of **$70,000**. Each candidate has a specific level of experience (`Senior` or `Junior`) and a corresponding monthly salary.

The company's hiring criteria are as follows:
1. **Hire the maximum number of Seniors** within the budget.
2. Use the **remaining budget** to hire the maximum number of Juniors.

### Tables:

#### Candidates Table:

| employee_id | experience | salary |
|------------|-----------|-------|
| 7          | Senior    | 30000 |
| 5          | Junior    | 12000 |
| 3          | Senior    | 18000 |
| 12         | Junior    | 9000  |
| 10         | Senior    | 25000 |
| 8          | Junior    | 15000 |

### Expected Output:

| experience | accepted_candidates |
|-----------|---------------------|
| Senior    | 2                   |
| Junior    | 2                   |

### Problem Constraints:
- The `employee_id` column is the **primary key**.
- The `experience` column is an enum with values `Senior` and `Junior`.
- The result can be returned in any order.

---

## Solution Approaches

### Approach 1: Sum of Salaries with Sorting
This approach prioritizes hiring seniors first to maximize the number of experienced candidates. If the budget permits, juniors are hired afterward to fill the remaining budget.

#### SQL Query:
```sql
WITH Seniors AS (
  SELECT *
  FROM (
    SELECT employee_id, experience, SUM(salary) OVER (ORDER BY salary, employee_id) AS cum_sal
    FROM Candidates
    WHERE experience = 'Senior') s
  WHERE cum_sal <= 70000),

RemainingBudget AS (
  SELECT 70000 - (SELECT IFNULL(MAX(cum_sal), 0) FROM Seniors) AS remaining_budget),

Juniors AS (
  SELECT *
  FROM (
    SELECT employee_id, experience, SUM(salary) OVER (ORDER BY salary, employee_id) AS cum_sal
    FROM Candidates
    WHERE experience = 'Junior') j
  WHERE cum_sal <= (SELECT remaining_budget FROM RemainingBudget))

SELECT experience, COUNT(employee_id) AS accepted_candidates
FROM (SELECT 'Senior' AS experience UNION SELECT 'Junior') sj1
LEFT JOIN (
  SELECT * FROM Seniors
  UNION ALL
  SELECT * FROM Juniors) sj2 
USING (experience)
GROUP BY experience;
```

#### Explanation:
1. **Select Seniors**: Sort seniors by salary and compute a cumulative sum to fit within the budget.
2. **Calculate Remaining Budget**: Subtract the total salary of hired seniors from the budget.
3. **Select Juniors**: Sort juniors by salary and compute a cumulative sum to fit within the remaining budget.
4. **Combine Results**: Return the number of seniors and juniors hired.

---

## Performance Analysis

### Method 1 (Sum of Salaries with Sorting):
- **Time complexity**: O(n log n), due to sorting the candidates based on salary.
- **Space complexity**: O(n), for storing intermediate results.

---

## Conclusion
- **Method 1**: Efficient for maximizing the number of seniors first, followed by juniors using the remaining budget. The cumulative sum approach is effective in maintaining budget constraints.
- Final Recommendation: This approach is optimal given the hiring criteria and budget limitations.
