# LeetCode Problem 2253: Dynamic Unpivoting of a Table

## Problem Description

You are given a table where each row represents a product and its price across multiple stores. The prices are stored in separate columns named after each store. Your task is to transform (unpivot) this wide-format table into a long-format table where each row contains a `product_id`, a `store` name, and the corresponding `price`.

Only include rows where the product is actually sold (i.e., `price` is not null). The number of store columns can vary between test cases, with a minimum of 1 and a maximum of 30 store columns.

### Tables:

#### Products Table:

| product_id | AlphaMart | MegaDeals | Shopster | UrbanBuy |
|------------|-----------|-----------|----------|----------|
| 1001       | 99        | null      | 89       | null     |
| 1002       | null      | 120       | null     | 115      |
| 1003       | null      | null      | 300      | 320      |

### Expected Output:

| product_id | store     | price |
|------------|-----------|-------|
| 1001       | AlphaMart | 99    |
| 1001       | Shopster  | 89    |
| 1002       | MegaDeals | 120   |
| 1002       | UrbanBuy  | 115   |
| 1003       | Shopster  | 300   |
| 1003       | UrbanBuy  | 320   |

### Problem Constraints:
- The table will always have one column for `product_id`, and 1 to 30 columns for store names.
- Column names for stores are dynamic and may differ in each test case.
- Do not include rows where the price is null (product not available at that store).

---

## Solution Approaches

### Approach 1: Dynamic SQL Unpivot using UNION ALL

This approach constructs a dynamic SQL query that unpivots the table by generating a `SELECT` statement for each store column. Each of these statements extracts the `product_id`, store name, and price where the price is not null. These individual queries are combined using `UNION ALL` to produce the final unpivoted table.

#### SQL Query:
```sql
DELIMITER //
CREATE PROCEDURE mp()
BEGIN
	SELECT GROUP_CONCAT(
		CONCAT('SELECT product_id, \'', column_name, '\' AS store, ', column_name, ' AS price FROM products_piv WHERE ', column_name, ' IS NOT NULL')
		SEPARATOR ' UNION ALL ') 
	INTO @sql
	FROM information_schema.columns 
	WHERE table_name = 'products_piv' AND column_name != 'product_id';

	SET @sql = CONCAT('WITH cte AS (', @sql, ') SELECT * FROM cte ORDER BY product_id;');

	PREPARE stmt FROM @sql;
	EXECUTE stmt;
	DEALLOCATE PREPARE stmt;
END //
DELIMITER ;

CALL mp();
DROP PROCEDURE mp;
```

#### Explanation:
- The list of store columns is retrieved dynamically from metadata (e.g., `information_schema.columns`).
- For each column (excluding `product_id`), a `SELECT` clause is created to extract the data in long format.
- Only rows where the store’s price is not null are included.
- All such queries are concatenated using `UNION ALL` to form a complete unpivoted result.
- The final output lists all valid (product_id, store, price) combinations, optionally ordered by `product_id`.

---

## Performance Analysis

### Method 1 (Dynamic SQL Unpivot using UNION ALL):

- **Time complexity**: O(R × C), where R is the number of product rows and C is the number of store columns.
- **Space complexity**: O(C), required to dynamically build the SQL query for all store columns.

---

## Conclusion

- **Method 1**: This method is highly effective for situations where the schema is not fixed. It dynamically adapts to different store column names, and efficiently transforms the data into a normalized format.

### Final Conclusion:
- Dynamic unpivoting via SQL string construction and execution provides a scalable and reusable solution for varying input schemas. It is ideal for ETL and reporting scenarios requiring denormalized views.
