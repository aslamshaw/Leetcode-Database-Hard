# LeetCode Problem 1767: Find the Subtasks That Did Not Execute

## Problem Description

Given two tables, **Tasks** and **Executed**, find the subtasks that did not execute for each task. The **Tasks** table contains information about each task and the number of subtasks it was divided into, while the **Executed** table records which subtasks were successfully executed. Your goal is to identify and list the missing subtasks for each task.

### Tables:

#### Tasks Table:

| task_id | subtasks_count |
|--------|----------------|
| 10     | 4              |
| 20     | 3              |
| 30     | 5              |

#### Executed Table:

| task_id | subtask_id |
|--------|------------|
| 10     | 1          |
| 10     | 3          |
| 30     | 2          |
| 30     | 4          |
| 30     | 5          |

### Expected Output:

| task_id | subtask_id |
|--------|------------|
| 10     | 2          |
| 10     | 4          |
| 20     | 1          |
| 20     | 2          |
| 20     | 3          |
| 30     | 1          |
| 30     | 3          |

### Problem Constraints:
- Each task is divided into a number of subtasks ranging from 2 to 20.
- The **task_id** and **subtask_id** together form the primary key in the **Executed** table.
- It is guaranteed that **subtask_id** is always less than or equal to **subtasks_count**.
- The result can be returned in any order.

---

## Solution Approaches

### Approach 1: Recursive CTE to Generate Missing Subtasks

To identify the missing subtasks for each task, we use a **Common Table Expression (CTE)** with recursion. The CTE generates all possible subtask IDs for each task and then identifies the missing ones by performing a **LEFT JOIN** with the `Executed` table.

#### SQL Query:
```sql
WITH RECURSIVE Subtasks AS (
    SELECT task_id, subtasks_count AS subtask_id
    FROM Tasks
    UNION ALL
    SELECT task_id, subtask_id - 1
    FROM Subtasks
    WHERE subtask_id > 1
)
SELECT s.task_id, s.subtask_id
FROM Subtasks s
LEFT JOIN Executed e ON s.task_id = e.task_id AND s.subtask_id = e.subtask_id
WHERE e.task_id IS NULL
ORDER BY s.task_id, s.subtask_id;
```

#### Explanation:
- The **recursive CTE** generates a list of subtask IDs for each task, starting from the maximum subtask count and decrementing until 1.
- The **base case** of the recursion selects the maximum subtask ID for each task.
- The **recursive step** generates the next lower subtask ID until it reaches 1.
- The final query performs a **LEFT JOIN** with the `Executed` table to find the subtasks that were not executed.
- Results are ordered by `task_id` and `subtask_id` to match the expected output.

## Performance Analysis

### Method 1 (Recursive CTE to Generate Missing Subtasks):
- **Time complexity**: O(n × m), where **n** is the number of tasks and **m** is the maximum number of subtasks per task.
- **Space complexity**: O(n × m), for storing the intermediate results.

---

## Conclusion
- **Method 1**: The recursive CTE approach is efficient for handling tasks with a moderate number of subtasks. It avoids the cartesian product issue and leverages recursion to generate subtasks dynamically. However, recursion depth may impact performance if the number of subtasks is very high.

### Final Conclusion:
- The recursive CTE approach is efficient for moderate input sizes, but recursion depth could affect performance if the number of subtasks is extremely large. Optimizations might include breaking the task into smaller parts or using iterative methods. Further optimizations could involve indexing or using more advanced SQL constructs.
