# LeetCode Problem 3554: Find Category Recommendation Pairs

## Problem Description

The goal is to analyze customer purchases to identify pairs of product categories that are commonly bought together. For each pair of categories `(category1, category2)` where `category1` comes before `category2` alphabetically, we need to count the number of unique users who have purchased at least one product from both categories. Only pairs with at least three shared customers should be reported. The final results are sorted first by the number of customers in descending order, then alphabetically by `category1` and `category2` for ties.

### Tables:

#### ProductPurchases Table:

| user_id | product_id | quantity |
|---------|------------|----------|
| 10      | 111        | 2        |
| 10      | 112        | 1        |
| 11      | 111        | 3        |
| 12      | 113        | 2        |
| 13      | 112        | 4        |
| 14      | 114        | 1        |
| 15      | 111        | 2        |
| 15      | 113        | 1        |
| 16      | 114        | 3        |
| 17      | 112        | 2        |

#### ProductInfo Table:

| product_id | category    | price |
|------------|-------------|-------|
| 111        | Gadgets     | 150   |
| 112        | Books       | 25    |
| 113        | Apparel     | 50    |
| 114        | Kitchen     | 80    |

### Expected Output:

| category1 | category2 | customer_count |
|-----------|-----------|----------------|
| Books     | Apparel   | 3              |
| Books     | Gadgets   | 3              |
| Apparel   | Gadgets   | 3              |

### Problem Constraints:
- Each row in `ProductPurchases` represents a single purchase by a user for a product.
- `user_id` and `product_id` combinations are unique in `ProductPurchases`.
- `product_id` is unique in `ProductInfo`.
- Only consider category pairs with at least 3 unique customers.
- Sorting should respect customer count first, then category names lexicographically.

---

## Solution Approaches

### Approach 1: Using Self-Join on User-Category Mapping

This approach first constructs a mapping of users to distinct categories they purchased from, and then performs a self-join on this mapping to count the number of users sharing each category pair.

#### SQL Query:
```sql
WITH UserCategory AS (
    SELECT DISTINCT *
    FROM (SELECT user_id, category FROM ProductPurchases JOIN ProductInfo USING(product_id)) s)

SELECT u1.category AS category1, u2.category AS category2, COUNT(*) AS customer_count
FROM UserCategory u1
JOIN UserCategory u2 ON u1.category < u2.category AND u1.user_id = u2.user_id
GROUP BY category1, category2
HAVING customer_count >= 3
ORDER BY customer_count DESC, category1, category2;
```

#### Explanation:
- Without `DISTINCT`, if a user buys multiple products in the same category, the inner join will produce multiple rows for the same `(user_id, category)`.
- When you later do the self-join to find category pairs, these duplicate rows will artificially inflate the customer_count.
  For example, if user 1 buys two books, the join might count them twice for Books-Apparel, which is wrong.
- Create a `UserCategory` dataset containing each user and the categories they have purchased from.
- Perform a self-join on `UserCategory` where the first category is lexicographically smaller than the second and the `user_id` matches.
- Count the occurrences to get `customer_count` for each category pair.
- Filter to only include pairs with at least three customers.
- Sort the result as per the requirements.

---

## Performance Analysis

### Method 1 (Self-Join on User-Category Mapping):

- **Time complexity**: O(U * C^2) where `U` is the number of users and `C` is the average number of categories per user.
- **Space complexity**: O(U * C) for storing the user-category mapping.

---

## Conclusion

- **Self-Join on User-Category Mapping**: Efficient for moderate-sized datasets and directly gives the required counts. It is simple to understand and implement, though performance may degrade for extremely large numbers of users with many category purchases.

### Final Conclusion:
- This approach is sufficient for solving the problem effectively, providing correct results with minimal complexity in logic. It balances clarity and efficiency well.
