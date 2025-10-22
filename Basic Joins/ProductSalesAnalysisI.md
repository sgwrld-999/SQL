### **Problem Name: Product Sales Analysis I**

**Problem Link:** [LeetCode](https://leetcode.com/problems/product-sales-analysis-i/)

-----

### **1. Problem Understanding** 

We are given two tables:

  * `Sales`: Contains transaction data, including which `product_id` was sold, in what `year`, and for what `price`.
  * `Product`: A lookup table that maps a `product_id` to its `product_name`.

**Task:**
For each sale recorded in the `Sales` table, we need to report the product's name along with the year and price of the sale. This requires fetching data from both tables simultaneously.

-----

### **2. The Core Logic: The `INNER JOIN`**

The key to solving this problem is to combine the two tables. A **`JOIN`** clause is used in SQL to combine rows from two or more tables based on a related column between them.

An **`INNER JOIN`** specifically selects all rows from both participating tables as long as there is a match between the columns in the `ON` clause. In this case, it finds pairs of rows where `Sales.product_id` is the same as `Product.product_id`.

For each row in the `Sales` table, the query looks up the corresponding `product_name` in the `Product` table and creates a single combined row for the output. Since every sale should correspond to a real product, an `INNER JOIN` is the perfect tool.

-----

### **3. Constructing the Query (Step-by-Step)**

1.  **`SELECT p.product_name, s.year, s.price`**: We list the final columns we want. We use table aliases (`p` for Product, `s` for Sales) to clearly specify where each column comes from. This is a best practice for readability.
2.  **`FROM Sales s`**: We start with the `Sales` table, which contains most of the data we need (`year` and `price`).
3.  **`INNER JOIN Product p`**: We specify that we want to join `Sales` with the `Product` table.
4.  **`ON s.product_id = p.product_id`**: This is the crucial join condition. It tells the database to match a row from `Sales` with a row from `Product` only if their `product_id` values are identical.

-----

### **4. Dry Run Example**

Let's trace the join with sample data.

**`Sales` (Table `s`):**
| sale\_id | product\_id | year | price |
| :--- | :--- | :--- | :--- |
| 1 | 100 | 2008 | 5000 |
| 2 | 100 | 2009 | 5000 |
| 7 | 200 | 2011 | 15000 |

**`Product` (Table `p`):**
| product\_id | product\_name |
| :--- | :--- |
| 100 | Nokia |
| 200 | Apple |

**Join Process on `s.product_id = p.product_id`:**

1.  **Sale 1 (`product_id=100`):** Finds a match in `Product` where `product_id=100`. The result combines columns to produce `('Nokia', 2008, 5000)`.
2.  **Sale 2 (`product_id=100`):** Finds the same match in `Product`. The result is `('Nokia', 2009, 5000)`.
3.  **Sale 7 (`product_id=200`):** Finds a match in `Product` where `product_id=200`. The result is `('Apple', 2011, 15000)`.

-----

### **5. SQL Query**

```sql
SELECT
    p.product_name,
    s.year,
    s.price
FROM
    Sales s
INNER JOIN
    Product p ON s.product_id = p.product_id;
```

-----

### **6. Key Takeaways** 

  * **`JOIN` is for Combining Data**: The primary purpose of a `JOIN` is to create a result set that pulls columns from multiple tables.
  * **The `ON` Clause is Key**: The `ON` clause is the most important part of a `JOIN`, defining the logical link between the tables.
  * **`INNER JOIN` for Matches**: Use `INNER JOIN` when you only want to see records that have a direct correspondence in both tables.


-----
### **7. Different Approach**

The key difference between the two queries is how they handle sales for products that might not exist in the `Product` table. The `INNER JOIN` will only show sales that have a matching product, while the `LEFT JOIN` will show **all** sales, regardless of whether a matching product is found.

-----

#### **`INNER JOIN` vs. `LEFT JOIN`**

##### **INNER JOIN (Your First Query)**

An **`INNER JOIN`** returns only the records that have matching values in both tables. Think of it as finding the intersection of two sets.

  * **Purpose:** To select rows where the join condition (`s.product_id = p.product_id`) is met in **both** the `Sales` and `Product` tables.
  * **Outcome:** If a sale exists in the `Sales` table with a `product_id` that does not exist in the `Product` table (perhaps due to a data entry error), that sales record will be **completely excluded** from your results.
  * **Use Case:** Ideal when you only want to analyze a clean, complete dataset where every sale is guaranteed to be linked to a known product.

<!-- end list -->

```sql
SELECT
    p.product_name,
    s.year,
    s.price
FROM
    Sales s
INNER JOIN -- Only keeps records that match in BOTH tables.
    Product p ON s.product_id = p.product_id;
```

-----

##### **LEFT JOIN (Your Second Query)**

A **`LEFT JOIN`** returns **all** records from the left table (`Sales`), and the matched records from the right table (`Product`). If there is no match, the result is `NULL` on the right side.

  * **Purpose:** To select **all** rows from the `Sales` table and supplement them with data from the `Product` table where a match is found.
  * **Outcome:** You will see every single sales record. If a `product_id` from a sale does not exist in the `Product` table, the `product_name` for that row will be `NULL`, but the `year` and `price` will still be shown.
  * **Use Case:** Perfect for when you want to see all sales data, and also for identifying data integrity problems (like sales of non-existent products).

<!-- end list -->

```sql
SELECT
    p.product_name,
    s.year,
    s.price
FROM
    Sales s
LEFT JOIN -- Keeps ALL records from the left table (Sales).
    Product p ON s.product_id = p.product_id;
```

-----

#### **Summary**

| Feature | `INNER JOIN` | `LEFT JOIN` (from `Sales`) |
| :--- | :--- | :--- |
| **Rows from `Sales`** | Only returns sales with a valid product. | Returns **all** sales. |
| **Non-Matching Sales**| Excludes them entirely. | Includes them, but with a `NULL` `product_name`.|
| **Primary Use** | Analyzing clean, complete data. | Finding all sales, including potential data issues. |