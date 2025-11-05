### **Problem Name: Confirmation Rate**

**Problem Link:** [LeetCode](https://leetcode.com/problems/confirmation-rate/)

---

### **1. Problem Understanding**

We have two tables: `Signups` (a list of all users) and `Confirmations` (a log of all confirmation attempts, which can be either 'confirmed' or 'timeout').

**Task:**
Compute the confirmation rate for **every user** in the `Signups` table.

* **Formula:** `Rate = (Number of 'confirmed' actions) / (Total number of actions)`
* **Edge Case:** If a user never appears in `Confirmations`, their confirmation rate is `0`.
* **Formatting Requirement:** Output must be rounded to two decimal places.

---

### **2. The Core Logic: `LEFT JOIN` + Conditional Aggregation**

The correct approach is a combination of:

1. **`LEFT JOIN`**

   * Ensures every user in `Signups` appears in the result.
   * Users with no confirmation attempts will still appear, with `NULL` values from `Confirmations`.

2. **Conditional Aggregation**

   * Count `confirmed` actions for the numerator.
   * Count all non-null actions for the denominator.

This strategy guarantees inclusion of all users and accurate confirmation rate calculations.

---

### **3. Constructing the Query (Step-by-Step)**

1. **Base Join**

   ```sql
   FROM Signups s LEFT JOIN Confirmations c ON s.user_id = c.user_id
   ```

   * Ensures all users from `Signups` are included.

2. **Group by User**

   ```sql
   GROUP BY s.user_id
   ```

   * We must compute rate per user.

3. **Numerator**

   ```sql
   SUM(c.action = 'confirmed')
   ```

   * MySQL evaluates the condition as `1` (true) or `0` (false).
   * Summing yields total confirmed actions.

4. **Denominator**

   ```sql
   COUNT(c.action)
   ```

   * Counts only non-null values.
   * Users with no confirmation entries get a denominator of `0`.

5. **Handling Division by Zero**

   ```sql
   IFNULL(...)
   ```

   * If a user has no attempts, result becomes `0`.

6. **Rounding**

   ```sql
   ROUND(..., 2)
   ```

   * Ensures two-decimal formatting.

---

### **4. Dry Run Example**

**Intermediate `LEFT JOIN` output**

| s.user_id | c.user_id |  c.action |
| --------: | --------: | --------: |
|         3 |         3 |   timeout |
|         3 |         3 |   timeout |
|         7 |         7 | confirmed |
|         7 |         7 | confirmed |
|         7 |         7 | confirmed |
|         2 |         2 | confirmed |
|         2 |         2 |   timeout |
|         6 |      NULL |      NULL |

**Grouped Calculations**

| User | Confirmed Count | Total Actions | Rate Calculation     | Final Rate |
| ---- | --------------- | ------------- | -------------------- | ---------: |
| 6    | 0               | 0             | NULL / 0 = NULL -> 0 |       0.00 |
| 3    | 0               | 2             | 0 / 2 = 0            |       0.00 |
| 7    | 3               | 3             | 3 / 3 = 1            |       1.00 |
| 2    | 1               | 2             | 1 / 2 = 0.5          |       0.50 |

---

### **5. SQL Query**

```sql
SELECT
    s.user_id,
    ROUND(
        IFNULL(
            SUM(c.action = 'confirmed') / COUNT(c.action),
            0
        ),
        2
    ) AS confirmation_rate
FROM
    Signups s
LEFT JOIN
    Confirmations c ON s.user_id = c.user_id
GROUP BY
    s.user_id;
```

---

### **6. Key Takeaways**

* **LEFT JOIN** preserves users with no confirmation attempts.
* **SUM(condition)** is an effective MySQL technique to count only matching values.
* **COUNT(column)** ignores `NULL`, making it ideal when working with `LEFT JOIN`.
* **IFNULL** avoids division by zero and outputs correct value for users with no actions.
* **ROUND** ensures format requirements are met.

---

