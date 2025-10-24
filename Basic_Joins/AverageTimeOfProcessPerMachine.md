## **Problem Name: Average Time of Process per Machine**

**Problem Link:** [LeetCode 1661 – Average Time of Process per Machine](https://leetcode.com/problems/average-time-of-process-per-machine/)

---

### **1. Problem Understanding**

We are given a table **`Activity`** that logs the start and end times of various processes running on different machines.

| Column          | Type                 | Description                                                      |
| --------------- | -------------------- | ---------------------------------------------------------------- |
| `machine_id`    | int                  | The ID of the machine on which the process ran                   |
| `process_id`    | int                  | The ID of the process running on that machine                    |
| `activity_type` | enum('start', 'end') | Indicates whether the record marks the start or end of a process |
| `timestamp`     | float                | The time when the activity occurred                              |

---

### **Goal**

For each `machine_id`, calculate the **average processing time** of all processes executed on that machine.

The **processing time** of a process is the difference between its end time and start time (`end_timestamp - start_timestamp`).

The output should have two columns:

| machine_id | processing_time |
| ---------- | --------------- |

where `processing_time` is rounded to **three decimal places**.

---

### **2. Core Logic**

The problem requires pairing each **start** record with its corresponding **end** record.
Since both records share the same `machine_id` and `process_id`, we can match them using an **INNER JOIN**.

Once matched, we can compute the difference between timestamps (`end - start`) for each process and then find the **average** processing time per machine.

#### **Key Points**

1. The **`INNER JOIN`** is performed on three matching conditions:

   * Same machine (`machine_id`)
   * Same process (`process_id`)
   * One record is start, the other is end (`activity_type < activity_type` ensures pairing of start with end)
2. The **`AVG()`** aggregation is used to compute the mean of all such differences.
3. The result is **grouped by `machine_id`** to get one row per machine.

---

### **3. Step-by-Step Query Breakdown**

#### **Step 1: Self-Join to Pair Start and End Activities**

```sql
FROM Activity a1
INNER JOIN Activity a2
ON a1.machine_id = a2.machine_id
AND a1.process_id = a2.process_id
AND a1.activity_type < a2.activity_type
```

This join matches every `start` record (`a1`) with its corresponding `end` record (`a2`) for the same machine and process.

#### **Step 2: Compute Time Difference**

```sql
a1.timestamp - a2.timestamp
```

Calculates how long a process took. The subtraction order ensures we get a positive duration (depending on activity order).

#### **Step 3: Take the Average per Machine**

```sql
AVG(a1.timestamp - a2.timestamp)
```

Computes the average processing time for all processes on each machine.

#### **Step 4: Group and Round**

```sql
GROUP BY a1.machine_id
ROUND(..., 3)
```

Ensures one output row per machine and rounds to three decimal places.

---

### **4. Full SQL Query**

```sql
# Write your MySQL query statement below
SELECT 
    a1.machine_id,
    ROUND(AVG(a1.timestamp - a2.timestamp), 3) AS processing_time
FROM 
    Activity a1 
    INNER JOIN Activity a2 
    ON a1.machine_id = a2.machine_id
    AND a1.process_id = a2.process_id
    AND a1.activity_type < a2.activity_type
GROUP BY 
    a1.machine_id;
```

---

### **5. Dry Run Example**

#### **Input Table (`Activity`):**

| machine_id | process_id | activity_type | timestamp |
| ---------- | ---------- | ------------- | --------- |
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |

---

#### **Step 1: Self-Join Result**

| a1.machine_id | a1.process_id | a1.activity_type | a1.timestamp | a2.activity_type | a2.timestamp |
| ------------- | ------------- | ---------------- | ------------ | ---------------- | ------------ |
| 0             | 0             | start            | 0.712        | end              | 1.520        |
| 0             | 1             | start            | 3.140        | end              | 4.120        |
| 1             | 0             | start            | 0.550        | end              | 1.550        |

---

#### **Step 2: Compute Time Difference**

| machine_id | process_id | duration              |
| ---------- | ---------- | --------------------- |
| 0          | 0          | 1.520 - 0.712 = 0.808 |
| 0          | 1          | 4.120 - 3.140 = 0.980 |
| 1          | 0          | 1.550 - 0.550 = 1.000 |

---

#### **Step 3: Compute Average per Machine**

| machine_id | average_processing_time     |
| ---------- | --------------------------- |
| 0          | (0.808 + 0.980) / 2 = 0.894 |
| 1          | 1.000                       |

---

#### **Final Output**

| machine_id | processing_time |
| ---------- | --------------- |
| 0          | 0.894           |
| 1          | 1.000           |

---

### **6. Alternative Approaches**

While the above query is the most concise, there are a few **alternate ways** to solve this:

#### **A. Using Conditional Aggregation (Subquery)**

Instead of joining, we can **aggregate timestamps directly** using conditional logic.

```sql
SELECT
    machine_id,
    ROUND(AVG(end_time - start_time), 3) AS processing_time
FROM (
    SELECT
        machine_id,
        process_id,
        MAX(CASE WHEN activity_type = 'end' THEN timestamp END) AS end_time,
        MAX(CASE WHEN activity_type = 'start' THEN timestamp END) AS start_time
    FROM Activity
    GROUP BY machine_id, process_id
) AS temp
GROUP BY machine_id;
```

**Explanation:**

* The subquery groups by `machine_id` and `process_id`, then extracts start and end timestamps using conditional aggregation.
* The outer query calculates the average duration per machine.

This avoids self-join and may perform better on large datasets.

---

#### **B. Using `CROSS APPLY` / `LATERAL JOIN` (in SQL Engines that Support It)**

If using SQL Server or PostgreSQL, we can also apply a **lateral join** to pair start–end values without explicit self-joins.

Example (conceptually similar, not MySQL):

```sql
SELECT machine_id, ROUND(AVG(end_t - start_t), 3) AS processing_time
FROM (
    SELECT 
        a.machine_id,
        a.process_id,
        MAX(CASE WHEN activity_type='start' THEN timestamp END) AS start_t,
        MAX(CASE WHEN activity_type='end' THEN timestamp END) AS end_t
    FROM Activity a
    GROUP BY a.machine_id, a.process_id
) sub
GROUP BY machine_id;
```

---

### **7. Key Takeaways**

* **Join Logic:**
  This problem uses a **self-join** on multiple conditions to pair related records.

* **Aggregation:**
  `AVG()` with `GROUP BY` is used to compute machine-level averages.

* **Rounding:**
  Always apply `ROUND(..., 3)` to match output precision requirements.

* **Alternative Solution:**
  Conditional aggregation (via subquery) is more efficient in large datasets and avoids multiple joins.

---
