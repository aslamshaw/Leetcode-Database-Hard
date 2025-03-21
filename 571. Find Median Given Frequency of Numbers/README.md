# LeetCode Problem 571: Find Median Given Frequency of Numbers

## Problem Description

The "Numbers" table stores a list of numbers and their respective frequencies.

### Tables:

#### Numbers Table:

| Number | Frequency |
|--------|-----------|
| 0      | 8         |
| 1      | 2         |
| 2      | 5         |
| 3      | 3         |
| 4      | 1         |

In this table, the numbers and their frequencies are as follows: the number `0` appears 8 times, `1` appears 2 times, `2` appears 5 times, `3` appears 3 times, and `4` appears once. If we expand this data, we get the following list of numbers: [0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 2, 2, 2, 2, 2, 3, 3, 3, 4]. The median of this list is calculated by finding the middle number. Since the list contains 19 numbers, the median is the 10th number, which is `2`.

### Expected Output:

| median |
|--------|
| 2.0000 |

### Problem Constraints:
- You are expected to return the median of the numbers based on their frequency.
- If the count of numbers is odd, the median is the middle element.
- If the count of numbers is even, the median is the average of the two middle numbers.

---

## Solution Approaches

### Approach 1: Expanding/Decompressing the Numbers Column

In this approach, we expand the numbers based on their frequencies. This means we generate a list where each number is repeated according to its frequency. After expansion, we can calculate the median as we would normally do by ordering the numbers and finding the middle one.

#### SQL Query:
```sql
WITH RECURSIVE ExpandedNumbers AS (
    SELECT Number, Frequency, 1 AS cnt 
    FROM Numbers
    UNION ALL 
    SELECT Number, Frequency, cnt + 1 AS cnt 
    FROM ExpandedNumbers 
    WHERE cnt < Frequency
),
RankedNumbers AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY Number) AS rk, COUNT(Number) OVER () AS n 
    FROM ExpandedNumbers
)
SELECT AVG(Number) AS median 
FROM RankedNumbers 
WHERE n / 2 <= rk AND rk <= n / 2 + 1;
```

#### Explanation:
- First, we recursively expand the numbers in the "Numbers" table by repeating each number according to its frequency.
- Once expanded, we assign a row number to each number ordered by the "Number" column.
- We calculate the total count of the numbers and then identify the median by averaging the middle numbers (if the total count is even) or selecting the middle number (if the count is odd).

---

## Performance Analysis

### Method 1 (Expanding/Decompressing Numbers):

- **Time complexity**: O(N) where N is the total count of numbers across all frequencies, as we recursively expand the data.
- **Space complexity**: O(N) due to the space required for storing the expanded list of numbers.

---

## Conclusion

- **Method 1**: Expanding the numbers is a straightforward and simple approach, but it is not efficient for large datasets as it requires a lot of space and time for expansion.

### Final Conclusion:
- Expanding the list based on frequencies is a simple solution but may not be optimal for larger datasets.
