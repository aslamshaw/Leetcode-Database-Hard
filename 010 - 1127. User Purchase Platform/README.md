# LeetCode Problem 1127: User Purchase Platform

## Problem Description

You are given a table `Spending` that logs the spending history of users who made purchases on an online shopping platform. The platform records both mobile and desktop purchases by users. For each date, you need to calculate the total number of users and the total amount spent using **mobile only**, **desktop only**, and **both platforms** together.

Write an SQL query to find the total number of users and the total amount spent using mobile only, desktop only, and both platforms for each spending date.

### Tables:

#### Spending Table:

| user_id | spend_date | platform | amount |
|---------|------------|----------|--------|
| 1       | 2019-07-01 | mobile   | 100    |
| 1       | 2019-07-01 | desktop  | 100    |
| 2       | 2019-07-01 | mobile   | 100    |
| 2       | 2019-07-02 | mobile   | 100    |
| 3       | 2019-07-01 | desktop  | 100    |
| 3       | 2019-07-02 | desktop  | 100    |

#### Expected Output:

| spend_date | platform | total_amount | total_users |
|------------|----------|--------------|-------------|
| 2019-07-01 | desktop  | 100          | 1           |
| 2019-07-01 | mobile   | 100          | 1           |
| 2019-07-01 | both     | 200          | 1           |
| 2019-07-02 | desktop  | 100          | 1           |
| 2019-07-02 | mobile   | 100          | 1           |
| 2019-07-02 | both     | 0            | 0           |

### Problem Constraints:
- The `user_id`, `spend_date`, and `platform` columns together form the primary key of the table.
- The platform column contains only two values: 'mobile' and 'desktop'.
- You must calculate the total number of users and total spending for each platform per day, and ensure that users who use both platforms on the same day are counted as "both" platforms.

---

## Solution Approach

### Method 1: Handle Users Using Both Platforms with `CASE`

To solve this problem, we need to replace the `user_id` for those days where they used both 'mobile' and 'desktop' platforms with the value 'both'. This will group two rows for the same user into a single entry for the 'both' platform, and we will sum their amounts accordingly.

#### SQL Query:
```sql
WITH DatePlatform AS (SELECT * FROM (SELECT DISTINCT spend_date FROM Spending) s1
                      CROSS JOIN (SELECT 'mobile' AS platform UNION SELECT 'desktop' UNION SELECT 'both') s2),
Combined AS (SELECT user_id, spend_date, SUM(amount) AS amount,
             CASE WHEN COUNT(platform) = 1 THEN MAX(platform) ELSE 'both' END AS platform
             FROM Spending GROUP BY user_id, spend_date)
SELECT spend_date, platform, IFNULL(SUM(amount), 0) AS total_amount, COUNT(user_id) AS total_users
FROM DatePlatform LEFT JOIN Combined USING (spend_date, platform) GROUP BY spend_date, platform;
```

#### Explanation:
1. **Identify the user’s platform usage**: For each user on each spending date, we check if they used both the mobile and desktop platforms on the same day.
2. **Replace with ‘both’ if both platforms are used**: If a user uses both platforms, we will classify their spending under the 'both' platform.
3. **Group by spend date and platform**: After categorizing the platforms, we group the results by `spend_date` and `platform` and compute the sum of amounts and the number of distinct users.
4. **Aggregate**: If only one platform is used on a given day, we calculate the total amount and number of users for that specific platform. If both are used, we aggregate the data for the 'both' platform.

---

## Performance Analysis

### Method 1 (Using CASE to Handle 'both' Platform):

- **Time complexity**: The time complexity is O(n), where n is the number of rows in the `Spending` table. This is because we are processing each row and performing aggregations based on `user_id` and `spend_date`.
- **Space complexity**: The space complexity is O(n), where n is the number of rows in the `Spending` table, as we need to store the intermediate results for processing.

---

## Conclusion

- **Method 1**: This method efficiently handles users who use both mobile and desktop by grouping them together under a "both" platform. The solution is straightforward, but it requires proper handling of aggregation and grouping to ensure correct results. This method should be preferred for its clarity and efficiency when handling this type of problem.

### Final Conclusion:
- The provided approach is optimal for solving the problem where we need to aggregate spending across different platforms and handle cases where users use multiple platforms on the same day. The method is efficient and scales well with the data size.
