# LeetCode Problem 1917: Leetcodify Friends Recommendations

## Problem Description
In this problem, we need to recommend friends to Leetcodify users based on their listening history. A friend recommendation occurs if:
- Users **x** and **y** are not friends.
- Users **x** and **y** listened to **at least three different songs on the same day**.

The recommendation should be **unidirectional**, meaning that if user **x** should be recommended to user **y**, the reverse should also be true. Also, the result should not contain duplicate recommendations.

The output should contain the following columns:
- **user_id**: The ID of the user receiving the recommendation.
- **recommended_id**: The ID of the recommended friend.

Return the result table in any order.

### Tables:

#### Listens Table:

| user_id | song_id | day        |
|--------|--------|------------|
| 7      | 25     | 2022-06-01 |
| 7      | 26     | 2022-06-01 |
| 7      | 27     | 2022-06-01 |
| 8      | 25     | 2022-06-01 |
| 8      | 26     | 2022-06-01 |
| 8      | 27     | 2022-06-01 |
| 9      | 25     | 2022-06-01 |
| 9      | 26     | 2022-06-01 |
| 9      | 27     | 2022-06-01 |
| 10     | 25     | 2022-06-01 |
| 10     | 26     | 2022-06-01 |
| 10     | 28     | 2022-06-01 |
| 11     | 25     | 2022-06-02 |
| 11     | 26     | 2022-06-02 |
| 11     | 27     | 2022-06-02 |

#### Friendship Table:

| user1_id | user2_id |
|---------|---------|
| 7       | 8       |

### Expected Output:

| user_id | recommended_id |
|--------|----------------|
| 7      | 9              |
| 8      | 9              |
| 9      | 7              |
| 9      | 8              |

### Problem Constraints:
- The **Friendship** table contains unique pairs of user IDs, where **user1_id < user2_id**.
- The **Listens** table may contain duplicate entries.
- The result should be returned in any order.

---

## Solution Approaches

### Approach 1: Using BDR, Recommended, and NotFriends CTEs

#### SQL Query:
```sql
WITH BDR AS (
    SELECT user1_id, user2_id FROM Friendship
    UNION ALL
    SELECT user2_id, user1_id FROM Friendship
),
Recommended AS (
    SELECT l1.user_id AS user1_id, l2.user_id AS user2_id, l1.song_id, l1.day
    FROM Listens l1
    JOIN Listens l2 ON l1.day = l2.day 
                   AND l1.song_id = l2.song_id 
                   AND l1.user_id != l2.user_id
),
NotFriends AS (
    SELECT r.user1_id, r.user2_id, r.song_id, r.day
    FROM Recommended r
    LEFT JOIN BDR b ON r.user1_id = b.user1_id AND r.user2_id = b.user2_id
    WHERE b.user1_id IS NULL
)
SELECT user1_id AS user_id, user2_id AS recommended_id
FROM NotFriends
GROUP BY user1_id, user2_id, day
HAVING COUNT(*) > 2
ORDER BY user1_id, user2_id;
```

The idea behind this approach is to use CTEs to efficiently find recommended friends:
- **Step 1:** Construct the **BDR** CTE to list all friends (bidirectionally).
- **Step 2:** Use the **Recommended** CTE to pair users who listened to the same song on the same day.
- **Step 3:** Use the **NotFriends** CTE to filter out those who are already friends.
- **Step 4:** Group the results by user and recommended friend, counting the number of common songs, and return the pairs with at least three matches.

---

## Performance Analysis

### Method 1 (BDR, Recommended, and NotFriends CTEs):
- **Time complexity:** O(N * M), where **N** is the number of listens and **M** is the number of friendships.
- **Space complexity:** O(N + M), due to the use of intermediate tables in CTEs.

---

## Conclusion
- **Method 1:** Effectively utilizes CTEs to break down the problem into manageable parts. Handles large data efficiently but may face performance challenges with extremely large datasets due to multiple joins.

### Final Conclusion:
- This approach is modular and easy to understand, making it the preferred choice for tackling this problem.
