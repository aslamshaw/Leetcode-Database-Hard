# LeetCode Problem 3374: First Letter Capitalization II

## Problem Description

Given a dataset of text content submitted by users, you are required to format the `content_text` column based on specific capitalization rules:

1. Capitalize the **first letter of each word** and convert all **other letters to lowercase**.
2. For **hyphenated words**, both parts of the word should be capitalized (e.g., `well-known` becomes `Well-Known`).
3. Maintain the **original spacing** and preserve any special formatting such as hyphens.
4. Only alphabetic characters and hyphens are present in the text. No other special characters will be included.

Return the result with the original text and the newly formatted version.

### Tables:

#### user_content Table:

| content_id | content_text                          |
|------------|----------------------------------------|
| 101        | master-of SQL                         |
| 102        | quick-WINS in coding                  |
| 103        | new-age TECHNOLOGY solutions          |
| 104        | object-oriented FRONT-end DESIGN      |

### Expected Output:

| content_id | original_text                         | converted_text                         |
|------------|----------------------------------------|-----------------------------------------|
| 101        | master-of SQL                         | Master-Of Sql                          |
| 102        | quick-WINS in coding                  | Quick-Wins In Coding                   |
| 103        | new-age TECHNOLOGY solutions          | New-Age Technology Solutions           |
| 104        | object-oriented FRONT-end DESIGN      | Object-Oriented Front-End Design       |

### Problem Constraints:
- Input strings may contain words joined by hyphens (`-`), which must be capitalized on both sides.
- Other than hyphens and letters, no special characters are present.
- Spacing between words is preserved exactly as in the input.
- `content_id` is unique for each row.

---

## Solution Approaches

### Approach 1: Recursive Word and Hyphen Parsing with REGEXP_SUBSTR and Reassembly

This solution builds on recursive extraction of both words and hyphens from the input strings. Using a recursive CTE, it handles splitting words, capitalizing correctly (even across hyphen boundaries), and then reconstructing the sentence.

#### SQL Query:
```sql
WITH RECURSIVE WordList AS (
  SELECT content_id, content_text AS words, 
         REGEXP_SUBSTR(content_text, '\\b(\\w+)|-\\b', 1, 1) AS word, 
         1 AS occurrence 
  FROM user_content
  UNION ALL
  SELECT content_id, words, 
         REGEXP_SUBSTR(words, '\\b(\\w+)|-\\b', 1, occurrence + 1), 
         occurrence + 1 
  FROM WordList 
  WHERE REGEXP_SUBSTR(words, '\\b(\\w+)|-\\b', 1, occurrence + 1) IS NOT NULL),

ConvertedText as (SELECT content_id, 
      words AS original_text, 
      GROUP_CONCAT(CONCAT(UPPER(SUBSTRING(word, 1, 1)), LOWER(SUBSTRING(word, 2))) 
      ORDER BY occurrence SEPARATOR ' ') AS converted_text
FROM WordList 
GROUP BY content_id, words)

SELECT content_id, original_text, REPLACE(converted_text, ' - ', '-') as converted_text
FROM ConvertedText ORDER BY content_id;
```

#### Explanation:
- A recursive common table expression (`WordList`) extracts each word or hyphen component using a regular expression pattern that accounts for both word boundaries and hyphen connections.
- Each extracted word is processed by:
  - Capitalizing the **first letter**.
  - Lowercasing the **remaining letters**.
- Reassembled text is created using `GROUP_CONCAT` to join the transformed words in order of appearance.
- As a final formatting step, any incorrectly spaced hyphens (like `" - "`) are corrected back to `"-"` using a `REPLACE`.

---

## Performance Analysis

### Method 1 (Recursive Word and Hyphen Parsing with REGEXP_SUBSTR and Reassembly):

- **Time complexity**: O(n * m), where `n` is the number of rows and `m` is the number of tokens (words or hyphen groups) per row.
- **Space complexity**: O(n * m), due to the recursive breakdown and temporary storage of tokens for each row.

---

## Conclusion

- **Method 1**: This method offers a precise and flexible approach to handling mixed-case words and hyphenated structures. By leveraging recursion and regular expressions, it efficiently transforms each row of text while preserving the original formatting.

### Final Conclusion:
- This approach is well-suited for database systems supporting recursive queries. It ensures accurate formatting, especially for complex cases involving hyphenated words, without compromising on performance or text fidelity.
