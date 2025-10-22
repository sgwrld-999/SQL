## **Problem Name:** Find Customer Referee

**Problem Link:** [LeetCode](https://leetcode.com/problems/find-customer-referee/)

### **1. Problem Understanding**

We have a `Customer` table with `id`, `name`, and `referee_id`. The `referee_id` for a customer is the `id` of the customer who referred them.

**Task:**
Find the names of all customers who were **not** referred by the customer with `id = 2`. This includes customers who were referred by someone else and customers who had no referee at all.

**Example:**
*Input:*
`Customer` table:

```
+----+------+------------+
| id | name | referee_id |
+----+------+------------+
| 1  | Will | NULL       |
| 2  | Jane | NULL       |
| 3  | Alex | 2          |
| 4  | Bill | NULL       |
| 5  | Zack | 1          |
| 6  | Mark | 2          |
+----+------+------------+
```

*Output:*

```
+------+
| name |
+------+
| Will |
| Jane |
| Bill |
| Zack |
+------+
```

*Explanation:* Alex and Mark were referred by `id = 2`, so they are excluded. Will, Jane, and Bill had no referee (`NULL`), and Zack was referred by `id = 1`. All four are included in the result.

-----

### **2. The Core Logic: Handling `NULL`**

The most important concept in this problem is how SQL treats `NULL`.

`NULL` isn't a value like `0` or an empty string; it represents the **absence of a value**. Because of this, you can't use standard comparison operators like `=` or `!=` to check for `NULL`.

  * Any comparison to `NULL` (e.g., `referee_id = NULL` or `referee_id != 2` when `referee_id` is `NULL`) results in `UNKNOWN`.
  * The `WHERE` clause only includes rows where the condition is `TRUE`. It treats `UNKNOWN` and `FALSE` the same way—by discarding the row.

Therefore, a query like `WHERE referee_id != 2` would incorrectly filter out all the customers who have `NULL` as their `referee_id`. To solve this, we must create two separate conditions and combine them.

1.  Find customers whose `referee_id` is not 2.
2.  Find customers whose `referee_id` **IS NULL**.

Since a customer is valid if they meet *either* of these conditions, we combine them with an `OR` operator.

-----

### **3. Constructing the Query (Step-by-Step)**

1.  **Select the Column**: We need to return the customer's name.
      * `SELECT name`
2.  **Specify the Table**: The data comes from the `Customer` table.
      * `FROM Customer`
3.  **Apply the Filters**: We use a `WHERE` clause to define our conditions.
      * Condition 1: The referee ID is not 2 (`referee_id != 2`).
      * Condition 2: The referee ID is missing (`referee_id IS NULL`).
4.  **Combine Conditions**: Link them with `OR` to include rows that satisfy either condition.
      * `WHERE referee_id != 2 OR referee_id IS NULL`

-----

### **4. Dry Run Example**

Let's trace the logic `WHERE referee_id != 2 OR referee_id IS NULL` on the example table.

| id | name | referee\_id | `referee_id != 2`? | `referee_id IS NULL`? | `OR` Result | Keep/Discard |
| :- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Will | NULL | UNKNOWN | TRUE | **TRUE** | **Keep** |
| 2 | Jane | NULL | UNKNOWN | TRUE | **TRUE** | **Keep** |
| 3 | Alex | 2 | FALSE | FALSE | **FALSE** | **Discard** |
| 4 | Bill | NULL | UNKNOWN | TRUE | **TRUE** | **Keep** |
| 5 | Zack | 1 | TRUE | FALSE | **TRUE** | **Keep** |
| 6 | Mark | 2 | FALSE | FALSE | **FALSE** | **Discard** |

The query correctly keeps Will, Jane, Bill, and Zack.

-----

### **5. SQL Query**

```sql
# Write your MySQL query statement below
SELECT 
    name
FROM 
    Customer
WHERE
    referee_id != 2 OR referee_id IS NULL;
```

-----

### **6. Key Takeaways**

  * **`NULL` is not a value**: Treat `NULL` as an unknown or missing state, not as the number zero or an empty string.
  * **Use `IS NULL` / `IS NOT NULL`**: This is the only correct way to explicitly check for the presence or absence of a value in a column.
  * **Combine Conditions**: Use `OR` to create filters that allow rows meeting one of several different criteria.