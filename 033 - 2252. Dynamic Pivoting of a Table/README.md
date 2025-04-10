# LeetCode Problem 2252: Dynamic Pivoting of a Table

## Problem Description

You're given a dataset of product prices across multiple stores. Each row represents the price of a product in a specific store. The goal is to transform this table into a pivoted format where each row corresponds to a unique product and each column represents a store. If a product is not sold in a store, the corresponding price should be `null`.

### Tables:

#### Products Table:

| product_id | store     | price |
|------------|-----------|-------|
| 101        | MegaMart  | 250   |
| 101        | ValueHub  | 245   |
| 102        | Shopperz  | 300   |
| 102        | eStore    | 280   |
| 103        | MegaMart  | 310   |
| 103        | eStore    | 295   |

### Expected Output:

| product_id | MegaMart | Shopperz | ValueHub | eStore |
|------------|----------|----------|----------|--------|
| 101        | 250      | null     | 245      | null   |
| 102        | null     | 300      | null     | 280    |
| 103        | 310      | null     | null     | 295    |

### Problem Constraints:
- Each combination of product and store is unique.
- There can be up to 30 distinct stores.
- Output column order must follow lexicographical sorting of store names.
- If a product isn't sold in a store, its price should be `null` in that column.

---

## Solution Approaches

### Approach 1: Dynamic SQL Pivot with Conditional Aggregation

This solution creates a pivoted table structure dynamically based on the unique store names. The transformation involves creating a column for each store where the cell values are the prices for corresponding products. Conditional logic ensures that only relevant prices appear under the correct store columns.

#### SQL Query:
```sql
DELIMITER //

CREATE PROCEDURE pp()
BEGIN
	SELECT GROUP_CONCAT(
		DISTINCT
		CONCAT(' MAX(CASE WHEN store = \'', store, '\' THEN price ELSE NULL END) AS ', store)
		SEPARATOR ', ')
	INTO @sql
	FROM products;

	SET @sql = CONCAT('SELECT product_id, ', @sql, ' FROM products GROUP BY product_id');

	PREPARE stmt FROM @sql;
	EXECUTE stmt;
	DEALLOCATE PREPARE stmt;
END //

DELIMITER ;

CALL pp();
DROP PROCEDURE pp;
```

#### Explanation:
- First, the list of distinct store names is extracted and ordered alphabetically.
- For each store, a conditional expression is constructed to extract the price where applicable and `null` otherwise.
- These expressions are combined to form a pivoted structure.
- Finally, the data is grouped by `product_id`, ensuring one row per product with price values in the respective store columns.

---

## Performance Analysis

### Method 1 (Dynamic SQL Pivot with Conditional Aggregation):

- **Time complexity**: O(N × S), where N is the number of product entries and S is the number of unique stores.
- **Space complexity**: O(S), for handling the dynamic generation of store-specific columns.

---

## Conclusion

- **Method 1**: This approach is highly adaptable and efficient for situations where the number of stores is dynamic or unknown in advance. It scales well with the number of stores and cleanly transforms row-based data into a pivoted format.

### Final Conclusion:
- Dynamic SQL pivoting is a powerful technique for reformatting data in SQL when column headers are not fixed and must be determined at runtime. This method solves the problem with precision and flexibility.
