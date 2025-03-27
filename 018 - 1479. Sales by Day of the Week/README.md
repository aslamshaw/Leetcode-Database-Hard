# LeetCode Problem 1479: Sales by Day of the Week

## Problem Description

We are given two tables, `Orders` and `Items`. The goal is to generate a sales report that shows how many units of each item category have been ordered on each day of the week.

The result table should:
1. Display `category` as the first column.
2. Show the number of units sold on `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, and `Sunday` as separate columns.
3. Be ordered by `category`.

### Tables:

#### Orders Table:

| order_id | customer_id | order_date | item_id | quantity |
|----------|------------|------------|---------|----------|
| 10       | 2          | 2021-07-12 | 3       | 8        |
| 11       | 4          | 2021-07-13 | 5       | 6        |
| 12       | 1          | 2021-07-14 | 2       | 12       |
| 13       | 3          | 2021-07-15 | 1       | 4        |
| 14       | 5          | 2021-07-16 | 4       | 7        |
| 15       | 2          | 2021-07-17 | 6       | 3        |
| 16       | 1          | 2021-07-18 | 2       | 9        |
| 17       | 5          | 2021-07-19 | 3       | 5        |
| 18       | 3          | 2021-07-20 | 1       | 11       |

#### Items Table:

| item_id | item_name       | item_category |
|---------|---------------|---------------|
| 1       | ML Handbook    | Book         |
| 2       | AI Principles  | Book         |
| 3       | Ultra Phone    | Phone        |
| 4       | Quantum Glass  | Glasses      |
| 5       | Running Shoes  | Shoes        |
| 6       | Sports Watch   | Watch        |

### Expected Output:

| Category | Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday |
|----------|--------|---------|-----------|----------|--------|----------|--------|
| Book     | 5      | 0       | 12        | 0        | 0      | 0        | 9      |
| Glasses  | 0      | 0       | 0         | 0        | 7      | 0        | 0      |
| Phone    | 8      | 0       | 0         | 0        | 0      | 0        | 5      |
| Shoes    | 0      | 6       | 0         | 0        | 0      | 0        | 0      |
| Watch    | 0      | 0       | 0         | 0        | 0      | 3        | 0      |

### Problem Constraints:
- `order_date` must be mapped to a weekday name (`Monday`, `Tuesday`, etc.).
- Categories with no sales on a given day should display `0` for that day.
- The final result should be sorted by `category` alphabetically.

---

## Solution Approaches

### Approach 1: Using Aggregation and Pivoting

#### SQL Query:
```sql
WITH WeekOrders AS (SELECT item_category AS category, DAYNAME(order_date) AS day_name, quantity
                    FROM Items LEFT JOIN Orders USING(item_id))
SELECT category,
       SUM(CASE WHEN day_name = 'Monday' THEN quantity ELSE 0 END) AS Monday,
       SUM(CASE WHEN day_name = 'Tuesday' THEN quantity ELSE 0 END) AS Tuesday,
       SUM(CASE WHEN day_name = 'Wednesday' THEN quantity ELSE 0 END) AS Wednesday,
       SUM(CASE WHEN day_name = 'Thursday' THEN quantity ELSE 0 END) AS Thursday,
       SUM(CASE WHEN day_name = 'Friday' THEN quantity ELSE 0 END) AS Friday,
       SUM(CASE WHEN day_name = 'Saturday' THEN quantity ELSE 0 END) AS Saturday,
       SUM(CASE WHEN day_name = 'Sunday' THEN quantity ELSE 0 END) AS Sunday
FROM WeekOrders GROUP BY category ORDER BY category;
```

#### Explanation:
- **Extracting Day Names**: Convert `order_date` to weekday names to distribute sales into the correct columns.
- **Joining Tables**: Associate orders with categories.
- **Pivoting Data**: Aggregate total quantity sold for each category per day.
- **Sorting and Formatting**: Ensure the final output is sorted by `category`.

---

## Performance Analysis

### Method 1 (Using Aggregation and Pivoting):

- **Time complexity**: O(n) - Scans and aggregates all rows from `Orders` once.
- **Space complexity**: O(n) - Stores the temporary grouped results before output.

---

## Conclusion

- **Method 1**: Efficient and structured approach using aggregation and conditional summing.
- **Alternative approaches**: Could use `PIVOT()` if supported or `GROUP BY + conditional aggregation`, but the chosen approach remains clear and effective.

### Final Conclusion:
This approach is optimal given the problem constraints, efficiently calculating sales per category while ensuring correct ordering and handling null cases properly.
