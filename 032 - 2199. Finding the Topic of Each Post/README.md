# LeetCode Problem 2199: Finding the Topic of Each Post

## Problem Description

Given a set of social media posts and a list of keywords that represent different topics, the task is to determine which topic(s) each post relates to based on the presence of any of these keywords. A post can relate to multiple topics if it includes keywords from each of them, and all keyword matching should be case-insensitive. If a post contains no recognizable keywords, it is considered ambiguous.

### Tables:

#### Keywords Table:

| topic_id | word       |
|----------|------------|
| 10       | quantum    |
| 10       | physics    |
| 20       | biology    |
| 30       | pandemic   |

#### Posts Table:

| post_id | content                                                                 |
|---------|-------------------------------------------------------------------------|
| 101     | Quantum mechanics is a fascinating area of physics                      |
| 102     | The biology exam was harder than expected                               |
| 103     | Is the pandemic over or will we see another wave                        |
| 104     | I enjoy hiking and painting on weekends                                 |

### Expected Output:

| post_id | topic       |
|---------|-------------|
| 101     | 10          |
| 102     | 20          |
| 103     | 30          |
| 104     | Ambiguous!  |

### Problem Constraints:
- A keyword may belong to multiple topics.
- A topic may have multiple keywords.
- Matching is **case-insensitive** and should only match **whole words**.
- Output must list topic IDs in **ascending order**, separated by commas, or `"Ambiguous!"` if none match.

---

## Solution Approaches

### Approach 1: Regex Pattern Matching using `REGEXP_LIKE`

This solution uses a `LEFT JOIN` between the `Posts` and `Keywords` tables, based on whether the `content` of each post contains a keyword using the `REGEXP_LIKE` function.

#### SQL Query:
```sql
SELECT post_id, 
IFNULL(GROUP_CONCAT(DISTINCT topic_id ORDER BY topic_id SEPARATOR ','), 'Ambiguous!') AS topic
FROM Posts LEFT JOIN Keywords 
ON REGEXP_LIKE(LOWER(content), CONCAT('\\b', LOWER(word), '\\b')) GROUP BY post_id;
```

#### Explanation:
- `REGEXP_LIKE` returns `TRUE` if the post content matches the keyword as a full word.
- To ensure case-insensitive matching, both `content` and `word` are converted to lowercase using `LOWER()`.
- Word boundaries (`\b`) are added using `CONCAT('\\b', LOWER(k.word), '\\b')` to ensure the match is against whole words and not substrings.
- The `LEFT JOIN` ensures all posts are included, even if they don’t match any keyword.
- `GROUP_CONCAT` is used to collect multiple matching topic IDs into a comma-separated string.
- `DISTINCT` removes duplicate topic IDs.
- `ORDER BY` inside `GROUP_CONCAT` ensures topic IDs are sorted.
- `IFNULL(..., 'Ambiguous!')` handles posts that don't match any topic.

---

## Performance Analysis

### Method 1 (Regex Pattern Matching using `REGEXP_LIKE`):

- **Time complexity**: O(N × M), where N is the number of posts and M is the number of keywords. Each post potentially checks against all keywords.
- **Space complexity**: O(N) for storing output, and additional space may be used internally by the regex engine.

---

## Conclusion

- **Method 1**: Efficient in terms of implementation and readability for small to moderately sized datasets. May face performance issues with a very large number of posts and keywords due to regex evaluation cost.

### Final Conclusion:
- For problems involving dynamic text pattern matching and keyword association, using `REGEXP_LIKE` is a robust and flexible approach. Despite potential overhead with large datasets, it provides accurate full-word, case-insensitive matching and cleanly handles multi-keyword and multi-topic associations.
