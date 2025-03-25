# LeetCode Problem 1159: Market Analysis II

## Problem Description

Given two tables, `Users` and `Orders`, along with an `Items` table, the task is to determine for each user whether the brand of their second sold item matches their favorite brand. If a user has never sold an item, they should be marked as "no".

### Tables:

#### Users Table:

| user_id | join_date  | favorite_brand |
|---------|------------|----------------|
| 10      | 2018-12-10 | Apple          |
| 20      | 2019-03-15 | Dell           |
| 30      | 2019-06-20 | Sony           |
| 40      | 2019-09-30 | Asus           |

#### Orders Table:

| order_id | order_date | item_id | buyer_id | seller_id |
|----------|------------|---------|----------|-----------|
| 101      | 2019-07-10 | 7       | 10       | 20        |
| 102      | 2019-08-11 | 3       | 10       | 30        |
| 103      | 2019-08-12 | 5       | 20       | 30        |
| 104      | 2019-09-01 | 2       | 40       | 20        |
| 105      | 2019-09-02 | 1       | 30       | 40        |
| 106      | 2019-09-05 | 3       | 20       | 40        |

#### Items Table:

| item_id | item_brand |
|---------|------------|
| 1       | HP         |
| 2       | Apple      |
| 3       | Sony       |
| 5       | Dell       |
| 7       | Asus       |

### Expected Output:

| seller_id | 2nd_item_fav_brand |
|-----------|--------------------|
| 10        | no                 |
| 20        | yes                |
| 30        | yes                |
| 40        | no                 |

### Problem Constraints:
- Each `user_id` is unique in the `Users` table.
- Each `order_id` is unique in the `Orders` table.
- Each `item_id` is unique in the `Items` table.
- If a user has never sold an item, they should be marked as "no".

---

## Solution Approaches

### Approach 1: Ranking Orders with Window Functions

This approach uses window functions to rank orders based on their sale date for each seller. We then check if the brand of the second sold item matches the seller's favorite brand.

#### SQL Query:
```sql
WITH RankedOrders AS (SELECT u.user_id AS seller_id, order_id, order_date, 
                      ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date) AS rk, item_brand, favorite_brand
                      FROM Users u LEFT JOIN Orders o ON u.user_id = o.seller_id LEFT JOIN Items USING(item_id))
SELECT seller_id, CASE WHEN item_brand = favorite_brand THEN 'yes' ELSE 'no' END AS 2nd_item_fav_brand
FROM RankedOrders WHERE order_id IS NULL OR rk = 2;
```

#### Explanation:
- We perform a **LEFT JOIN** between `Users`, `Orders`, and `Items` tables to ensure all users are considered.
- We use **`ROW_NUMBER()`** to rank the sold items per seller by `order_date`.
- We filter out:
  - Users who never sold an item.
  - Users where the ranked order is exactly 2.
- If a user’s **second sold item's brand** matches their **favorite brand**, we return "yes"; otherwise, we return "no".

---

## Performance Analysis

### Method 1 (Ranking Orders with Window Functions):

- **Time complexity**: `O(N log N)`, as we use window functions that require sorting within partitions.
- **Space complexity**: `O(N)`, as we store ranked orders in an intermediate result set.

---

## Conclusion

- **Method 1**: Efficient approach leveraging SQL window functions. It ensures that all users are considered while effectively identifying second sales.

### Final Conclusion:
- This method is optimal for handling large datasets, as `ROW_NUMBER()` allows efficient ranking and filtering of the required orders.
