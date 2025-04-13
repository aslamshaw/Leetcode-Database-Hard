# LeetCode Problem 3061: Calculate Trapping Rain Water

## Problem Description

Given a table that represents the elevation map of a landscape, where each entry corresponds to the height of a vertical bar at a specific position, determine how much rainwater can be trapped after raining. Each bar has a width of 1 unit, and water can only be trapped between taller bars.

The trapped water at a given position depends on the tallest bars to its left and right. For each position, the amount of trapped water is calculated as:

min(max_left, max_right) - height_at_position

If this value is negative or zero, no water is trapped at that position.

### Tables:

#### Heights Table:

| id | height |
|----|--------|
| 1  | 1      |
| 2  | 0      |
| 3  | 2      |
| 4  | 1      |
| 5  | 0      |
| 6  | 2      |
| 7  | 1      |
| 8  | 3      |
| 9  | 2      |
| 10 | 1      |

### Expected Output:

| total_trapped_water |
|---------------------|
| 7                   |

### Problem Constraints:
- The `id` column is the primary key and appears in a sequential order.
- Heights are non-negative integers.
- Water is only trapped between taller bars; edges cannot hold water unless bounded on both sides.
- Output can be returned in any order.

---

## Solution Approaches

### Approach 1: Window Function + Summation

This method leverages SQL **window functions** to efficiently determine the tallest bars to the left and right of each position. For each bar, we compute the water it can trap based on the minimum of these two values minus its own height.

#### SQL Query:
```sql
WITH MaxHeights AS (
    SELECT *, 
           MAX(height) OVER (ORDER BY id) AS max_left, 
           MAX(height) OVER (ORDER BY id DESC) AS max_right
    FROM Heights)

SELECT SUM(LEAST(max_left, max_right) - height) AS total_trapped_water 
FROM MaxHeights;
```

#### Explanation:
- Use `MAX(height) OVER (ORDER BY id)` to calculate the tallest bar to the **left** (including itself).
- Use `MAX(height) OVER (ORDER BY id DESC)` to calculate the tallest bar to the **right** (including itself).
- For each bar, compute the trapped water: `LEAST(max_left, max_right) - height`.
- Sum the result over all rows to get the total trapped water.
- This approach avoids any loops or complex subqueries, making it clean and efficient.

---

## Performance Analysis

### Method 1 (Window Function + Summation):

- **Time complexity**: `O(n)` – Each window function processes the table in a single pass, and summation is linear.
- **Space complexity**: `O(n)` – Additional columns for max heights are stored temporarily.

---

## Conclusion

- **Method 1**: This approach is both simple and performant, leveraging built-in SQL features. It provides a concise and scalable way to solve the problem without manual iteration or recursion.

### Final Conclusion:
The window function-based method is ideal for this problem due to its clarity, efficiency, and native SQL support. It is recommended for use in practice and interviews involving SQL-based data analysis.
