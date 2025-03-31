# LeetCode Problem 1892: Page Recommendations II

## Problem Description
You are tasked with creating a page recommendation system for a social media website. The system should recommend a page to a user if at least one of their friends likes that page and the user has not already liked it.

The final output should contain three columns:
- **user_id**: The ID of the user receiving the recommendation.
- **page_id**: The ID of the page being recommended.
- **friends_likes**: The number of the user’s friends who liked the recommended page.

Return the result table in any order.

### Tables:

#### Friendship Table:

| user1_id | user2_id |
|---------|---------|
| 10      | 20      |
| 10      | 30      |
| 10      | 40      |
| 20      | 30      |
| 20      | 40      |
| 20      | 50      |
| 60      | 10      |

#### Likes Table:

| user_id | page_id |
|--------|--------|
| 10     | 99     |
| 20     | 28     |
| 30     | 29     |
| 40     | 58     |
| 50     | 17     |
| 60     | 37     |
| 20     | 77     |
| 30     | 77     |
| 60     | 99     |

### Expected Output:

| user_id | page_id | friends_likes |
|--------|--------|---------------|
| 10     | 77     | 2             |
| 10     | 28     | 1             |
| 10     | 29     | 1             |
| 10     | 58     | 1             |
| 10     | 37     | 1             |
| 20     | 29     | 1             |
| 20     | 58     | 1             |
| 20     | 17     | 1             |
| 20     | 99     | 1             |
| 30     | 99     | 1             |
| 30     | 28     | 1             |
| 40     | 99     | 1             |
| 40     | 77     | 1             |
| 40     | 28     | 1             |
| 50     | 77     | 1             |
| 50     | 28     | 1             |

### Problem Constraints:
- The **Friendship** table contains unique pairs of user IDs.
- The **Likes** table contains unique combinations of user and page IDs.
- The result should be returned in any order.

---

## Solution Approaches

### Approach 1: Using BDR and FriendLikes CTEs

The idea behind this approach is to first gather all possible friendships from the **Friendship** table by unifying both `user1_id` and `user2_id` into a single set of user-friend pairs. Next, we calculate how many friends liked each page using the **Likes** table. Finally, we filter out pages already liked by the user.

#### SQL Query:
```sql
WITH BDR AS (
    SELECT user1_id AS user_id, user2_id AS friend_id FROM Friendship
    UNION ALL
    SELECT user2_id AS user_id, user1_id AS friend_id FROM Friendship
),
FriendLikes AS (
    SELECT B.user_id, L.page_id, COUNT(*) AS friends_likes
    FROM BDR B
    JOIN Likes L ON B.friend_id = L.user_id
    GROUP BY B.user_id, L.page_id
)
SELECT F.user_id, F.page_id, F.friends_likes
FROM FriendLikes F LEFT JOIN Likes L ON F.user_id = L.user_id AND F.page_id = L.page_id
WHERE L.user_id IS NULL ORDER BY user_id, page_id
```

#### Explanation:
- **Step 1:** Use a Common Table Expression (CTE) `BDR` to create a unified list of friendships.
- **Step 2:** Another CTE, `FriendLikes`, counts the number of friends who liked a specific page.
- **Step 3:** Perform a left join to eliminate pages already liked by the user.
- **Step 4:** Return the recommended pages ordered by user ID and page ID.

---

## Performance Analysis

### Method 1 (BDR and FriendLikes CTEs):
- **Time complexity:** O(N * M), where N is the number of friendships and M is the number of likes. The join operations can be computationally intensive.
- **Space complexity:** O(N + M), as we store intermediate friendship and like data in CTEs.

---

## Conclusion
- **Method 1:** Efficiently handles the problem by breaking it down into smaller manageable subproblems using CTEs. However, it might struggle with large data due to multiple joins.

### Final Conclusion:
- This approach is recommended as it logically separates friendship calculations and like aggregations, making the query modular and maintainable.
