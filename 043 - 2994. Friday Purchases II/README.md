# LeetCode Problem 2994: Friday Purchases II

## Problem Description

You are given a table named `Purchases` that records user purchases made throughout November 2023. Each record contains the user ID, the date of purchase, and the amount spent.

The goal is to calculate the total spending on **each Friday** of the month. If no purchases were made on a particular Friday, it should still appear in the result with a total of `0`.

The final output should list each Friday along with:
- The **week of the month** it belongs to (1 to 5),
- The **purchase date** (i.e., the Friday),
- The **total amount spent** on that day (or 0 if none).

The output should be **ordered by the week of the month in ascending order**.

---

## Example

### Input: `Purchases` Table

| user_id | purchase_date | amount_spend |
|---------|----------------|---------------|
| 11      | 2023-11-07     | 1126          |
| 15      | 2023-11-30     | 7473          |
| 17      | 2023-11-14     | 2414          |
| 12      | 2023-11-24     | 9692          |
| 8       | 2023-11-03     | 5117          |
| 1       | 2023-11-16     | 5241          |
| 10      | 2023-11-12     | 8266          |
| 13      | 2023-11-24     | 12000         |

### Output:

| week_of_month | purchase_date | total_amount |
|---------------|----------------|---------------|
| 1             | 2023-11-03     | 5117          |
| 2             | 2023-11-10     | 0             |
| 3             | 2023-11-17     | 0             |
| 4             | 2023-11-24     | 21692         |

---

## Week of the Month Calculation

To determine which week a date falls into:
- Use the **day of the month** and divide by `7`, then apply the ceiling function to group days into weekly buckets.
  - Days 1–7 → Week 1
  - Days 8–14 → Week 2
  - Days 15–21 → Week 3
  - Days 22–28 → Week 4
  - Days 29–31 → Week 5 (if applicable)

For example:
- `CEIL(15 / 7)` = `3` → Day 15 falls in the **3rd week**.

---

## Solution Approach

### Approach 1: Generate Fridays with a Recursive CTE

#### SQL Query:
```sql
WITH RECURSIVE November AS (
  SELECT '2023-11-01' AS purchase_date 
  UNION ALL 
  SELECT purchase_date + INTERVAL 1 DAY FROM November WHERE purchase_date < '2023-11-30'
)
SELECT 
  CEIL(DAY(purchase_date)/7) AS week_of_month, 
  purchase_date, 
  IFNULL(SUM(amount_spend), 0) AS total_amount 
FROM 
  November 
  LEFT JOIN Purchases USING(purchase_date) 
WHERE 
  DAYOFWEEK(purchase_date) = 6 
GROUP BY 
  purchase_date;
```

1. Generate every date from **2023-11-01** to **2023-11-30** using a recursive date series.
2. From this set of dates, filter out only the **Fridays** (based on weekday logic).
   - In MySQL: `DAYOFWEEK(date) = 6` (where Sunday = 1, Monday = 2, ..., Saturday = 7).
3. Perform a **LEFT JOIN** with the `Purchases` table on `purchase_date` so that Fridays with no purchases will still be present in the result.
4. Group by `purchase_date` and calculate the `SUM(amount_spend)`, using `IFNULL` to convert null totals to `0`.
5. Add a computed column `week_of_month` by using the formula `CEIL(DAY(purchase_date) / 7)`.

---

## Performance Analysis

- **Time complexity**: O(30), since the date range is fixed to November 2023.
- **Space complexity**: O(1), as the dataset is restricted to a specific month and few days.

---

## Conclusion

This problem highlights:
- Handling missing data via **LEFT JOIN**,
- Grouping dates into weeks using simple **mathematical bucketing**,
- Dynamic computation over a fixed time window using **recursive CTEs** and **date functions**.

The final result includes every Friday in the month, whether or not purchases occurred — a common pattern in reporting and analytics tasks.
