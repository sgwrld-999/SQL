### **Problem Name: Count Unique Subjects per Teacher**

**Problem Link:** [LeetCode – 2356. Number of Unique Subjects Taught by Each Teacher](https://leetcode.com/problems/number-of-unique-subjects-taught-by-each-teacher/)

---

### **1. Problem Understanding**

We are given a table **`Teacher`** with the following schema:

| Column     | Type | Description                                |
| ---------- | ---- | ------------------------------------------ |
| subject_id | int  | The subject being taught                   |
| dept_id    | int  | The department where the subject is taught |
| teacher_id | int  | The teacher responsible for the subject    |

**Goal:**
For each teacher (`teacher_id`), find the **number of distinct subjects** they teach.

Key observation:
A teacher may appear multiple times — possibly teaching the **same subject** across different departments — so we must ensure **unique counting per subject**.

---

### **2. The Core Logic: Grouping + Distinct Counting**

When you see a question asking for **“per teacher”** or **“per entity”** totals, think **GROUP BY**.
When you see **“unique”** or **“distinct”**, think **`COUNT(DISTINCT ...)`**.

| Requirement           | SQL Construct                |
| --------------------- | ---------------------------- |
| One row per teacher   | `GROUP BY teacher_id`        |
| Count unique subjects | `COUNT(DISTINCT subject_id)` |
| Rename the result     | `AS cnt`                     |

Together, these form the minimal correct aggregation query.

---

### **3. Step-by-Step Reasoning**

1. **Group by teacher:**
   We want results per teacher — so we group by `teacher_id`.

2. **Count distinct subjects:**
   We use `COUNT(DISTINCT subject_id)` to ensure subjects aren’t double-counted if they appear across multiple departments.

3. **Alias the result:**
   We rename the computed column as `cnt` (count).

---

### **4. Final Query**

```sql
SELECT 
    teacher_id,
    COUNT(DISTINCT subject_id) AS cnt
FROM 
    Teacher
GROUP BY 
    teacher_id;
```

**Explanation:**

* `COUNT(DISTINCT subject_id)` ensures we only count each subject once per teacher.
* `GROUP BY teacher_id` ensures one row per teacher.

---

### **5. Example Walkthrough**

**Input Table**

| subject_id | dept_id | teacher_id |
| ---------- | ------- | ---------- |
| 1          | 1       | 1          |
| 2          | 1       | 1          |
| 1          | 2       | 1          |
| 2          | 1       | 2          |
| 3          | 1       | 2          |
| 4          | 1       | 2          |
| 1          | 3       | 3          |

**Step 1 – Group by teacher:**

| teacher_id | subject_ids (grouped) |
| ---------- | --------------------- |
| 1          | {1, 2}                |
| 2          | {2, 3, 4}             |
| 3          | {1}                   |

**Step 2 – Count distinct subjects:**

| teacher_id | cnt |
| ---------- | --- |
| 1          | 2   |
| 2          | 3   |
| 3          | 1   |

---

### **6. Common Mistakes to Avoid**

1.  **Forgetting `GROUP BY`** → MySQL will throw an aggregate error.
2.  **Using simple `COUNT(subject_id)`** → Will overcount duplicates if a subject appears multiple times.
3.  **Adding unnecessary subqueries** → A single aggregation handles everything.

---

### **7. Key Takeaways**

* Use **`GROUP BY`** for per-entity summaries.
* Use **`COUNT(DISTINCT ...)`** whenever the question involves uniqueness.
* Keep it simple: avoid joins or nested subqueries when a single aggregation suffices.
* Always alias output columns meaningfully (`AS cnt`, `AS total`, etc.) for clarity.

---

This clean reasoning pattern — **Group → Aggregate → Label** — is universal across most SQL aggregation problems.