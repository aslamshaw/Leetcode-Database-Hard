# LeetCode Problem 3214: Year on Year Growth Rate

## Problem Description

Given a transactional dataset, the objective is to compute the **year-over-year (YoY) growth rate** of total spending for each product. Each transaction contains a unique identifier, a product ID, the amount spent, and the transaction's date and time.

For each product, the total annual spend should be calculated per year, and then the growth rate compared to the previous year should be determined. If there is no data for the previous year, the growth rate should be reported as `NULL`. The output must be sorted by `product_id` and `year`.

### Tables:

#### user_transactions Table:

| transaction_id | product_id | spend  | transaction_date      |
|----------------|------------|--------|------------------------|
| 101            | 555001     | 980.40 | 2018-12-15 11:30:00    |
| 102            | 555001     | 1050.00| 2019-11-22 14:45:00    |
| 103            | 555001     | 1300.10| 2020-10-05 16:20:00    |
| 104            | 555001     | 1420.75| 2021-09-14 18:10:00    |

### Expected Output:

| year | product_id | curr_year_spend | prev_year_spend | yoy_rate |
|------|------------|------------------|------------------|----------|
| 2018 | 555001     | 980.40           | NULL             | NULL     |
| 2019 | 555001     | 1050.00          | 980.40           | 7.10     |
| 2020 | 555001     | 1300.10          | 1050.00          | 23.82    |
| 2021 | 555001     | 1420.75          | 1300.10          | 9.29     |

### Problem Constraints:
- Output should include only valid YoY rates where the previous year exists.
- Spend amounts are decimal numbers.
- The final result must be **sorted by `product_id` and `year` in ascending order**.
- Round `yoy_rate` to **two decimal places**.
- If previous year's spend is missing, `yoy_rate` should be `NULL`.

---

## Solution Approaches

### Approach 1: CTE with LAG Window Function

This method leverages **Common Table Expressions (CTEs)** and **window functions** to efficiently calculate the required values.

#### SQL Query:
```sql
WITH YearlySpend AS (
    SELECT 
        YEAR(transaction_date) AS year, 
        product_id, 
        SUM(spend) AS curr_year_spend 
    FROM user_transactions 
    GROUP BY YEAR(transaction_date), product_id),
    
PrevYearSpend AS (
    SELECT 
        year,
        product_id,
        curr_year_spend,
        LAG(year) OVER (PARTITION BY product_id ORDER BY year) AS prev_year,
        LAG(curr_year_spend) OVER (PARTITION BY product_id ORDER BY year) AS prev_year_spend
    FROM YearlySpend)
    
SELECT
    year,
    product_id,
    curr_year_spend,
    CASE 
        WHEN year = prev_year + 1 THEN prev_year_spend
        ELSE NULL
    END AS prev_year_spend,
    CASE 
        WHEN year = prev_year + 1 THEN ROUND((curr_year_spend - prev_year_spend) * 100.0 / prev_year_spend, 2)
        ELSE NULL
    END AS yoy_rate
FROM PrevYearSpend
ORDER BY product_id, year;
```

#### Explanation:
- **Step 1**: Extract the year from each transaction date and compute the total spend per product per year.
- **Step 2**: Use the `LAG()` function to fetch the spend from the previous year for each product.
- **Step 3**: Filter to calculate the year-on-year growth rate **only** when the previous year's data is directly available (i.e., the previous year is exactly one less than the current year).
- **Step 4**: Use a `CASE` statement to apply the YoY rate formula only where valid, and return `NULL` otherwise.

---

## Performance Analysis

### Method 1 (CTE with LAG Window Function):

- **Time complexity**: O(N log N) — due to sorting in the window function and aggregation by year and product.
- **Space complexity**: O(N) — to store intermediate CTE results and window calculations.

---

## Conclusion

- **Method 1**: This approach is clean and efficient. It separates concerns logically using CTEs, leverages window functions for historical comparisons, and ensures only relevant YoY values are calculated. The use of `LAG()` avoids self-joins, which boosts performance on large datasets.

### Final Conclusion:
- The CTE + `LAG()` method is the **preferred approach** due to its simplicity, clarity, and performance efficiency. It elegantly handles both aggregation and comparison within the same query structure, making it ideal for this problem.
