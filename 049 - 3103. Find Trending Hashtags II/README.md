# LeetCode Problem 3103: Find Trending Hashtags II

## Problem Description

You're given a list of tweets made by users during February 2024. Each tweet may contain one or more hashtags, and your task is to identify the **top 3 most frequently used hashtags** during that month.

Hashtags are defined as any word prefixed by a `#` symbol within the tweet. Each hashtag should be counted every time it appears across tweets, regardless of its position or case sensitivity.

Return a table with two columns:
- `hashtag`: The hashtag text.
- `count`: How many times the hashtag appeared.

Sort the output by:
1. The count in **descending order**.
2. The hashtag **lexicographically descending** if counts are the same.

### Tables:

#### Tweets Table:

| user_id | tweet_id | tweet                                                  | tweet_date |
|---------|----------|--------------------------------------------------------|------------|
| 201     | 301      | Another victory! #WinBig #SuccessStory                 | 2024-02-02 |
| 202     | 302      | Late night coding grind. #CodeLife #NightOwl          | 2024-02-03 |
| 203     | 303      | Powering through deadlines. #WorkMode #Focus          | 2024-02-04 |
| 204     | 304      | Grateful for wins. #SuccessStory #GrindTime           | 2024-02-05 |
| 205     | 305      | Embracing the challenge. #WorkMode #GrowthMindset     | 2024-02-06 |
| 206     | 306      | Brainstorming next big idea. #Innovate #Create        | 2024-02-07 |
| 207     | 307      | Hustle and repeat. #GrindTime #Discipline             | 2024-02-08 |

### Expected Output:

| hashtag        | count |
|----------------|-------|
| #SuccessStory  | 2     |
| #WorkMode      | 2     |
| #GrindTime     | 2     |

### Problem Constraints:
- Every tweet may contain zero or more hashtags.
- All tweet dates are guaranteed to fall within February 2024.
- Return only the **top 3 hashtags** based on the frequency of occurrence.
- Each hashtag is case-sensitive in terms of display but counted uniformly if appearing more than once.

---

## Solution Approaches

### Approach 1: Recursive Hashtag Extraction and Frequency Count

This method uses a recursive common table expression (CTE) to extract each hashtag from the tweet content using regular expressions. It then aggregates and ranks them by their frequency.

#### SQL Query:
```sql
WITH RECURSIVE HashTagList AS (
  SELECT tweet_id, tweet, 
         REGEXP_SUBSTR(tweet, '#(\\w+)', 1, 1) AS hashtag, 
         1 AS occurrence 
  FROM Tweets
  UNION ALL
  SELECT tweet_id, tweet, 
         REGEXP_SUBSTR(tweet, '#(\\w+)', 1, occurrence + 1), 
         occurrence + 1 
  FROM HashTagList 
  WHERE REGEXP_SUBSTR(tweet, '#(\\w+)', 1, occurrence + 1) IS NOT NULL)
SELECT hashtag, COUNT(*) AS count 
FROM HashTagList 
GROUP BY hashtag
ORDER BY count DESC, hashtag DESC 
LIMIT 3;
```

#### Explanation:
- A `REGEXP_SUBSTR` pattern is used to recursively extract each hashtag from the tweet string.
- The recursion continues until no more hashtags are found within a tweet.
- Hashtags are counted using `GROUP BY`, and results are sorted and limited to the top 3 based on the given rules.

---

## Performance Analysis

### Method 1 (Recursive Hashtag Extraction and Frequency Count):

- **Time complexity**: O(n * m), where `n` is the number of tweets and `m` is the average number of hashtags per tweet.
- **Space complexity**: O(n * m), due to intermediate storage of extracted hashtags.

---

## Conclusion

- **Method 1**: This solution efficiently parses hashtags from each tweet using recursive SQL constructs. It ensures all hashtags are counted accurately and returned in the desired sorted format.

### Final Conclusion:
- The recursive approach is both scalable and reliable for parsing multiple hashtags per tweet, making it well-suited for this type of analysis.

---

### ⚠️ Disclaimer: On Incorrect Regex Usage

The pattern `\\b#(\\w+)\\b` is commonly assumed to match hashtags but **fails in practical use cases** due to misuse of word boundaries.

#### Why it fails:
- The `\b` word boundary doesn't treat `#` as a word character.
- So, `\b#` fails to match hashtags like `#Goal` in "Today is a #Goal day" because the boundary doesn't exist between space and `#`.
