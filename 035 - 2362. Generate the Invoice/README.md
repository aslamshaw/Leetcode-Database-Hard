# LeetCode Problem 2362: Generate the Invoice

## Problem Description

You are given two tables: `Products` and `Purchases`. The `Products` table contains the `product_id` and `price` for each product. The `Purchases` table contains the `invoice_id`, `product_id`, and `quantity` for each purchase made in an invoice. 

Your task is to write an SQL query to show the details of the invoice with the highest total price. If two or more invoices have the same total price, return the details of the invoice with the smallest `invoice_id`.

### Tables:

#### Products Table:

| product_id | price |
|------------|-------|
| 1          | 120   |
| 2          | 250   |

#### Purchases Table:

| invoice_id | product_id | quantity |
|------------|------------|----------|
| 1          | 1          | 3        |
| 2          | 2          | 2        |
| 3          | 1          | 5        |
| 4          | 2          | 1        |
| 5          | 1          | 7        |

### Expected Output:

| product_id | quantity | price |
|------------|----------|-------|
| 1          | 7        | 840   |
| 2          | 2        | 500   |

### Problem Constraints:
- `product_id` is the primary key in the `Products` table.
- `(invoice_id, product_id)` is the primary key in the `Purchases` table.
- Each row in the `Products` table represents one product, and each row in the `Purchases` table represents an ordered quantity of a product in a specific invoice.

---

## Solution Approaches

### Approach 1: Transforming and Filtering MAX Price Sum and MIN Invoice ID

In this approach, we first calculate the total price for each invoice by multiplying the quantity of each product by its price. Then, we calculate the total price sum per invoice using a window function (`SUM() OVER()`). We filter out invoices that don't have the highest price and select the one with the smallest `invoice_id` among them.

#### SQL Query:
```sql
WITH ProductInvoiceDetails AS (SELECT invoice_id, product_id, quantity, (quantity * price) AS price, 
                               SUM(quantity * price) OVER (PARTITION BY invoice_id) AS price_sum
                               FROM Products JOIN Purchases USING(product_id) ORDER BY invoice_id, product_id),
MaxInvoicePrice AS (SELECT * FROM ProductInvoiceDetails 
                    WHERE price_sum = (SELECT MAX(price_sum) FROM ProductInvoiceDetails))
SELECT product_id, quantity, price 
FROM MaxInvoicePrice WHERE invoice_id = (SELECT MIN(invoice_id) FROM MaxInvoicePrice);
```

#### Explanation:
- We create a common table expression (CTE) `ProductInvoiceDetails` that computes the `price` for each product in each invoice and the `price_sum` for each invoice using a window function.
- In the second CTE `MaxInvoicePrice`, we filter the rows where the `price_sum` is equal to the maximum price sum.
- Finally, we select the rows from `MaxInvoicePrice` where the `invoice_id` is the smallest among those with the maximum price sum.

### Approach 2: Grouping, Ordering, and Limiting to Handle Ties

In this approach, we group by `invoice_id` to calculate the total price for each invoice. Then we order by the aggregated price in descending order and, in case of ties, order by `invoice_id` in ascending order. We limit the results to only the highest total price and join the result back with the details of the invoice.

#### SQL Query:
```sql
WITH Invoices AS (
  SELECT invoice_id, product_id, quantity, (quantity * price) AS price
  FROM Purchases JOIN Products USING (product_id)
),
InvoiceId AS (
  SELECT invoice_id, SUM(price) AS agg_price
  FROM Invoices
  GROUP BY invoice_id
  ORDER BY agg_price DESC, invoice_id
  LIMIT 1
)
SELECT product_id, quantity, price
FROM Invoices
JOIN InvoiceId USING (invoice_id);
```

#### Explanation:
- The first CTE `Invoices` calculates the `price` for each product in each invoice.
- The second CTE `InvoiceId` groups the invoices by `invoice_id`, calculates the total price (`agg_price`), and orders them by `agg_price` (descending) and `invoice_id` (ascending).
- We use `LIMIT 1` to select the invoice with the highest total price and the smallest `invoice_id` in case of ties.
- Finally, we join the `Invoices` CTE with `InvoiceId` to get the final result.

---

## Performance Analysis

### Method 1 (Transforming and Filtering MAX Price Sum and MIN Invoice ID):

- **Time complexity**: `O(n log n)` due to the sorting and window function operations over `n` rows in the `Products` and `Purchases` tables.
- **Space complexity**: `O(n)` for storing intermediate results in the CTEs.

### Method 2 (Grouping, Ordering, and Limiting to Handle Ties):

- **Time complexity**: `O(n log n)` due to the sorting and grouping operations.
- **Space complexity**: `O(n)` for storing intermediate results in the CTEs.

---

## Conclusion

- **Method 1**: This approach leverages window functions to calculate the total price per invoice and efficiently filters the maximum price sum with minimal complexity. It is straightforward but involves creating multiple intermediate CTEs.
- **Method 2**: This approach is more focused on grouping and handling ties by ordering the results, which might be more intuitive but requires an additional join operation.

### Final Conclusion:
- Both methods are efficient with a time complexity of `O(n log n)` and will likely perform similarly in practice. The choice between the two methods depends on the preference for either window functions or grouping-based queries. Method 1 may be preferable when window functions are familiar, while Method 2 is straightforward and might be easier to reason about.
