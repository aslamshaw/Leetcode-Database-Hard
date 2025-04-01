# LeetCode Problem 1919: Leetcodify Similar Friends

## Problem Description

In this problem, you're given two tables: **Listens** and **Friendship**. You are tasked to find similar friends based on the following criteria:

1. Users x and y are friends.
2. Users x and y listened to at least three of the same songs on the same day.

You must return all such pairs of friends, ordered by `user1_id < user2_id`.

### Tables:

#### Listens Table:

| user_id | song_id | day        |
|---------|---------|------------|
| 1       | 10      | 2021-03-15 |
| 1       | 11      | 2021-03-15 |
| 1       | 12      | 2021-03-15 |
| 2       | 10      | 2021-03-15 |
| 2       | 11      | 2021-03-15 |
| 2       | 12      | 2021-03-15 |
| 3       | 10      | 2021-03-15 |
| 3       | 11      | 2021-03-15 |
| 3       | 12      | 2021-03-15 |
| 4       | 10      | 2021-03-15 |
| 4       | 11      | 2021-03-15 |
| 4       | 13      | 2021-03-15 |
| 5       | 10      | 2021-03-16 |
| 5       | 11      | 2021-03-16 |
| 5       | 12      | 2021-03-16 |

#### Friendship Table:

| user1_id | user2_id |
|----------|----------|
| 1        | 2        |
| 2        | 4        |
| 2        | 5        |

### Expected Output:

| user1_id | user2_id |
|----------|----------|
| 1        | 2        |

### Problem Constraints:
- The input contains multiple entries for the same `user_id`, `song_id`, and `day`.
- The result must show pairs of users that listened to at least three songs in common on the same day, and those users should be friends.
- The result should return the pairs in any order but must ensure `user1_id < user2_id`.

---

## Solution Approaches

### Approach 1: Grouping and Counting Similar Songs

This approach involves finding the users who listened to the same songs on the same day, and filtering those who are friends and listened to at least three common songs.

#### SQL Query:
```sql
WITH Recommended AS (
    SELECT l1.user_id AS user1_id, l2.user_id AS user2_id, l1.song_id, l1.day
    FROM Listens l1
    JOIN Listens l2 ON l1.day = l2.day 
                   AND l1.song_id = l2.song_id 
                   AND l1.user_id != l2.user_id
),
Friends AS (
    SELECT r.user1_id, r.user2_id, r.song_id, r.day
    FROM Recommended r
    JOIN Friendship f ON r.user1_id = f.user1_id AND r.user2_id = f.user2_id
)
SELECT user1_id, user2_id  
FROM Friends  
GROUP BY user1_id, user2_id, day HAVING COUNT(*) > 2;
```

#### Explanation:
- **Step 1:** Identify pairs of users who listened to the same song on the same day.
- **Step 2:** Filter out pairs who are friends using the Friendship table.
- **Step 3:** Group by user pairs and day to check if they listened to at least three songs together on the same day.

---

## Performance Analysis

### Method 1 (Grouping and Counting Similar Songs):

- **Time complexity:** O(N^2) due to the need to compare each user’s song choices across multiple days.
- **Space complexity:** O(N) for storing the intermediate results.

---

## Conclusion

- **Method 1:** This method works well for problems where checking shared conditions (like songs on the same day) between two users is necessary. However, the quadratic time complexity could become a bottleneck for very large datasets.

### Final Conclusion:
- This approach is efficient and intuitive for this problem but may need optimization for very large datasets.
