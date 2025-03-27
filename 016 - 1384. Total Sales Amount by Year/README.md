# LeetCode Problem 1384: Total Sales Amount by Year

## Problem Description

Given two tables, `Product` and `Sales`, you need to calculate the total sales amount of each product for each year between 2018 and 2020. Each product has sales periods with an associated `average_daily_sales` value. The goal is to report the total sales amount for each item, with the corresponding `product_id`, `product_name`, and `report_year`.

The `Sales` table contains periods with `start_date` and `end_date`, both inclusive, and the average daily sales during that period. The output should display the total sales amount for each product for each year.

### Tables:

#### Product Table:

| product_id | product_name |
|------------|--------------|
| 101        | Smart Watch  |
| 102        | Wireless Earbuds |
| 103        | Bluetooth Speaker |

#### Sales Table:

| product_id | period_start | period_end  | average_daily_sales |
|------------|--------------|-------------|---------------------|
| 101        | 2019-03-01   | 2019-03-15  | 150                 |
| 102        | 2018-11-20   | 2019-01-15  | 50                  |
| 103        | 2020-02-01   | 2020-02-10  | 30                  |

### Expected Output:

| product_id | product_name    | report_year | total_amount |
|------------|-----------------|-------------|--------------|
| 101        | Smart Watch     | 2019        | 2250         |
| 102        | Wireless Earbuds | 2018        | 1500         |
| 102        | Wireless Earbuds | 2019        | 750          |
| 103        | Bluetooth Speaker | 2020        | 300          |

- The total sales for `Smart Watch` during 2019 is calculated as `150 (average_daily_sales) * 15 (days) = 2250`.
- `Wireless Earbuds` has sales spanning from 2018 to 2020, resulting in total sales for each year: 2018 has 31 days and 2019 has 15 days.
- `Bluetooth Speaker` was sold in February 2020, with 10 days of sales, resulting in a total of `30 * 10 = 300`.

### Problem Constraints:
- Sales records for the years between 2018 and 2020.
- `product_id` is a unique identifier for each product.
- Each sales period is represented with a `start_date` and `end_date`, which are inclusive.

---

## Solution Approaches

### Approach 1: Generate Report Years Based on Sales Data

In this approach, all the possible years between the earliest `start_date` and latest `end_date` are generated. Then, for each year, the relevant sales period is calculated by determining the start and end dates for each year, and the total sales are computed.

#### SQL Query:
```sql
WITH RECURSIVE YearRange AS (SELECT YEAR(MIN(period_start)) AS report_year FROM Sales 
UNION ALL SELECT report_year + 1 FROM YearRange WHERE report_year < (SELECT YEAR(MAX(period_end)) FROM Sales)),
PreReport AS (SELECT *, CONCAT(report_year, '-01-01') AS year_start, CONCAT(report_year, '-12-31') AS year_end
              FROM Sales CROSS JOIN YearRange 
              WHERE YEAR(period_start) <= report_year AND report_year <= YEAR(period_end))
SELECT product_id, product_name, report_year,
(DATEDIFF(LEAST(period_end, year_end), GREATEST(period_start, year_start)) + 1) * average_daily_sales 
AS total_amount FROM PreReport JOIN Product USING(product_id) ORDER BY product_id;
```

#### Explanation:
- The approach generates all the years that fall within the sales period, ensuring that the start and end dates for each year are correctly calculated.
- The total sales for each year are calculated by multiplying the number of days within the sales period for that year by the `average_daily_sales`.

---

## Performance Analysis

### Method 1 (Generate Report Years Based on Sales Data):
- **Time complexity**: `O(N * Y)`, where `N` is the number of sales records and `Y` is the number of years in the sales period.
- **Space complexity**: `O(N + Y)`, as both sales data and generated years are stored.

---

## Conclusion

- **Method 1**: This approach efficiently generates the total sales amount by breaking down the sales data into individual years and calculating the total sales for each year.
  - **Strengths**: Handles all the edge cases and accurately computes total sales by year.
  - **Weaknesses**: The approach could be inefficient if the sales data is large, as it generates multiple rows for each report year.

### Final Conclusion:
- The first approach is optimal for generating sales reports by year, but may require optimizations in cases where there is a large dataset with multiple overlapping periods. It is clear, systematic, and ensures the correctness of calculations.
