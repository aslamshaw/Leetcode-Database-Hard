# LeetCode Problem 3052: Maximize Items

## Problem Description

You are given an inventory of items, each with a unique ID, type (either `prime_eligible` or `not_prime`), category, and its respective square footage. The goal is to maximize the number of items that can be stored in a warehouse with a capacity of **500,000 square feet**.

The warehouse prioritizes stocking **prime_eligible** items first, as complete sets, followed by using any remaining space to store **not_prime** items—also in complete sets. A *set* consists of **all items** of a given type in the inventory.

You need to calculate how many total individual **items** (not sets) of each type can be stored, respecting the priority and the space constraint.

### Tables:

#### Inventory Table:

| item_id | item_type      | item_category | square_footage |
|---------|----------------|---------------|----------------|
| 1011    | prime_eligible | Kitchen       | 40.00          |
| 2034    | not_prime      | Electronics   | 60.25          |
| 3055    | prime_eligible | Sports        | 110.75         |
| 4021    | not_prime      | Books         | 33.50          |
| 5093    | not_prime      | Music         | 28.00          |
| 6077    | prime_eligible | Jewelry       | 75.30          |
| 7084    | not_prime      | Office        | 18.95          |
| 8023    | prime_eligible | Gaming        | 290.80         |
| 9121    | prime_eligible | Toys          | 65.10          |
| 1001    | prime_eligible | Tools         | 28.00          |

### Expected Output:

| item_type      | item_count |
|----------------|------------|
| prime_eligible | 5400       |
| not_prime      | 8          |

### Problem Constraints:
- Each *set* includes all available items for a given type (`prime_eligible` or `not_prime`).
- Sets are added to the warehouse as a whole; partial sets are not allowed.
- You must **first** fill in as many `prime_eligible` sets as possible before allocating space to `not_prime` sets.
- Results must be ordered in descending order by `item_count`.

---

## Solution Approaches

### Approach 1: CTE-Based Set Calculation

This approach uses **Common Table Expressions (CTEs)** to organize the process of computing:
1. The total square footage per type (`prime_eligible`, `not_prime`).
2. The size of each set and the number of sets that can fit.
3. The remaining space after accommodating prime sets, and how many not_prime sets can then be stored.

#### SQL Query:
```sql
    WITH InventorySummary AS (SELECT item_type, COUNT(*) AS set_size, SUM(square_footage) AS set_area 
                              FROM Inventory GROUP BY item_type),
    PrimeEligible AS (SELECT item_type, FLOOR(500000 / set_area) AS prime_set_count, set_area AS prime_set_area,
                      set_size * FLOOR(500000 / set_area) AS item_count
                      FROM InventorySummary WHERE item_type = 'prime_eligible'),
    RemainingArea AS (SELECT 500000 - (prime_set_count * prime_set_area) AS remaining_area FROM PrimeEligible)
    SELECT item_type, item_count FROM PrimeEligible
    UNION ALL
    SELECT item_type, set_size * FLOOR((SELECT remaining_area FROM RemainingArea) / set_area)
    FROM InventorySummary WHERE item_type = 'not_prime' ORDER BY item_count DESC;
```

#### Explanation:
- **Step 1**: Compute the number of items and total area per item type.
- **Step 2**: Calculate how many full sets of `prime_eligible` items fit into 500,000 sq ft.
- **Step 3**: Calculate the remaining area, and determine how many full sets of `not_prime` items can fit.
- **Step 4**: Multiply the number of sets by the number of items in a set to get total item count for each type.
- Results are combined using `UNION ALL` and sorted by `item_count` in descending order.

### Approach 2: Variable-Based Procedural Calculation

This method uses SQL variables to perform step-by-step calculations:
1. Storing square footage and item count for both types.
2. Determining maximum number of sets that can be stored for each type.
3. Computing remaining space and how many `not_prime` sets can fit.
4. Returning total item counts accordingly.

#### SQL Query:
```sql
SET @prime_set_size = (SELECT SUM(square_footage) FROM Inventory WHERE item_type = 'prime_eligible');
SET @prime_set_count = (SELECT COUNT(*) FROM Inventory WHERE item_type = 'prime_eligible');

SET @not_prime_set_size = (SELECT SUM(square_footage) FROM Inventory WHERE item_type = 'not_prime');
SET @not_prime_set_count = (SELECT COUNT(*) FROM Inventory WHERE item_type = 'not_prime');

SET @prime_count = FLOOR(500000 / @prime_set_size);
SET @required_size = @prime_set_size * @prime_count;

SET @remaining_size = 500000 - @required_size;
SET @not_prime_count = FLOOR(@remaining_size / @not_prime_set_size);

SELECT 'prime_eligible' AS item_type, FLOOR(@prime_count * @prime_set_count) AS item_count
UNION
SELECT 'not_prime', FLOOR(@not_prime_count * @not_prime_set_count)
ORDER BY item_count DESC;
```

#### Explanation:
- **Step 1**: Store total square footage and count for each type into SQL variables.
- **Step 2**: Calculate maximum number of `prime_eligible` sets that fit.
- **Step 3**: Determine the space used and what remains.
- **Step 4**: Using remaining space, calculate how many `not_prime` sets fit.
- **Step 5**: Multiply set counts by the item count per set to get final item counts for both types.
- Final result is ordered by total number of items per type, descending.

---

## Performance Analysis

### Method 1 (CTE-Based Set Calculation):

- **Time complexity**: O(N) — Each item is scanned once per aggregation or computation step.
- **Space complexity**: O(1) — Fixed number of intermediate results and groupings.

### Method 2 (Variable-Based Procedural Calculation):

- **Time complexity**: O(N) — Each variable assignment queries the whole table once.
- **Space complexity**: O(1) — Only a few variables are used.

---

## Conclusion

- **Method 1**: Clear and modular via CTEs; easily readable and scalable. Ideal for declarative SQL systems or complex logic branching.
- **Method 2**: Efficient for procedural SQL environments with support for session variables. Useful when step-by-step clarity is required.

### Final Conclusion:
- Both approaches produce the correct result, but **CTE-based Approach (Method 1)** is more readable and easier to maintain. It's the recommended solution unless the SQL dialect lacks CTE support.
