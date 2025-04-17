# LeetCode Problem 3188: Find Top Scoring Students II

## Problem Description

The task is to identify students who meet the following academic criteria within their major:

1. They have successfully completed **all mandatory courses** for their major with a **grade of A**.
2. They have completed **at least two elective courses** (non-mandatory) in their major with grades of **A or B**.
3. Their **overall GPA** across all enrolled courses (regardless of major) is **at least 2.5**.

Return a list of student IDs who satisfy all these conditions, ordered by `student_id` in ascending order.

### Tables:

#### students Table:

| student_id | name     | major            |
|------------|----------|------------------|
| 1          | Emma     | Computer Science |
| 2          | Noah     | Computer Science |
| 3          | Olivia   | Mathematics      |
| 4          | Liam     | Mathematics      |

#### courses Table:

| course_id | name              | credits | major            | mandatory |
|-----------|-------------------|---------|------------------|-----------|
| 201       | Programming       | 3       | Computer Science | Yes       |
| 202       | Databases         | 3       | Computer Science | Yes       |
| 203       | Trigonometry      | 4       | Mathematics      | Yes       |
| 204       | Discrete Math     | 4       | Mathematics      | Yes       |
| 205       | AI Basics         | 3       | Computer Science | No        |
| 206       | Combinatorics     | 3       | Mathematics      | No        |
| 207       | Web Dev           | 3       | Computer Science | No        |
| 208       | Statistics I      | 3       | Mathematics      | No        |

#### enrollments Table:

| student_id | course_id | semester     | grade | GPA |
|------------|-----------|--------------|-------|-----|
| 1          | 201       | Fall 2023    | A     | 4.0 |
| 1          | 202       | Spring 2023  | A     | 4.0 |
| 1          | 205       | Spring 2023  | A     | 4.0 |
| 1          | 207       | Fall 2023    | B     | 3.5 |
| 2          | 201       | Fall 2023    | A     | 4.0 |
| 2          | 202       | Spring 2023  | B     | 3.0 |
| 3          | 203       | Fall 2023    | A     | 4.0 |
| 3          | 204       | Spring 2023  | A     | 4.0 |
| 3          | 206       | Spring 2023  | A     | 4.0 |
| 3          | 208       | Fall 2023    | B     | 3.5 |
| 4          | 203       | Fall 2023    | B     | 3.0 |
| 4          | 204       | Spring 2023  | B     | 3.0 |

### Expected Output:

| student_id |
|------------|
| 1          |
| 3          |

### Problem Constraints:
- `student_id`, `course_id`, and `semester` combinations are unique in the enrollments table.
- A student may enroll in multiple courses per semester.
- GPA values are decimal numbers between 0.0 and 4.0.
- Mandatory field values are enums: 'Yes' or 'No'.
- Grades are limited to standard letter grades ('A', 'B', etc.).

---

## Solution Approaches

### Approach 1: Join and Aggregate with GPA & Grade Filtering

This approach uses a combination of CTEs and aggregations to identify students who meet all the criteria in one pass.

#### SQL Query:
```sql
WITH AboveAvgStudents AS (
        SELECT student_id
        FROM enrollments
        GROUP BY 1
        HAVING AVG(GPA) >= 2.5)
        
SELECT student_id
FROM
    AboveAvgStudents
    JOIN students USING (student_id)
    JOIN courses USING (major)
    LEFT JOIN enrollments USING (student_id, course_id)
GROUP BY student_id
HAVING SUM(mandatory = 'Yes') = SUM(mandatory = 'Yes' AND grade = 'A')
AND SUM(mandatory = 'No' AND grade IS NOT NULL) = SUM(mandatory = 'No' AND grade IN ('A', 'B'))
AND SUM(mandatory = 'No' AND grade IS NOT NULL) > 1
```

#### Explanation:
- **AboveAvgStudents CTE**: First, compute the students who have an average GPA of 2.5 or higher across all courses.
- **Main Query**:
  - Join the `students` and `courses` tables based on their major, and then join with `enrollments` to fetch each student's grades.
  - For each student, aggregate the number of mandatory courses and check that all were graded A.
  - Count elective courses with grades of A or B, ensuring there are at least two.
  - Filter students who meet all three conditions.

This strategy filters early using GPA, reducing computation in later joins and groupings.

---

## Performance Analysis

### Method 1 (Join and Aggregate with GPA & Grade Filtering):

- **Time complexity**: O(n), where n is the total number of enrollments, assuming joins and groupings are indexed and optimized. Each enrollment is scanned once.
- **Space complexity**: O(n), primarily for intermediate join results and aggregations.

---

## Conclusion

- **Method 1**: The approach is effective and efficient. It smartly segments the task into GPA filtering and detailed checks using aggregations. This keeps the logic concise and avoids unnecessary passes through the data.

### Final Conclusion:
- This method is optimal for solving the problem with large datasets due to its early filtering and aggregated validation logic. It's the recommended approach for this use case.
