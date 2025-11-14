### **Problem Name: Product Sales Analysis III**

**Problem Link:** [LeetCode – 1070. Product Sales Analysis III](https://leetcode.com/problems/product-sales-analysis-iii/)

---

### **1. Problem Understanding**

We are given a table `Sales` with product sales data across years.

Typical schema:

| Column     | Type | Description                 |
| ---------- | ---- | --------------------------- |
| product_id | int  | Product identifier          |
| year       | int  | Year of the sale            |
| quantity   | int  | Units sold in that year     |
| price      | int  | Price per unit in that year |

**Goal:**
For each `product_id`, return the record corresponding to the **first year** that product appears in the sales table, along with its `quantity` and `price` for that first year.

So for every product:

1. Find `MIN(year)` for that `product_id`.
2. Return the row for that `(product_id, first_year)` with the correct `quantity` and `price`.

Output columns:

* `product_id`
* `first_year`
* `quantity`
* `price`

---

### **2. The Core Logic: “Earliest Year per Product” + Row Filtering**

Key idea:
We are not just aggregating per product; we need the **full row** (including `quantity`, `price`) for the earliest year.

1. **Find earliest year per product**

   * `SELECT product_id, MIN(year) FROM Sales GROUP BY product_id`
2. **Match those (product_id, year) pairs back to the main table**

   * Use a tuple `IN` condition:
     `(product_id, year) IN (SELECT product_id, MIN(year) ...)`
3. **Select columns**

   * From the matching rows, select `product_id`, `year AS first_year`, `quantity`, `price`.

This pattern is common when you want “row corresponding to min/max of something per group”.

---

### **3. Incorrect Query and Why It Fails**

**Wrong Query:**

```sql
SELECT 
    product_id,
    MIN(year) AS first_year,
    quantity,
    price
FROM 
    Sales
GROUP BY 
    product_id;
```

**Issues:**

1. **Invalid non-aggregated columns in GROUP BY context**

   * In strict SQL (and in most modern SQL modes), every selected column must either:

     * appear in `GROUP BY`, or
     * be aggregated (MIN, MAX, SUM, etc.).
   * Here, `quantity` and `price` are neither grouped nor aggregated.

2. **Logical ambiguity**

   * Even if MySQL allowed it in non-strict mode, which `quantity` and `price` should it show?
   * A product may have multiple rows over different years with different quantities and prices. There is no guarantee the `quantity` and `price` align with the `MIN(year)`.

So this query does not reliably return the correct row corresponding to the earliest year.

---

### **4. Correct Query and How It Works**

**Right Query:**

```sql
SELECT 
    product_id,
    year AS first_year,
    quantity,
    price
FROM Sales
WHERE (product_id, year) IN (
    SELECT product_id, MIN(year)
    FROM Sales
    GROUP BY product_id
);
```

**Why this is correct:**

1. **Inner subquery: earliest year per product**

   ```sql
   SELECT product_id, MIN(year)
   FROM Sales
   GROUP BY product_id
   ```

   * For each product, returns exactly one row:

     * `product_id`
     * `MIN(year)` (the first year that product appears)

2. **Outer query: filter rows to those first-year pairs**

   ```sql
   WHERE (product_id, year) IN (
       SELECT product_id, MIN(year)
       ...
   )
   ```

   * The tuple `(product_id, year)` from `Sales` must match one of the `(product_id, first_year)` pairs in the subquery.
   * This guarantees we only keep rows where `year` is the earliest year for that product.

3. **Selecting all needed columns**

   * Once filtered, every remaining row is “first year row” for that product.
   * We can safely select:

     * `product_id`
     * `year AS first_year`
     * `quantity`
     * `price`

No ambiguity in `quantity` and `price` remains: they are taken from the actual row of that earliest year.

---

### **5. Example Walkthrough**

Suppose `Sales` contains:

| product_id | year | quantity | price |
| ---------: | ---- | -------- | ----- |
|          1 | 2018 | 10       | 100   |
|          1 | 2019 | 20       | 120   |
|          2 | 2019 | 5        | 200   |
|          2 | 2020 | 15       | 220   |

**Step 1: Subquery (earliest year per product)**

```sql
SELECT product_id, MIN(year)
FROM Sales
GROUP BY product_id;
```

Result:

| product_id | MIN(year) |
| ---------: | --------- |
|          1 | 2018      |
|          2 | 2019      |

**Step 2: Filter main table using `(product_id, year) IN (...)`**

Rows kept:

* (1, 2018, 10, 100) because (1, 2018) is in the subquery.
* (2, 2019, 5, 200) because (2, 2019) is in the subquery.

**Final Output:**

| product_id | first_year | quantity | price |
| ---------: | ---------- | -------- | ----- |
|          1 | 2018       | 10       | 100   |
|          2 | 2019       | 5        | 200   |

---

### **6. Alternative Approach (Using Window Functions, MySQL 8+)**

If window functions are allowed, you can also solve this with `ROW_NUMBER()`:

```sql
WITH ranked_sales AS (
    SELECT
        product_id,
        year,
        quantity,
        price,
        ROW_NUMBER() OVER (
            PARTITION BY product_id
            ORDER BY year
        ) AS rn
    FROM Sales
)
SELECT
    product_id,
    year AS first_year,
    quantity,
    price
FROM ranked_sales
WHERE rn = 1;
```

* `ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY year)` assigns rank 1 to the earliest year row per product.
* Filtering `WHERE rn = 1` selects exactly the first-year row.

---

### **7. Key Takeaways**

* When you need the **row corresponding to MIN/MAX per group**, simple `GROUP BY` with non-aggregated columns is not enough.
* Use:

  * A **subquery** with `(product_id, min_value)` and filter via tuple `IN`, or
  * A **window function** (`ROW_NUMBER()`), where supported.
* Always ensure that `quantity` and `price` (or similar columns) are taken from the correct row, not arbitrarily picked from grouped data.
