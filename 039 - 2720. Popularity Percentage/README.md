# LeetCode Problem 2720: Popularity Percentage

## Problem Description

Given a `Friends` table representing friendship relationships between users on a social media platform, you are to compute the **popularity percentage** of each user. 

Popularity percentage is defined as:
> (Number of unique friends a user has / Total number of unique users on the platform) × 100

The result should display each user along with their popularity percentage, rounded to two decimal places. The output must be sorted by the user ID in ascending order.

### Tables:

#### Friends Table:

| user1 | user2 |
|-------|-------|
| 10    | 11    |
| 11    | 12    |
| 13    | 11    |
| 11    | 14    |
| 11    | 15    |
| 10    | 15    |
| 16    | 10    |
| 17    | 12    |
| 12    | 18    |

### Expected Output:

| user1 | percentage_popularity |
|-------|------------------------|
| 10    | 33.33                  |
| 11    | 55.56                  |
| 12    | 33.33                  |
| 13    | 11.11                  |
| 14    | 11.11                  |
| 15    | 22.22                  |
| 16    | 11.11                  |
| 17    | 11.11                  |
| 18    | 11.11                  |

### Problem Constraints:
- The `Friends` table represents undirected friendships. If user A is friends with B, both A and B are considered friends of each other.
- Users must be counted only once in the total platform user base.
- Output must show all users involved in any friendship either as `user1` or `user2`.

---

## Solution Approaches

### Approach 1: Normalize Friendships and Count Relations

This approach handles the undirected nature of friendships by standardizing entries so that each friendship is counted for both users.

#### SQL Query:
```sql
WITH BDR AS (
    SELECT user1, user2 FROM Friends
    UNION
    SELECT user2, user1 FROM Friends),
    
Users AS (
    SELECT COUNT(DISTINCT user1) AS total FROM BDR)

SELECT user1, ROUND(COUNT(*) / (SELECT total FROM Users) * 100, 2) AS percentage_popularity
FROM BDR
GROUP BY user1
ORDER BY user1;
```

#### Explanation:
- **Step 1**: Use a `UNION` operation to combine both `(user1, user2)` and `(user2, user1)` rows. This ensures every friendship is considered bidirectional.
- **Step 2**: Count the **total unique users** using a CTE.
- **Step 3**: Group by each `user1` to count the number of friends each has.
- **Step 4**: Calculate the popularity percentage using the total unique user count.
- **Step 5**: Round the result to two decimal places for clarity and format.

---

## Performance Analysis

### Method 1 (Normalize Friendships and Count Relations):

- **Time complexity**: O(N), where N is the number of friendship records. The `UNION`, `GROUP BY`, and aggregation operations all operate in linear time relative to the data.
- **Space complexity**: O(U), where U is the number of unique users, due to storage required for user relationships and intermediate CTEs.

---

## Conclusion

- **Method 1**: Effectively handles the bidirectional nature of friendships and ensures accurate calculation of popularity percentages. The use of CTEs improves readability and maintains logical separation of tasks.

### Final Conclusion:
- This method is efficient and clear, making it a reliable and scalable solution for calculating user popularity metrics on a platform like Meta/Facebook.
