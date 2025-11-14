### **Problem Name: Classes With At Least Five Students**

**Problem Link:** [LeetCode – 596. Classes More Than 5 Students](https://leetcode.com/problems/classes-more-than-5-students/)

---

### **1. Problem Understanding**

We are given a table `Courses` with the following structure:

| Column  | Description                      |
| ------- | -------------------------------- |
| student | Name or ID of the student        |
| class   | Class the student is enrolled in |

**Goal:**
Return only those classes that have **five or more students**.

In other words, count how many students belong to each class, and output the class names where this count is at least 5.

---

### **2. The Core Logic: Grouping + Conditional Filtering**

This is a classic aggregation task.

The steps are:

1. **Group** rows by `class`, since we want statistics per class.
2. **Count** the number of students in each class.
3. **Filter** classes meeting the threshold using `HAVING`.

| Task                         | SQL Operation                |
| ---------------------------- | ---------------------------- |
| Group rows by class          | `GROUP BY class`             |
| Count number of students     | `COUNT(student)`             |
| Keep classes with at least 5 | `HAVING COUNT(student) >= 5` |

---

### **3. Correct Query**

```sql
SELECT 
    class
FROM 
    Courses
GROUP BY 
    class
HAVING 
    COUNT(student) >= 5;
```

**Why it is correct:**

* `GROUP BY class` ensures all students belonging to the same class are aggregated together.
* `COUNT(student)` counts the number of rows for each class.
* `HAVING COUNT(student) >= 5` filters out classes with fewer than five students.
* The query returns only the `class` column as required by the problem.

---

### **4. Common Mistake (Your Initial Query)**

Incorrect query:

```sql
SELECT
    (select from courses where COUNT(class) >= 5) where class
FROM  
    Courses;
```

**Issues:**

1. Nested `SELECT` inside another `SELECT` without fields is invalid syntax.
2. `COUNT()` cannot be used without `GROUP BY`.
3. Filtering aggregated results must be done using `HAVING`, not `WHERE`.
4. A subquery is unnecessary; a single aggregation query is sufficient.

---

### **5. Example Walkthrough**

**Input:**

| student | class   |
| ------- | ------- |
| A       | Math    |
| B       | Math    |
| C       | Math    |
| D       | Math    |
| E       | Math    |
| F       | History |

**Step 1:** Group by class
**Step 2:** Count students per class
**Step 3:** Keep only classes with counts >= 5

**Output:**

| class |
| ----- |
| Math  |

---

### **6. Key Takeaways**

* Use `GROUP BY` to aggregate rows by category.
* Use `HAVING` (not `WHERE`) to filter on aggregate conditions.
* `COUNT(student)` gives the number of entries per class.
* Keep queries simple and avoid unnecessary subqueries when grouping is sufficient.

---

If you'd like, I can also rewrite this in the full structured note format matching the earlier “Confirmation Rate” problem style.
