### **Problem Name: Article Views I** 

**Problem Link:** [LeetCode](https://leetcode.com/problems/article-views-i/)

-----

### **1. Problem Understanding**

We are given a `Views` table that logs every instance an article is viewed. Each row captures the `article_id`, its `author_id`, the `viewer_id`, and the `view_date`.

**Task:**
The goal is to find all authors who have viewed at least one of their own articles. The final output must be a list of these author IDs, ensuring the list is **unique** and **sorted** in ascending order.

-----

### **2. The Core Logic: Self-Comparison and Uniqueness**

The problem's logic has two key parts:

1.  **Identification**: The main task is to find rows where the person who wrote the article (`author_id`) is the same as the person who viewed it (`viewer_id`). This is achieved with a direct equality check in the `WHERE` clause: `author_id = viewer_id`. This condition is evaluated for every row, filtering for only those that represent an author viewing their own work.

2.  **Uniqueness**: An author might view their own articles multiple times (e.g., viewing the same article on different days or viewing several of their different articles). This would create multiple rows for the same author that satisfy our `WHERE` condition. Since the problem demands a unique list of authors, we must use the **`DISTINCT`** keyword to remove these duplicates from our final result.

-----

### **3. Constructing the Query (Step-by-Step)**

1.  **`SELECT author_id`**: We start by specifying that we want to retrieve the `author_id`.
2.  **`FROM Views`**: We identify the `Views` table as our data source.
3.  **`WHERE author_id = viewer_id`**: This is the critical filter that keeps only the rows where the author and viewer are the same person.
4.  **`DISTINCT`**: We add this keyword to `SELECT` to guarantee that each author ID in the final list is unique.
5.  **`ORDER BY author_id`**: Finally, we sort the unique list of author IDs in ascending order, as required.

-----

### **4. Dry Run Example**

Let's trace the query using a sample `Views` table.

| article\_id | author\_id | viewer\_id | `author_id = viewer_id`? | Keep/Discard |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 3 | 5 | FALSE | **Discard** |
| 1 | 3 | 3 | TRUE | **Keep `author_id` 3** |
| 2 | 7 | 7 | TRUE | **Keep `author_id` 7** |
| 3 | 4 | 4 | TRUE | **Keep `author_id` 4** |
| 2 | 7 | 6 | FALSE | **Discard** |
| 3 | 4 | 4 | TRUE | **Keep `author_id` 4** |

1.  **After `WHERE`**: The list of `author_id`s is `(3, 7, 4, 4)`.
2.  **After `DISTINCT`**: The list is made unique: `(3, 7, 4)`.
3.  **After `ORDER BY`**: The final list is sorted: `(3, 4, 7)`.

-----

### **5. SQL Query**

```sql
SELECT
    DISTINCT author_id AS id
FROM
    Views
WHERE
    author_id = viewer_id
ORDER BY
    author_id;
```

-----

### **6. Key Takeaways** 

  * **Row-wise Logic**: The `WHERE` clause is excellent for comparing values in different columns *within the same row* to find specific patterns.
  * **`DISTINCT` is for Uniqueness**: Whenever a problem asks for a "unique list," the `DISTINCT` keyword should be your first consideration to eliminate duplicate entries.
  * **`ORDER BY` for Formatting**: Use `ORDER BY` to present your final results in a sorted manner, which is often a requirement for the answer to be considered correct.