\## **Problem Name: Managers with at Least 5 Direct Reports**

**Problem Link:** [LeetCode 570 – Managers with at Least 5 Direct Reports](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/)

---

### **1. Problem Understanding**

We are given a single table named **`Employee`**, which stores employee details along with their respective manager.

#### **Table: Employee**

| Column       | Type    | Description                                                 |
| ------------ | ------- | ----------------------------------------------------------- |
| `id`         | int     | Primary key representing each employee                      |
| `name`       | varchar | Employee name                                               |
| `department` | varchar | Department name (not relevant for this problem)             |
| `managerId`  | int     | ID of the employee’s manager (foreign key referencing `id`) |

---

### **2. Goal**

Find all **managers** who have **at least 5 direct reports** — i.e., employees who report directly to them.

Output only the **name** of each qualifying manager.

---

### **3. Core Logic**

This is a classic **self-join** problem — since both employees and managers are part of the same table, we must join the table with itself to establish the relationship.

#### **Key Concepts:**

* Every employee has a `managerId` that points to another employee’s `id`.
* By joining the table to itself, we can associate each manager with all their direct reports.
* Then, by grouping by the manager’s ID and counting the number of subordinates, we can identify which managers have five or more.

---

### **4. Step-by-Step Query Breakdown**

#### **Step 1: Self-Join the Table**

```sql
FROM Employee m
JOIN Employee e
ON m.id = e.managerId
```

* `m` represents the manager table alias.
* `e` represents the employee table alias.
* The join condition `m.id = e.managerId` pairs each manager with all employees reporting to them.

#### **Step 2: Group by Manager**

```sql
GROUP BY m.id
```

This groups all direct reports under each manager, allowing aggregation functions (like `COUNT`) to work per manager.

#### **Step 3: Filter by Count**

```sql
HAVING COUNT(e.id) >= 5
```

* The `HAVING` clause is used instead of `WHERE` because it filters after aggregation.
* Only managers with 5 or more employees appear in the result.

#### **Step 4: Select the Manager’s Name**

```sql
SELECT m.name
```

This returns the names of the managers who satisfy the condition.

---

### **5. Full SQL Query**

```sql
# Write your MySQL query statement below
SELECT 
    m.name
FROM 
    Employee m
JOIN 
    Employee e
ON 
    m.id = e.managerId
GROUP BY 
    m.id
HAVING 
    COUNT(e.id) >= 5;
```

---

### **6. Dry Run Example**

#### **Employee Table**

| id | name  | managerId |
| -- | ----- | --------- |
| 1  | John  | NULL      |
| 2  | Dan   | 1         |
| 3  | James | 1         |
| 4  | Amy   | 1         |
| 5  | Anne  | 1         |
| 6  | Ron   | 1         |
| 7  | Brad  | 2         |

---

#### **Step 1: Self-Join (m.id = e.managerId)**

| m.id | m.name | e.id | e.name |
| ---- | ------ | ---- | ------ |
| 1    | John   | 2    | Dan    |
| 1    | John   | 3    | James  |
| 1    | John   | 4    | Amy    |
| 1    | John   | 5    | Anne   |
| 1    | John   | 6    | Ron    |
| 2    | Dan    | 7    | Brad   |

---

#### **Step 2: Group by Manager**

| m.id | m.name | COUNT(e.id) |
| ---- | ------ | ----------- |
| 1    | John   | 5           |
| 2    | Dan    | 1           |

---

#### **Step 3: Apply `HAVING COUNT(e.id) >= 5`**

| m.name |
| ------ |
| John   |

---

#### **Final Output**

| name |
| ---- |
| John |

---

### **7. Alternative Approaches**

Although the self-join method is the most common and intuitive, there are other valid techniques.

#### **A. Using Subquery and GROUP BY**

```sql
SELECT 
    name
FROM 
    Employee
WHERE 
    id IN (
        SELECT managerId
        FROM Employee
        WHERE managerId IS NOT NULL
        GROUP BY managerId
        HAVING COUNT(*) >= 5
    );
```

**Explanation:**

* The inner query identifies all manager IDs who have at least five subordinates.
* The outer query retrieves the corresponding names.

---

#### **B. Using Common Table Expression (CTE)**

```sql
WITH ManagerCounts AS (
    SELECT managerId, COUNT(*) AS report_count
    FROM Employee
    WHERE managerId IS NOT NULL
    GROUP BY managerId
)
SELECT e.name
FROM Employee e
JOIN ManagerCounts mc
ON e.id = mc.managerId
WHERE mc.report_count >= 5;
```

**Explanation:**

* The CTE computes report counts for all managers.
* The outer query joins this result to fetch only the names of managers with ≥5 reports.
* This approach improves readability for complex queries.

---

### **8. Key Takeaways**

* **Self-Join**: Used to relate rows within the same table (employee ↔ manager).
* **GROUP BY + HAVING**: Essential for filtering based on aggregated counts.
* **Alternative Options**: Subqueries and CTEs can achieve the same logic with different readability trade-offs.
* **Performance**: For large datasets, indexing `managerId` can significantly speed up join and group operations.

---

