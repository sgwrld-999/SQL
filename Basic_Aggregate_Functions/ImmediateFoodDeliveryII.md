### **Problem Name: Immediate First-Delivery Rate**

**Problem Link:** [LeetCode – Immediate Food Delivery II](https://leetcode.com/problems/immediate-food-delivery-ii/)

---

### **1. Problem Understanding**

We have a table `Delivery` that records customers’ orders and their preferred delivery dates.
Relevant columns:

* `delivery_id` (unique row id)
* `customer_id`
* `order_date`
* `customer_pref_delivery_date`

**Task:**
Compute the **percentage of customers whose very first order was “immediate”**, where *immediate* means:

```
order_date = customer_pref_delivery_date
```

Tie-breaking for “first order”:

* First, pick the **earliest order_date** per `customer_id`.
* If multiple rows share that earliest date, pick the **smallest delivery_id** among them.

**Output:**

* A single value named `immediate_percentage`, rounded to **two decimals**.

---

### **2. The Core Logic: “First Order Isolation” + Ratio via AVG**

1. **Isolate each customer’s first order**

   * Prefer window functions: `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date, delivery_id)` and keep only `rn = 1`.
   * Without windows: join to subqueries that (a) get the earliest date per customer and (b) among ties on that date, choose the smallest `delivery_id`.

2. **Compute the share of immediate first orders**

   * For the first-order rows only, evaluate
     `order_date = customer_pref_delivery_date`.
   * Convert to numeric (1/0) and **take the average × 100**, then **ROUND(..., 2)**.

---

### **3. Constructing the Query (Step-by-Step)**

1. **Mark first orders deterministically**

   ```sql
   ROW_NUMBER() OVER (
     PARTITION BY customer_id
     ORDER BY order_date, delivery_id
   ) AS rn
   ```
2. **Filter to `rn = 1`** to keep exactly one row per customer.
3. **Compute percentage**

   ```sql
   AVG(CASE WHEN order_date = customer_pref_delivery_date THEN 1.0 ELSE 0 END) * 100
   ```
4. **Round to two decimals**

   ```sql
   ROUND(<avg_expression>, 2) AS immediate_percentage
   ```

---

### **4. Dry Run Example**

**Sample `Delivery` rows (conceptual):**

| delivery_id | customer_id | order_date | customer_pref_delivery_date |
| ----------- | ----------- | ---------- | --------------------------- |
| 10          | 1           | 2024-01-05 | 2024-01-05                  |
| 12          | 1           | 2024-03-01 | 2024-03-02                  |
| 21          | 2           | 2024-01-05 | 2024-01-07                  |
| 22          | 2           | 2024-01-05 | 2024-01-05                  |
| 35          | 3           | 2024-02-10 | 2024-02-10                  |

**Determine first orders:**

* Customer 1: earliest date = `2024-01-05` → `delivery_id=10` → immediate (yes).
* Customer 2: earliest date = `2024-01-05` → tie between `delivery_id=21` and `22` → pick `delivery_id=21` → immediate (no).
* Customer 3: only one row → immediate (yes).

**Computation:**
Immediate first orders = 2 (cust 1, cust 3)
Total first orders = 3
Percentage = `(2 / 3) * 100 = 66.666...` → `66.67`.

---

### **5. SQL Query (Window Function – MySQL 8+, Preferred)**

```sql
WITH first_orders AS (
  SELECT
    customer_id,
    order_date,
    customer_pref_delivery_date,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY order_date, delivery_id
    ) AS rn
  FROM Delivery
)
SELECT
  ROUND(
    AVG(CASE WHEN order_date = customer_pref_delivery_date THEN 1.0 ELSE 0 END) * 100
  , 2) AS immediate_percentage
FROM first_orders
WHERE rn = 1;
```

**Why this works**
`ROW_NUMBER` chooses exactly one earliest row per customer (ties resolved by `delivery_id`). `AVG(CASE ...) * 100` gives the proportion, and `ROUND` formats it.

---

### **6. SQL Query (No Window Functions – Aggregation Joins)**

```sql
SELECT
  ROUND(
    AVG(d.order_date = d.customer_pref_delivery_date) * 100
  , 2) AS immediate_percentage
FROM Delivery AS d
JOIN (
  -- Earliest order_date per customer
  SELECT customer_id, MIN(order_date) AS first_order_date
  FROM Delivery
  GROUP BY customer_id
) f
  ON d.customer_id = f.customer_id
 AND d.order_date  = f.first_order_date
JOIN (
  -- Among rows on the earliest date, pick the smallest delivery_id
  SELECT
    x.customer_id,
    MIN(x.delivery_id) AS first_delivery_id
  FROM Delivery x
  JOIN (
    SELECT customer_id, MIN(order_date) AS first_order_date
    FROM Delivery
    GROUP BY customer_id
  ) y
    ON x.customer_id = y.customer_id
   AND x.order_date  = y.first_order_date
  GROUP BY x.customer_id
) g
  ON d.customer_id = g.customer_id
 AND d.delivery_id = g.first_delivery_id;
```

**Why this works**
The first subquery fixes the earliest date; the second resolves ties by the smallest `delivery_id`. The final `AVG(condition) * 100` computes the percentage.

---

### **7. SQL Query (Tuple Filter Style)**

This style filters rows to first orders using a tuple predicate and computes the percentage using `SUM(IF(...))` over a `COUNT(DISTINCT customer_id)` denominator.

**Important:** the basic `(customer_id, order_date) IN (...)` form alone does **not** break ties when multiple rows share the earliest date. Below is the **corrected** version that also enforces the smallest `delivery_id` on that earliest date.

```sql
SELECT
  ROUND(
    SUM(IF(order_date = customer_pref_delivery_date, 1, 0)) * 100
    / COUNT(DISTINCT customer_id)
  , 2) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date, delivery_id) IN (
  SELECT d1.customer_id, d1.order_date, d1.delivery_id
  FROM Delivery d1
  JOIN (
    -- earliest date per customer
    SELECT customer_id, MIN(order_date) AS first_order_date
    FROM Delivery
    GROUP BY customer_id
  ) f
    ON d1.customer_id = f.customer_id
   AND d1.order_date  = f.first_order_date
  WHERE d1.delivery_id = (
    -- tie-break: smallest delivery_id among rows on earliest date
    SELECT MIN(d2.delivery_id)
    FROM Delivery d2
    WHERE d2.customer_id = d1.customer_id
      AND d2.order_date  = d1.order_date
  )
);
```

**Why this works**

* The `WHERE (customer_id, order_date, delivery_id) IN (...)` filter restricts the outer query to exactly one first-order row per customer.
* `SUM(IF(...))` counts immediate first orders; `COUNT(DISTINCT customer_id)` counts total first orders.
* `ROUND(..., 2)` formats the result.

**If you intentionally ignore tie-breaking** (not recommended), the simplified version looks like this (your screenshot’s idea), but it can overcount first orders when ties exist:

```sql
-- Simplified, lacks deterministic tie-break on same earliest date
SELECT
  ROUND(
    SUM(IF(order_date = customer_pref_delivery_date, 1, 0)) * 100
    / COUNT(DISTINCT customer_id)
  , 2) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN (
  SELECT customer_id, MIN(order_date)
  FROM Delivery
  GROUP BY customer_id
);
```

---

### **8. Key Takeaways**

* **Define “first” precisely**: earliest `order_date`, then break ties with the smallest `delivery_id`.
* **Window functions** (`ROW_NUMBER`) are the clearest way to pick one row per group; use them when available.
* **Boolean-to-numeric aggregation** (`AVG(condition)` or `SUM(IF(...))/COUNT(...)`) is an effective way to compute ratios.
* **Tuple filters** can be concise, but must also include the tie-break field to be correct.
* **Rounding** with `ROUND(..., 2)` ensures the required output format.
