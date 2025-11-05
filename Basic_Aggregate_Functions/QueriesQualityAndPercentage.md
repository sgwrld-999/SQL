# **Problem Name: Queries Quality and Percentage**

**(LeetCode #1211)**

---

## **1. Problem Understanding** 📊

We are given one table — `Queries` — that tracks the results of different queries along with their ratings and positions.

---

### **Table: `Queries`**

| Column     | Type    | Meaning                          |
| :--------- | :------ | :------------------------------- |
| query_name | varchar | The name of the query            |
| result     | varchar | `'passed'` or `'failed'`         |
| position   | int     | The position of the query result |
| rating     | int     | Rating score from 1 to 5         |

---

### **Task**

For each `query_name`, calculate:

1. **quality** = average of `(rating / position)`
   → `ROUND(SUM(rating / position) / COUNT(*), 2)`

2. **poor_query_percentage** = percentage of queries with `rating < 3`
   → `ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100 / COUNT(*), 2)`

---

## **2. The Core Logic: Aggregation and Conditional Counting**

The problem has two key aggregation operations:

1. **Average Calculation (Quality)**

   * For each query, compute the mean of all `(rating / position)` values.
   * Use `SUM()` to add up all `(rating / position)` results, then divide by the total count of rows.

2. **Conditional Counting (Poor Query Percentage)**

   * Count how many rows have `rating < 3`.
   * Divide that count by total rows for the same `query_name`.
   * Multiply by 100 and round to two decimal places.

3. **Grouping**

   * Use `GROUP BY query_name` to perform the above calculations per query.

---

## **3. Constructing the Query (Step-by-Step)**

1. **Start from the `Queries` table**
   All the required data is in a single table — no joins needed.

2. **Compute `quality`**

   ```sql
   ROUND(SUM(rating / position) / COUNT(*), 2)
   ```

3. **Compute `poor_query_percentage`**
   Use conditional counting:

   ```sql
   ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100 / COUNT(*), 2)
   ```

4. **Group and Select**

   ```sql
   GROUP BY query_name
   ```

---

## **4. Dry Run Example**

### **`Queries` Table**

| query_name | result | position | rating |
| :--------- | :----- | :------- | :----- |
| Dog        | passed | 1        | 5      |
| Dog        | passed | 2        | 5      |
| Dog        | failed | 3        | 2      |
| Cat        | passed | 1        | 3      |
| Cat        | failed | 2        | 2      |

---

### **Step 1: Group by `query_name`**

| query_name | ratings/positions   | poor_ratings             |
| :--------- | :------------------ | :----------------------- |
| Dog        | (5/1), (5/2), (2/3) | rating = 2 (<3) → 1 poor |
| Cat        | (3/1), (2/2)        | rating = 2 (<3) → 1 poor |

---

### **Step 2: Compute for each group**

**For `Dog`:**

* `SUM(rating / position)` = (5/1 + 5/2 + 2/3) = 8.67
* `COUNT(*)` = 3
* **Quality** = 8.67 / 3 = 2.89
* **Poor Count** = 1
* **Poor Query %** = (1 / 3) × 100 = 33.33

**For `Cat`:**

* `SUM(rating / position)` = (3/1 + 2/2) = 4.00
* `COUNT(*)` = 2
* **Quality** = 4 / 2 = 2.00
* **Poor Count** = 1
* **Poor Query %** = (1 / 2) × 100 = 50.00

---

### **Step 3: Final Output**

| query_name | quality | poor_query_percentage |
| :--------- | :------ | :-------------------- |
| Cat        | 2.00    | 50.00                 |
| Dog        | 2.89    | 33.33                 |

---

## **5. SQL Query**

```sql
SELECT
    query_name,
    ROUND(SUM(rating / position) / COUNT(*), 2) AS quality,
    ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100 / COUNT(*), 2) AS poor_query_percentage
FROM 
    Queries
GROUP BY 
    query_name;
```

---

## **6. Understanding Key Components**

| Expression                               | Purpose                                                 |
| :--------------------------------------- | :------------------------------------------------------ |
| `SUM(rating / position) / COUNT(*)`      | Calculates average `(rating / position)` per query      |
| `CASE WHEN rating < 3 THEN 1 ELSE 0 END` | Converts a condition into numeric (1 or 0) for counting |
| `SUM(...)`                               | Totals poor ratings (adds all 1’s)                      |
| `COUNT(*)`                               | Counts total queries for that `query_name`              |
| `ROUND(..., 2)`                          | Rounds output to 2 decimal places                       |
| `GROUP BY query_name`                    | Aggregates results per query                            |

---

## **7. Key Concepts Explained** 🔍

### **(i) Why `COUNT(*)` is used**

* `COUNT(*)` counts **all rows** in each group.
* It provides the denominator for both:

  * Average calculation (`SUM / COUNT(*)`)
  * Percentage calculation (`poor / COUNT(*)`)

---

### **(ii) Why `CASE WHEN` is used**

SQL doesn’t allow direct logical conditions inside aggregate functions.
So, we use `CASE WHEN` as a conditional expression.

Example:

| rating | CASE WHEN rating < 3 THEN 1 ELSE 0 END |
| :----- | :------------------------------------- |
| 5      | 0                                      |
| 4      | 0                                      |
| 2      | 1                                      |

Then,
`SUM()` of this column = count of poor ratings.

---

### **(iii) Final Formula Recap**

[
\text{poor_query_percentage} = \frac{\text{SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END)}}{\text{COUNT(*)}} × 100
]

[
\text{quality} = \frac{\text{SUM(rating / position)}}{\text{COUNT(*)}}
]

---

## **8. Key Takeaways** 🔑

* **Single table problem** → No joins required.
* **Aggregation functions** (`SUM`, `COUNT`) are used for averages and ratios.
* **`CASE WHEN`** converts conditions into countable values (1 or 0).
* **`ROUND(..., 2)`** ensures precision matches expected output.
* Always `GROUP BY query_name` when computing per-query statistics.

---
