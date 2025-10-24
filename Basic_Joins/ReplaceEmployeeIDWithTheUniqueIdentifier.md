### **Problem Name: Replace Employee ID With The Unique Identifier**

**Problem Link:** [LeetCode](https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier/)

-----

### **1. Problem Understanding** 

We are given two tables:

  * `Employees`: Contains the `id` and `name` of every employee.
  * `EmployeeUNI`: Contains the `id` and a `unique_id` for *some* of the employees.

**Task:**
We need to show the `unique_id` and `name` for **each** employee. If an employee does not have a unique ID, their `unique_id` should be shown as `null`.

The requirement to include **all** employees, even those without a match in the second table, is a clear signal that we need to use a `LEFT JOIN`.

-----

### **2. The Core Logic: The `LEFT JOIN`**

The key to this problem is understanding the difference between an `INNER JOIN` and a `LEFT JOIN`.

  * An **`INNER JOIN`** only returns rows where the join key (in this case, `id`) exists in **both** tables. If we used an `INNER JOIN` here, we would lose any employees who don't have a `unique_id`.

  * A **`LEFT JOIN`**, on the other hand, returns **all** rows from the left table (the one mentioned first, `Employees`) and the matched rows from the right table (`EmployeeUNI`). If there is no match in the right table for a row from the left table, the result will contain `NULL` values for the columns from the right table. This behavior is exactly what's needed to solve the problem.

-----

### **3. Constructing the Query (Step-by-Step)**

1.  **`SELECT empUNI.unique_id, emp.name`**: We specify the columns we want in our final output. We use aliases (`emp` and `empUNI`) to make it clear which column comes from which table.
2.  **`FROM Employees emp`**: We start with the `Employees` table as our "left" table because we want to keep every record from it.
3.  **`LEFT JOIN EmployeeUNI empUNI`**: We specify that we are joining with the `EmployeeUNI` table and that we want to keep all records from the left side (`Employees`).
4.  **`ON emp.id = empUNI.id`**: This is the join condition. It tells the database to match rows from both tables where the `id` column has the same value.

-----

### **4. Dry Run Example**

Let's trace the `LEFT JOIN` with sample data.

**`Employees` (Left Table):**
| id | name |
| :-- | :--- |
| 1 | Alice |
| 7 | Bob |
| 11 | Meir |

**`EmployeeUNI` (Right Table):**
| id | unique\_id |
| :-- | :--- |
| 7 | 2 |
| 11 | 3 |

**Join Process:**

1.  **Alice (`id=1`):** The query looks for `id=1` in `EmployeeUNI`. No match is found. The result is `(NULL, 'Alice')`.
2.  **Bob (`id=7`):** The query looks for `id=7` in `EmployeeUNI`. A match is found (`unique_id=2`). The result is `(2, 'Bob')`.
3.  **Meir (`id=11`):** The query looks for `id=11` in `EmployeeUNI`. A match is found (`unique_id=3`). The result is `(3, 'Meir')`.

**Final Output:**
| unique\_id | name |
| :--- | :--- |
| NULL | Alice |
| 2 | Bob |
| 3 | Meir |

-----

### **5. SQL Query**

```sql
SELECT
    empUNI.unique_id,
    emp.name
FROM
    Employees emp
LEFT JOIN
    EmployeeUNI empUNI ON emp.id = empUNI.id;
```

-----

### **6. Key Takeaways**

  * **`LEFT JOIN` for Inclusivity**: Use a `LEFT JOIN` when you need to retrieve all records from one table (the "left" one) and any matching records from a second table.
  * **`NULL` for Non-Matches**: `LEFT JOIN` is perfect for scenarios where a lack of a match is meaningful information; it represents this by returning `NULL`.
  * **Table Aliases**: Using aliases (like `emp` for `Employees`) is a best practice that makes queries with joins cleaner and easier to read.