### **Problem Name: Customers Who Bought All Products**

**Problem Link:** [LeetCode – 1045. Customers Who Bought All Products](https://leetcode.com/problems/customers-who-bought-all-products/)

---

### **1. Problem Understanding**

We are given two tables:

**Customer**

| Column      | Description                              |
| ----------- | ---------------------------------------- |
| customer_id | Identifier of the customer               |
| product_key | Identifier of the product they purchased |

(Usually `(customer_id, product_key)` is the primary key here.)

**Product**

| Column      | Description                    |
| ----------- | ------------------------------ |
| product_key | Identifier of the product (PK) |

**Goal:**
Find all `customer_id` values for customers who have bought **every product** listed in the `Product` table.

In other words, for each customer:

* Count how many **distinct products** they have purchased.
* Compare that count against the **total number of distinct products** in the `Product` table.
* If both counts match, that customer has bought all products.

---

### **2. The Core Logic: Per-Customer Product Count vs Total Product Count**

The solution hinges on comparing:

1. **Per-customer distinct product count** from `Customer`.
2. **Global distinct product count** from `Product`.

| Task                                         | SQL Operation                                          |
| -------------------------------------------- | ------------------------------------------------------ |
| Count how many products each customer bought | `COUNT(DISTINCT product_key)` grouped by `customer_id` |
| Count total number of distinct products      | `SELECT COUNT(DISTINCT product_key) FROM Product`      |
| Filter customers who bought all products     | `HAVING COUNT(DISTINCT product_key) = (subquery)`      |

Note that we do not actually need a `JOIN` here, because:

* `Customer` already links customers to products.
* `Product` is only needed to compute how many distinct products exist.

---

### **3. Incorrect Query and Why It Is Wrong**

Wrong idea (conceptual):

```sql
SELECT 
    customer_id
FROM 
    Customer 
HAVING   
    COUNT(DISTINCT product_key) = -- count of customer_id --
GROUP BY 
    customer_id;
```

**Issues:**

1. The `HAVING` clause is referencing a comment placeholder (`-- count of customer_id --`), not a real expression.
2. The comparison should be against the **total number of products**, not a per-customer count of `customer_id`.
3. The total number of products is not computed here at all. You need a subquery referencing the `Product` table.

The key mistake is misunderstanding what to compare:

* We need: per-customer `COUNT(DISTINCT product_key)`
  compared to
  global `COUNT(DISTINCT product_key)` from `Product`.

---

### **4. Correct Query**

```sql
SELECT 
    customer_id
FROM 
    Customer
GROUP BY 
    customer_id
HAVING 
    COUNT(DISTINCT product_key) = (
        SELECT COUNT(DISTINCT product_key)
        FROM Product
    );
```

---

### **5. Why This Query Works**

1. **`GROUP BY customer_id`**
   Aggregates all rows belonging to each customer.

2. **`COUNT(DISTINCT product_key)` in the outer query**
   For every `customer_id`, this gives the number of different products that customer has purchased.

3. **Subquery:**

   ```sql
   SELECT COUNT(DISTINCT product_key)
   FROM Product
   ```

   This returns the total number of distinct products available in the `Product` table.

4. **`HAVING` clause comparison:**

   ```sql
   HAVING COUNT(DISTINCT product_key) = (subquery)
   ```

   Keeps only those customers whose number of distinct purchased products equals the total number of products. These are exactly the customers who bought all products.

No join is necessary because:

* `Customer` already tells us which products each customer has bought.
* `Product` is only used to provide a single scalar value (total distinct product count).

---

### **6. Example Walkthrough**

**Product table:**

| product_key |
| ----------: |
|           1 |
|           2 |
|           3 |

**Customer table:**

| customer_id | product_key |
| ----------: | ----------: |
|           1 |           1 |
|           1 |           2 |
|           1 |           3 |
|           2 |           1 |
|           2 |           2 |
|           3 |           1 |
|           3 |           2 |
|           3 |           3 |

Total distinct products: `3`.

Per-customer distinct product counts:

| customer_id | COUNT(DISTINCT product_key) |
| ----------: | --------------------------- |
|           1 | 3                           |
|           2 | 2                           |
|           3 | 3                           |

Applying the `HAVING` condition:

* Customer 1: 3 = 3 → included
* Customer 2: 2 ≠ 3 → excluded
* Customer 3: 3 = 3 → included

**Final output:**

| customer_id |
| ----------: |
|           1 |
|           3 |

---

### **7. Key Takeaways**

* When a problem asks for “customers who bought all products,” think:
  **per-customer count of unique products vs total number of products**.
* Use `GROUP BY customer_id` and `COUNT(DISTINCT product_key)` on the `Customer` table.
* Use a scalar subquery to get `COUNT(DISTINCT product_key)` from `Product`.
* Use `HAVING` (not `WHERE`) to filter on aggregated values.
