# LeetCode Problem 2010: The Number of Seniors and Juniors to Join the Company II

## Problem Description
A company aims to hire new employees within a budget of $70,000. Each candidate is classified as either a **Senior** or a **Junior** and has a unique salary. The hiring criteria are as follows:

1. Hire as many seniors as possible starting from the one with the **smallest salary**.
2. If the remaining budget allows, hire as many juniors as possible starting from the one with the **smallest salary**.

Return the IDs of the seniors and juniors hired according to the given criteria.

### Tables:
#### Candidates Table:
| employee_id | experience | salary |
|------------|------------|-------|
| 7          | Senior     | 12000 |
| 3          | Junior     | 10000 |
| 15         | Senior     | 30000 |
| 9          | Junior     | 15000 |
| 21         | Senior     | 50000 |
| 12         | Junior     | 40000 |

### Expected Output:
| employee_id |
|------------|
| 7          |
| 15         |
| 3          |
| 9          |

### Problem Constraints:
- Each candidate has a **unique salary**.
- The budget for hiring is **$70,000**.
- The company prioritizes hiring **seniors with the smallest salaries first**, followed by **juniors with the smallest salaries**.

---

## Solution Approaches

### Approach 1: Cumulative Sum with Prioritized Hiring
The goal is to maximize the number of hires by selecting candidates with the smallest salaries. We divide the problem into the following steps:
1. **Identify and prioritize senior candidates** based on the smallest salary.
2. **Track cumulative salary** to ensure it does not exceed the budget.
3. **Calculate remaining budget** after hiring seniors.
4. **Identify and hire juniors** with the remaining budget.
5. **Return the IDs of the hired candidates**.

#### SQL Query:
```sql
WITH Seniors AS (
    SELECT *
    FROM (SELECT *, SUM(salary) OVER (ORDER BY salary, employee_id) AS cs 
          FROM Candidates WHERE experience = 'Senior') s
    WHERE cs <= 70000),
RemainingBudget AS (SELECT 70000 - IFNULL(MAX(cs), 0) AS remaining_budget FROM Seniors),
Juniors AS (
    SELECT * 
    FROM (SELECT *, SUM(salary) OVER (ORDER BY salary, employee_id) AS cs 
          FROM Candidates WHERE experience = 'Junior') j
    WHERE cs <= (SELECT remaining_budget FROM RemainingBudget))
SELECT employee_id FROM Seniors UNION ALL SELECT employee_id FROM Juniors
```

#### Explanation:
- We use **cumulative sum (cs)** to calculate the total cost while iterating through sorted candidates.
- First, we hire as many **seniors as possible**. Once the budget is exhausted or cannot accommodate another senior, we move on to **juniors**.
- We utilize the `SUM()` window function to efficiently calculate the cumulative salary while ordering by **salary** and **employee_id**.
- The `IFNULL` function handles cases where no seniors can be hired, allowing us to move directly to hiring juniors.

---

## Performance Analysis

### Method 1 (Cumulative Sum with Prioritized Hiring):
- **Time complexity**: O(n log n) due to sorting by salary and employee_id.
- **Space complexity**: O(n) for storing intermediate cumulative sums.

---

## Conclusion
- **Method 1**: Efficiently maximizes the number of hired candidates within the budget. The use of cumulative sums minimizes redundancy, and sorting ensures that the smallest salaries are considered first.

### Final Conclusion:
- This method is both **efficient and optimal** for the given problem constraints. The logic is clear and leverages SQL window functions effectively to minimize computational overhead.
