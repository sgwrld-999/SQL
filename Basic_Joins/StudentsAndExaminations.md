## **Problem Name: Students and Examinations**

**Problem Link:** [LeetCode 1280 – Students and Examinations](https://leetcode.com/problems/students-and-examinations/)

---

### **1. Problem Understanding**

We are given three relational tables that record information about students, subjects, and examination attendance.

#### **Table: Students**

| Column         | Type    | Description                           |
| -------------- | ------- | ------------------------------------- |
| `student_id`   | int     | Primary key representing each student |
| `student_name` | varchar | Name of the student                   |

#### **Table: Subjects**

| Column         | Type    | Description                           |
| -------------- | ------- | ------------------------------------- |
| `subject_name` | varchar | Primary key representing each subject |

#### **Table: Examinations**

| Column         | Type    | Description                                     |
| -------------- | ------- | ----------------------------------------------- |
| `student_id`   | int     | Foreign key referencing `Students.student_id`   |
| `subject_name` | varchar | Foreign key referencing `Subjects.subject_name` |

---

### **2. Goal**

For every possible combination of student and subject, we must count **how many exams** that student attended for that subject.

If a student has **never appeared** for a particular subject, they must still be listed with an attendance count of `0`.

---

### **3. Core Logic**

This problem involves generating a **complete mapping** of all student–subject pairs, followed by attaching their actual exam attendance information.

To achieve this, we use:

1. **`CROSS JOIN`** → to generate all possible student–subject pairs
2. **`LEFT JOIN`** → to attach exam data (if available)
3. **`COUNT()`** → to calculate the total number of times each student took that subject

---

### **4. Step-by-Step Query Breakdown**

#### **Step 1: Create All Student–Subject Pairs**

```sql
FROM Students s
CROSS JOIN Subjects sub
```

This forms every possible combination of students and subjects.
If there are `m` students and `n` subjects, this produces `m × n` rows.

#### **Step 2: Attach Exam Records**

```sql
LEFT JOIN Examinations e
ON s.student_id = e.student_id
AND sub.subject_name = e.subject_name
```

This join connects each student–subject pair to their corresponding exam record (if it exists).
If no exam record is found, columns from `Examinations` become `NULL`.

#### **Step 3: Count the Exams**

```sql
COUNT(e.subject_name) AS attended_exams
```

Since `COUNT()` ignores `NULL` values, this correctly returns `0` for subjects the student never attended.

#### **Step 4: Group and Order the Results**

```sql
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

Grouping ensures one row per student–subject pair, and ordering aligns with the expected output format.

---

### **5. Final SQL Query**

```sql
SELECT 
    s.student_id,
    s.student_name,
    sub.subject_name,
    COUNT(e.subject_name) AS attended_exams
FROM 
    Students s
CROSS JOIN 
    Subjects sub
LEFT JOIN 
    Examinations e
ON 
    s.student_id = e.student_id
    AND sub.subject_name = e.subject_name
GROUP BY 
    s.student_id, s.student_name, sub.subject_name
ORDER BY 
    s.student_id, sub.subject_name;
```

---

### **6. Dry Run Example**

#### **Students**

| student_id | student_name |
| ---------- | ------------ |
| 1          | Alice        |
| 2          | Bob          |

#### **Subjects**

| subject_name |
| ------------ |
| Math         |
| Physics      |

#### **Examinations**

| student_id | subject_name |
| ---------- | ------------ |
| 1          | Math         |
| 1          | Math         |
| 2          | Physics      |

---

#### **Step 1: CROSS JOIN Output**

| student_id | student_name | subject_name |
| ---------- | ------------ | ------------ |
| 1          | Alice        | Math         |
| 1          | Alice        | Physics      |
| 2          | Bob          | Math         |
| 2          | Bob          | Physics      |

---

#### **Step 2: LEFT JOIN with Examinations**

| student_id | student_name | subject_name | e.subject_name |
| ---------- | ------------ | ------------ | -------------- |
| 1          | Alice        | Math         | Math           |
| 1          | Alice        | Math         | Math           |
| 1          | Alice        | Physics      | NULL           |
| 2          | Bob          | Math         | NULL           |
| 2          | Bob          | Physics      | Physics        |

---

#### **Step 3: After GROUP BY and COUNT()**

| student_id | student_name | subject_name | attended_exams |
| ---------- | ------------ | ------------ | -------------- |
| 1          | Alice        | Math         | 2              |
| 1          | Alice        | Physics      | 0              |
| 2          | Bob          | Math         | 0              |
| 2          | Bob          | Physics      | 1              |

---

### **7. Alternative Approaches**

Although the `CROSS JOIN` + `LEFT JOIN` approach is the most readable and direct, there are a few other valid strategies.

#### **A. Using Nested Subquery with Join**

```sql
SELECT 
    s.student_id,
    s.student_name,
    sub.subject_name,
    COALESCE(COUNT(e.subject_name), 0) AS attended_exams
FROM Students s, Subjects sub
LEFT JOIN Examinations e
ON e.student_id = s.student_id AND e.subject_name = sub.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name;
```

Here, the implicit comma join (`Students, Subjects`) behaves like a `CROSS JOIN`.

#### **B. Using Derived Table for All Combinations**

```sql
SELECT 
    c.student_id,
    c.student_name,
    c.subject_name,
    COUNT(e.subject_name) AS attended_exams
FROM (
    SELECT s.student_id, s.student_name, sub.subject_name
    FROM Students s
    JOIN Subjects sub
) AS c
LEFT JOIN Examinations e
ON c.student_id = e.student_id AND c.subject_name = e.subject_name
GROUP BY c.student_id, c.student_name, c.subject_name
ORDER BY c.student_id, c.subject_name;
```

This approach explicitly creates a temporary table of all combinations before attaching exam data.

---

### **8. Key Takeaways**

* **CROSS JOIN** ensures full coverage of student–subject pairs.
* **LEFT JOIN** maintains students who never attended a subject.
* **COUNT(column_name)** correctly returns zero when no matching rows exist.
* The **combination of both joins** is a classic pattern for “include missing pairs” problems in SQL.
* Alternative approaches can use subqueries or derived tables for the same logic but with less readability.

---
