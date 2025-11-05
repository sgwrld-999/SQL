

### **Problem Name: Monthly Transaction Summary**

**Problem Link:** [LeetCode](https://leetcode.com/problems/monthly-transactions-i/)
(or equivalent SQL challenge on monthly transaction reporting)

---

### **1. Problem Understanding**

We are given a table named `Transactions`, which records all financial transactions.
Each record contains details such as transaction ID, country, state (approved or declined), amount, and date.

| id | country | state    | amount | trans_date |
| -- | ------- | -------- | ------ | ---------- |
| 1  | USA     | approved | 1200   | 2022-01-10 |
| 2  | India   | declined | 800    | 2022-01-12 |

**Task:**
We must calculate, **for each month and country**, the following statistics:

* The **month** (derived from `trans_date`)
* The **country**
* The **number of total transactions**
* The **total transaction amount**
* The **number of approved transactions**
* The **total approved transaction amount**

---

### **2. The Core Logic: `GROUP BY` + Conditional Aggregation**

The problem is a **data aggregation** task that combines grouping and conditional logic.

1. **Grouping**

   * The phrase *“for each month and country”* indicates that we must group our results by **month** and **country**.

2. **Aggregation**

   * We must compute:

     * total counts of transactions
     * total sums of transaction amounts
     * counts and sums **only for approved transactions**
   * This is achieved using a combination of `COUNT()`, `SUM()`, and conditional `CASE WHEN` statements inside those functions.

Together, these allow us to summarize transactional data efficiently.

---

### **3. Extracting the Month**

The `trans_date` column includes the full date, but we only need the **month** portion (formatted as `'YYYY-MM'`).

In MySQL, this can be done using:

```sql
DATE_FORMAT(trans_date, '%Y-%m') AS month
```

For example:

| trans_date | month   |
| ---------- | ------- |
| 2022-01-10 | 2022-01 |
| 2022-02-15 | 2022-02 |

This gives us a clean monthly grouping key.

---

### **4. Building the Aggregations**

We now define each metric step by step.

| Metric Description                    | SQL Expression                                      | Explanation                                        |
| ------------------------------------- | --------------------------------------------------- | -------------------------------------------------- |
| Total number of transactions          | `COUNT(id)`                                         | Counts all transactions for that month and country |
| Total transaction amount              | `SUM(amount)`                                       | Adds up all amounts in that group                  |
| Number of approved transactions       | `COUNT(CASE WHEN state = 'approved' THEN 1 END)`    | Counts rows where state = 'approved'               |
| Total amount of approved transactions | `SUM(CASE WHEN state = 'approved' THEN amount END)` | Adds the amounts only for approved transactions    |

This technique — using **CASE WHEN inside aggregation** — is called **conditional aggregation**.

---

### **5. Constructing the Full Query**

Combining the grouping, formatting, and aggregations gives:

```sql
SELECT 
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(id) AS trans_count,
    SUM(amount) AS trans_total_amount,
    COUNT(CASE WHEN state = 'approved' THEN 1 END) AS approved_trans,
    SUM(CASE WHEN state = 'approved' THEN amount END) AS approved_total_amount
FROM 
    Transactions
GROUP BY 
    month, country
ORDER BY
    month, country;
```

**Explanation:**

* `DATE_FORMAT()` extracts the month.
* `GROUP BY month, country` ensures the results are summarized by both dimensions.
* `COUNT()` and `SUM()` perform the aggregations.
* `CASE WHEN` handles condition-based computations.
* `ORDER BY` sorts the results in chronological and geographical order.

---

### **6. Dry Run Example**

Assume the following data:

| id | country | state    | amount | trans_date |
| -- | ------- | -------- | ------ | ---------- |
| 1  | USA     | approved | 1200   | 2022-01-10 |
| 2  | USA     | declined | 400    | 2022-01-18 |
| 3  | USA     | approved | 600    | 2022-01-28 |
| 4  | India   | approved | 800    | 2022-01-12 |
| 5  | India   | declined | 300    | 2022-01-20 |

**Step 1: Extract Month**

All transactions fall under `'2022-01'`.

**Step 2: Group by Month and Country**

| month   | country | Total Transactions | Total Amount | Approved Count | Approved Amount |
| ------- | ------- | ------------------ | ------------ | -------------- | --------------- |
| 2022-01 | USA     | 3                  | 2200         | 2              | 1800            |
| 2022-01 | India   | 2                  | 1100         | 1              | 800             |

**Step 3: Final Output**

| month   | country | trans_count | trans_total_amount | approved_trans | approved_total_amount |
| ------- | ------- | ----------- | ------------------ | -------------- | --------------------- |
| 2022-01 | India   | 2           | 1100               | 1              | 800                   |
| 2022-01 | USA     | 3           | 2200               | 2              | 1800                  |

---

### **7. Key Takeaways**

* **GROUP BY** defines the dimensions (month, country).
* **Aggregate functions** summarize data (COUNT, SUM).
* **Conditional aggregation** enables selective computation within groups.
* **DATE_FORMAT()** simplifies temporal grouping.
* **Clean formatting** and explicit aliasing improve readability and maintainability.
* **Order your output** (`ORDER BY`) to ensure structured, interpretable results.

---


