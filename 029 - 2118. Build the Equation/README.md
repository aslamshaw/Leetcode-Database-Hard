# LeetCode Problem 2118: Build the Equation

## Problem Description

The problem requires building an equation from the given terms in a specific format. The equation must satisfy the following criteria:
- The left-hand side (LHS) should contain all the terms.
- The right-hand side (RHS) should be zero.
- Each term of the LHS follows the format `<sign><fact>X^<pow>`, where:
  - `<sign>` is either `+` or `-`.
  - `<fact>` is the absolute value of the factor.
  - `<pow>` is the value of the power.
- If the power is 1, omit `^<pow>`.
- If the power is 0, omit `X` and `^<pow>`.
- The powers in the LHS are sorted in descending order.

### Tables:

#### Terms Table:

| power | factor |
|------|--------|
| 2    | 1      |
| 1    | -4     |
| 0    | 2      |

### Expected Output:

| equation       |
|----------------|
| +1X^2-4X+2=0   |

### Problem Constraints:
- Powers range from 0 to 100.
- Factors range from -100 to 100 and cannot be zero.
- The powers in the LHS must be sorted in descending order.

---

## Solution Approaches

### Approach 1: Equation Construction with String Manipulation

To build the equation, we perform the following steps:
1. For each term, determine the sign (`+` or `-`) based on the factor's value.
2. Construct the term by appending the appropriate power and variable format.
3. Use `GROUP_CONCAT` to concatenate all terms in descending order of power.
4. Append `=0` to the end of the concatenated string to complete the equation.

#### SQL Query:
```sql
WITH EquationPart AS (
    SELECT *, CASE
        WHEN power = 0 THEN IF(factor > 0, CONCAT('+', factor), factor)
        WHEN power = 1 THEN CONCAT(IF(factor > 0, CONCAT('+', factor), factor), 'X')
        WHEN power > 1 THEN CONCAT(IF(factor > 0, CONCAT('+', factor), factor), 'X^', power) 
    END AS eq_part
    FROM Terms)
SELECT CONCAT(GROUP_CONCAT(eq_part ORDER BY power DESC SEPARATOR ''), '=0') AS equation 
FROM EquationPart;
```

#### Explanation:
- The `CASE` statement handles different power scenarios (0, 1, or greater than 1) and correctly formats each term.
- The `GROUP_CONCAT` function ensures the terms are combined into a single string with the correct order.
- Using `ORDER BY power DESC` guarantees the descending order of terms.

---

## Performance Analysis

### Method 1 (Equation Construction with String Manipulation):

- **Time complexity**: O(n log n), where n is the number of terms, due to the `ORDER BY` clause.
- **Space complexity**: O(n), as we need to concatenate the terms into a single string.

---

## Conclusion

- **Method 1**: Efficient for the given problem size, leveraging SQL functions to handle string concatenation and ordering.

### Final Conclusion:
- The approach is optimal as it minimizes sorting and string operations, leveraging SQL's built-in functions effectively.
