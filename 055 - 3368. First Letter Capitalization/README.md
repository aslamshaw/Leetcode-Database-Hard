# LeetCode Problem 3368: First Letter Capitalization

## Problem Description

You're given a table containing user-submitted content. Each row in the table includes a unique ID and a column `content_text` that holds a string of words. Your task is to reformat the text by converting:

- The **first letter of each word** to uppercase.
- The **rest of the letters** in each word to lowercase.
- Keep **all spaces unchanged** and in the same positions.

You can assume that `content_text` has **no special characters**, only alphabetic characters and spaces.

### Tables:

#### user_content Table:

| content_id | content_text                          |
|------------|----------------------------------------|
| 10         | welcome TO the jungle                 |
| 11         | adVANCED sql QUERIES                  |
| 12         | daTaBaSe systems AND optimizations    |
| 13         | cLean CODE and Best PRACTICES         |

### Expected Output:

| content_id | original_text                         | converted_text                         |
|------------|----------------------------------------|-----------------------------------------|
| 10         | welcome TO the jungle                 | Welcome To The Jungle                  |
| 11         | adVANCED sql QUERIES                  | Advanced Sql Queries                   |
| 12         | daTaBaSe systems AND optimizations    | Database Systems And Optimizations     |
| 13         | cLean CODE and Best PRACTICES         | Clean Code And Best Practices          |

### Problem Constraints:
- Input strings contain only alphabetic characters and spaces.
- Each word is separated by one or more spaces.
- The result must maintain the original spacing.
- `content_id` is unique for each row.

---

## Solution Approaches

### Approach 1: Recursive Word Extraction with REGEXP_SUBSTR and GROUP_CONCAT

This approach breaks each string into individual words using a recursive common table expression (CTE) and regular expressions. It then reassembles the modified words back into the final capitalized sentence using `GROUP_CONCAT`.

#### SQL Query:
```sql
WITH RECURSIVE WordList AS (
  SELECT content_id, content_text AS words, 
         REGEXP_SUBSTR(content_text, '\\b(\\w+)\\b', 1, 1) AS word, 
         1 AS occurrence 
  FROM user_content
  UNION ALL
  SELECT content_id, words, 
         REGEXP_SUBSTR(words, '\\b(\\w+)\\b', 1, occurrence + 1), 
         occurrence + 1 
  FROM WordList 
  WHERE REGEXP_SUBSTR(words, '\\b(\\w+)\\b', 1, occurrence + 1) IS NOT NULL
)

SELECT content_id, 
       words AS original_text, 
       GROUP_CONCAT(CONCAT(UPPER(SUBSTRING(word, 1, 1)), LOWER(SUBSTRING(word, 2))) 
       ORDER BY occurrence SEPARATOR ' ') AS converted_text
FROM WordList 
GROUP BY content_id, words 
ORDER BY content_id;
```

#### Explanation:
- A recursive CTE (`WordList`) is used to extract each word from the `content_text` using `REGEXP_SUBSTR`.
- Each word is processed to:
  - Convert the **first character to uppercase**.
  - Convert the **remaining characters to lowercase**.
- Words are reassembled using `GROUP_CONCAT`, ordered by their occurrence to preserve the original word order.
- The final result includes both the original text and the modified version.

---

## Performance Analysis

### Method 1 (Recursive Word Extraction with REGEXP_SUBSTR and GROUP_CONCAT):

- **Time complexity**: O(n * m), where `n` is the number of rows and `m` is the number of words per row, due to recursive extraction and string manipulation.
- **Space complexity**: O(n * m) for storing intermediate recursive results and final transformations.

---

## Conclusion

- **Method 1**: This approach is elegant and leverages SQL's recursive capabilities to handle the task effectively. It performs well with manageable data sizes but may have performance trade-offs with very large datasets due to recursive processing.

### Final Conclusion:
- The recursive method provides a clean and scalable way to manipulate and transform string data into title case while preserving word order and spacing. For SQL-based environments that support recursion, this method is a strong and reliable choice.
