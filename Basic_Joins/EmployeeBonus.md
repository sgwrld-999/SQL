## **Problem Name: Employee Bonus**

**Problem Link:** [LeetCode 577 – Employee Bonus](https://leetcode.com/problems/employee-bonus/)

---

### **1. Problem Understanding**

We are given two tables — **`Employee`** and **`Bonus`** — and are asked to find employees whose **bonus is either less than 1000 or missing entirely (NULL)**.

---

#### **Table: Employee**

| Column       | Type    | Description                                   |
| ------------ | ------- | --------------------------------------------- |
| `empId`      | int     | Primary key (unique employee ID)              |
| `name`       | varchar | Employee name                                 |
| `supervisor` | int     | Supervisor’s ID (not relevant for this query) |
| `salary`     | int     | Employee’s salary                             |

---

#### **Table: Bonus**

| Column  | Type | Description                            |
| ------- | ---- | -------------------------------------- |
| `empId` | int  | Foreign key referencing Employee.empId |
| `bonus` | int  | Bonus amount                           |

---

### **Goal**

Return the list of all employees whose **bonus is less than 1000 or NULL** (meaning they did not receive any bonus).
The result should include:

* `name`
* `bonus`

---

### **2. Core Logic**

This is a **LEFT JOIN + NULL check** problem — similar in concept to identifying missing or low-value relationships across tables.

#### **Key Observations**

* Not all employees may appear in the `Bonus` table (some employees have no bonus record).
* Hence, we must use a **LEFT JOIN** to ensure all employees are retained.
* After joining, `NULL` in the `bonus` column indicates “no bonus assigned.”
* Finally, apply a condition to filter out only those with `bonus < 1000 OR bonus IS NULL`.

---

### **3. Step-by-Step Query Breakdown**

#### **Step 1: LEFT JOIN**

```sql
FROM Employee e
LEFT JOIN Bonus b
ON e.empId = b.empId
```

* This join keeps all employees, regardless of whether a bonus record exists.
* If no matching record exists in `Bonus`, the columns from `Bonus` will be `NULL`.

#### **Step 2: Filter**

```sql
WHERE b.bonus < 1000 OR b.bonus IS NULL
```

* Keeps employees whose bonus is either missing (`NULL`) or below 1000.

#### **Step 3: Select Output**

```sql
SELECT e.name, b.bonus
```

* Displays the employee’s name and their corresponding bonus.

---

### **4. Full SQL Query**

```sql
# Write your MySQL query statement below
SELECT
    e.name,
    b.bonus
FROM 
    Employee e
LEFT JOIN 
    Bonus b
ON 
    e.empId = b.empId
WHERE 
    b.bonus < 1000 OR b.bonus IS NULL;
```

---

### **5. Dry Run Example**

#### **Employee Table**

| empId | name   | salary |
| ----- | ------ | ------ |
| 1     | John   | 1000   |
| 2     | Dan    | 2000   |
| 3     | Brad   | 1500   |
| 4     | Thomas | 4000   |

#### **Bonus Table**

| empId | bonus |
| ----- | ----- |
| 2     | 500   |
| 4     | 2000  |

---

#### **Step 1: LEFT JOIN Result**

| e.empId | e.name | b.bonus |
| ------- | ------ | ------- |
| 1       | John   | NULL    |
| 2       | Dan    | 500     |
| 3       | Brad   | NULL    |
| 4       | Thomas | 2000    |

---

#### **Step 2: Apply Filter (`b.bonus < 1000 OR b.bonus IS NULL`)**

| e.name | b.bonus |
| ------ | ------- |
| John   | NULL    |
| Dan    | 500     |
| Brad   | NULL    |

---

#### **Final Output**

| name | bonus |
| ---- | ----- |
| John | NULL  |
| Dan  | 500   |
| Brad | NULL  |

---

### **6. Alternate Approaches**

#### **A. Using a Subquery**

You can achieve the same result using a `NOT IN` + `UNION` approach, though it’s less efficient:

```sql
SELECT name, bonus
FROM Employee e
JOIN Bonus b
ON e.empId = b.empId
WHERE b.bonus < 1000

UNION

SELECT name, NULL AS bonus
FROM Employee e
WHERE e.empId NOT IN (SELECT empId FROM Bonus);
```

**Explanation:**

* The first part finds employees with bonus < 1000.
* The second part finds employees with no bonus record at all.
* The `UNION` merges both result sets.

---

#### **B. Using `COALESCE()` for Cleaner Output**

If you prefer to display a numeric value (like 0) instead of NULL bonuses:

```sql
SELECT
    e.name,
    COALESCE(b.bonus, 0) AS bonus
FROM 
    Employee e
LEFT JOIN 
    Bonus b
ON 
    e.empId = b.empId
WHERE 
    b.bonus < 1000 OR b.bonus IS NULL;
```

`COALESCE()` replaces `NULL` with `0` (or any desired default value).

---

### **7. Key Takeaways**

* **LEFT JOIN** ensures all employees are included, even if they lack a bonus record.
* **`IS NULL`** is essential to detect missing entries in joined tables.
* Combining `LEFT JOIN` with conditional filters like `< 1000 OR IS NULL` is a standard approach for “missing or low value” queries.
* Alternative methods like **subqueries** or **COALESCE()** can provide flexibility in reporting style.

---
