## **Problem Name: Customer Who Visited but Did Not Make Any Transactions**

**Problem Link:** [LeetCode](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/)

---

### **1. Problem Understanding**

We are given two tables:

* **`Visits`**: Contains every `visit_id` and the corresponding `customer_id` who made that visit.
* **`Transactions`**: Contains every `transaction_id` and the `visit_id` during which the transaction occurred.

**Goal:**
Find all customers who have visited the store but **did not make any transactions**.
For these customers, we must return their `customer_id` and the **count of visits** that did not result in any transaction.

This problem is about identifying what’s *missing* — the visits that have no matching records in the `Transactions` table.

---

### **2. Core Logic: `LEFT JOIN` + `IS NULL`**

The fundamental idea to find missing relationships between two tables is the combination of `LEFT JOIN` and `IS NULL`.

1. **`LEFT JOIN`** ensures we keep all rows from the `Visits` table, even if there are no matches in the `Transactions` table.
2. **`IS NULL`** identifies the visits for which no matching transaction exists (since unmatched columns will appear as `NULL` after the join).
3. **Filtering**: We apply a `WHERE` clause to retain only the non-transacting visits.

---

### **3. Step-by-Step Query Construction**

1. **Join the Tables:**

   ```sql
   FROM Visits v
   LEFT JOIN Transactions t
   ON v.visit_id = t.visit_id
   ```

   This ensures all visits are retained.

2. **Filter Missing Transactions:**

   ```sql
   WHERE t.transaction_id IS NULL
   ```

   Keeps only visits that did not lead to transactions.

3. **Group by Customer:**

   ```sql
   GROUP BY v.customer_id
   ```

   Groups non-transacting visits by each customer.

4. **Count the Non-Transacting Visits:**

   ```sql
   SELECT v.customer_id, COUNT(v.visit_id) AS count_no_trans
   ```

---

### **4. Dry Run Example**

Let’s walk through the official LeetCode example to understand the query execution.

#### **`Visits` Table (`v`)**

| visit_id | customer_id |
| -------- | ----------- |
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |

#### **`Transactions` Table (`t`)**

| transaction_id | visit_id |
| -------------- | -------- |
| 2              | 5        |
| 3              | 5        |
| 9              | 5        |
| 12             | 1        |
| 13             | 2        |

---

#### **Step 1: LEFT JOIN Result (Before Filtering)**

| v.visit_id | v.customer_id | t.transaction_id |
| ---------- | ------------- | ---------------- |
| 1          | 23            | 12               |
| 2          | 9             | 13               |
| 4          | 30            | NULL             |
| 5          | 54            | 2                |
| 5          | 54            | 3                |
| 5          | 54            | 9                |
| 6          | 96            | NULL             |
| 7          | 54            | NULL             |
| 8          | 54            | NULL             |

Here, `visit_id` values **4**, **6**, **7**, and **8** have no corresponding `transaction_id` (NULL).

---

#### **Step 2: After `WHERE t.transaction_id IS NULL`**

| v.visit_id | v.customer_id | t.transaction_id |
| ---------- | ------------- | ---------------- |
| 4          | 30            | NULL             |
| 6          | 96            | NULL             |
| 7          | 54            | NULL             |
| 8          | 54            | NULL             |

Only visits with no transactions remain.

---

#### **Step 3: After `GROUP BY v.customer_id`**

| customer_id | count_no_trans |
| ----------- | -------------- |
| 30          | 1              |
| 96          | 1              |
| 54          | 2              |

Explanation:

* Customer **30** → 1 visit without transaction (visit 4)
* Customer **96** → 1 visit without transaction (visit 6)
* Customer **54** → 2 visits without transactions (visits 7 and 8)

---

### **5. Final SQL Query**

```sql
SELECT
    v.customer_id,
    COUNT(v.visit_id) AS count_no_trans
FROM
    Visits AS v
LEFT JOIN
    Transactions AS t
ON
    v.visit_id = t.visit_id
WHERE
    t.transaction_id IS NULL
GROUP BY
    v.customer_id;
```

---

### **6. Key Takeaways**

* **Identifying Missing Data:**
  The `LEFT JOIN ... IS NULL` pattern is an effective method to detect rows in one table without matches in another.

* **Execution Order:**
  SQL processes queries in this order:
  `FROM/JOIN → WHERE → GROUP BY → SELECT`.
  Understanding this helps build logical query flow.

* **Counting by Category:**
  Using `GROUP BY` with `COUNT()` allows us to compute per-customer statistics, such as the number of visits without transactions.

---

