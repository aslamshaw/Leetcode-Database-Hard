# LeetCode Problem 1412: Find the Quiet Students in All Exams

## Problem Description

We are given two tables, `Student` and `Exam`. We need to identify students who are considered "quiet". A "quiet" student is defined as a student who:

1. Has taken at least one exam.
2. Has **never** scored the highest or the lowest score in **any** of the exams they participated in.

We need to return the `student_id` and `student_name` for all quiet students, ordered by their `student_id`.

### Tables:

#### Student Table:

| student_id | student_name |
|------------|--------------|
| 1          | Daniel       |
| 2          | Jade         |
| 3          | Stella       |
| 4          | Jonathan     |
| 5          | Will         |

#### Exam Table:

| exam_id | student_id | score |
|---------|------------|-------|
| 10      | 1          | 70    |
| 10      | 2          | 80    |
| 10      | 3          | 90    |
| 20      | 1          | 80    |
| 30      | 1          | 70    |
| 30      | 3          | 80    |
| 30      | 4          | 90    |
| 40      | 1          | 60    |
| 40      | 2          | 70    |
| 40      | 4          | 80    |

### Expected Output:

| student_id | student_name |
|------------|--------------|
| 2          | Jade         |

### Problem Constraints:
- We need to filter out students who have never taken any exam.
- We should identify those who are not the highest or lowest scorer in any of the exams they participated in.

---

## Solution Approaches

### Approach 1: Using `DENSE_RANK()`

This approach uses the `DENSE_RANK()` window function to rank students based on their scores in each exam. By calculating rankings for the highest and lowest scores, we can filter out students who rank 1 for either the highest or lowest score.

#### SQL Query:
```sql
    WITH RankedScores AS (SELECT student_id,
            DENSE_RANK() OVER (PARTITION BY exam_id ORDER BY score) AS lowest_rank,
            DENSE_RANK() OVER (PARTITION BY exam_id ORDER BY score DESC) AS highest_rank FROM Exam)
    SELECT s.student_id, s.student_name
    FROM RankedScores r JOIN Student s ON r.student_id = s.student_id GROUP BY s.student_id
    HAVING SUM(lowest_rank = 1) = 0 AND SUM(highest_rank = 1) = 0 ORDER BY s.student_id;
```

#### Explanation:
- **Ranking**: For each exam, `DENSE_RANK()` is used to rank students based on their scores, both ascending and descending (to handle lowest and highest scores).
- **Filter**: After ranking, we can simply exclude students who rank 1 for either the highest or lowest score in any exam. 
- This is efficient because `DENSE_RANK()` ensures that we are not repeatedly scanning the entire table for each student's score.

### Approach 2: Transforming Min and Max Scores and Summing the Condition

This approach involves calculating the highest and lowest scores for each exam and checking whether a student’s score is equal to these values.

#### SQL Query:
```sql
    WITH MinMaxScore AS (SELECT *, MAX(score) OVER (PARTITION BY exam_id) AS max_score,
                                   MIN(score) OVER (PARTITION BY exam_id) AS min_score FROM Exam)
    SELECT mms.student_id, s.student_name FROM MinMaxScore mms JOIN Student s ON mms.student_id = s.student_id
    GROUP BY mms.student_id HAVING SUM(mms.score = mms.max_score) = 0 AND SUM(mms.score = mms.min_score) = 0;
```

#### Explanation:
- **MinMax Calculation**: For each exam, we calculate the minimum and maximum scores using window functions (`MIN()` and `MAX()`).
- **Filtering**: In the `HAVING` clause, we filter out students whose score matches either the `max_score` or `min_score` for any exam. If they do, they are excluded from the results.

### Approach 3: Using `CASE WHEN` with Min and Max Scores

#### SQL Query:
```sql
  WITH MinMaxScore AS (SELECT *, MAX(score) OVER (PARTITION BY exam_id) AS max_score, 
                                 MIN(score) OVER (PARTITION BY exam_id) AS min_score FROM Exam)
  SELECT student_id, student_name FROM MinMaxScore JOIN Student USING(student_id) GROUP BY student_id 
  HAVING SUM(CASE WHEN score = max_score OR score = min_score THEN 1 ELSE 0 END) = 0
```

This approach is similar to the previous one but uses a `CASE WHEN` clause to explicitly check whether a student's score matches the highest or lowest score for each exam.

#### Explanation:
- **MinMax Calculation**: As in the previous approach, calculate the minimum and maximum scores for each exam.
- **Condition Checking**: Use `CASE WHEN` to check if a student’s score matches the min or max score for each exam. If a student’s score matches the min or max score, they are excluded from the result.

---

## Performance Analysis

### Method 1 (Using `DENSE_RANK()`):

- **Time complexity**: O(n log n) - The time complexity is dominated by the window function `DENSE_RANK()`, which requires sorting the data.
- **Space complexity**: O(n) - The space complexity is due to the need to store the rankings and the intermediate data in memory.

### Method 2 (Transforming Min and Max Scores and Summing the Condition):

- **Time complexity**: O(n) - The time complexity is linear because we are just calculating the min and max scores using window functions, and then filtering based on the sum of conditions.
- **Space complexity**: O(n) - We store the intermediate calculations for each row of data.

### Method 3 (Using `CASE WHEN` with Min and Max Scores):

- **Time complexity**: O(n) - Similar to the previous approach, the complexity is linear due to the use of window functions and `CASE WHEN`.
- **Space complexity**: O(n) - The space complexity is the same as the previous approach because we store intermediate results.

---

## Conclusion

- **Method 1 (Using `DENSE_RANK()`)**: This approach provides a very clear and efficient way of solving the problem using ranking. It is optimal for handling tie situations in exam scores.
- **Method 2 (Transforming Min and Max Scores)**: This approach is simple and direct, but it can be slightly less intuitive when handling edge cases like multiple students with the same score.
- **Method 3 (Using `CASE WHEN`)**: This method is a variation of the second one and is helpful when you want to explicitly check conditions in the query logic.

### Final Conclusion:
The best approach would depend on the preference for clarity or simplicity. **Method 1 (Using `DENSE_RANK()`)** is generally the most elegant and efficient for this problem, especially when dealing with tie situations in rankings.
