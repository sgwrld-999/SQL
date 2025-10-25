
## **Problem Name: Not Boring Movies**

**Problem Link:** [LeetCode 620 – Not Boring Movies](https://leetcode.com/problems/not-boring-movies/)

---

### **1. Problem Understanding**

We are given a table called **`Cinema`** that contains movie information.

#### **Table: Cinema**

| Column        | Type    | Description                        |
| ------------- | ------- | ---------------------------------- |
| `id`          | int     | Primary key identifying each movie |
| `movie`       | varchar | Name of the movie                  |
| `description` | varchar | Type or description of the movie   |
| `rating`      | float   | Rating score of the movie          |

---

### **2. Goal**

Find all **non-boring** movies that have **odd `id` numbers**, and display them in order of **descending ratings** (highest-rated first).

---

### **3. Core Logic**

The problem involves **filtering** and **sorting** data based on simple conditions:

1. **Odd ID Selection:**
   The condition `id % 2 != 0` ensures only movies with odd-numbered IDs are considered.
   The modulo operator (`%`) returns the remainder — if it’s not zero, the number is odd.

2. **Non-Boring Description:**
   We only include movies where the description is **not** equal to `"boring"`.

3. **Descending Order by Rating:**
   Once filtered, movies are sorted in descending order of their rating (`ORDER BY rating DESC`).

---

### **4. Step-by-Step Query Breakdown**

#### **Step 1: Filter Odd IDs**

```sql
WHERE c.id % 2 != 0
```

Selects only movies whose ID numbers are **odd**.

#### **Step 2: Filter Non-Boring Movies**

```sql
AND c.description != 'boring'
```

Ensures we exclude all movies described as “boring”.

#### **Step 3: Sort by Rating**

```sql
ORDER BY c.rating DESC
```

Sorts the final list so that higher-rated movies appear first.

---

### **5. Full SQL Query**

```sql
# Write your MySQL query statement below
SELECT
    c.id,
    c.movie,
    c.description,
    c.rating
FROM 
    Cinema c
WHERE 
    c.id % 2 != 0
    AND c.description != 'boring'
ORDER BY 
    c.rating DESC;
```

---

### **6. Dry Run Example**

#### **Cinema Table**

| id | movie     | description | rating |
| -- | --------- | ----------- | ------ |
| 1  | Joker     | exciting    | 8.9    |
| 2  | Frozen II | boring      | 7.5    |
| 3  | Parasite  | thrilling   | 9.1    |
| 4  | Cats      | boring      | 5.2    |
| 5  | Soul      | inspiring   | 8.3    |

---

#### **Step 1: Filter by `id % 2 != 0`**

Keep only rows with odd IDs:

| id | movie    | description | rating |
| -- | -------- | ----------- | ------ |
| 1  | Joker    | exciting    | 8.9    |
| 3  | Parasite | thrilling   | 9.1    |
| 5  | Soul     | inspiring   | 8.3    |

---

#### **Step 2: Exclude “boring” Descriptions**

All remaining movies are non-boring, so no change in this case.

---

#### **Step 3: Order by Rating (Descending)**

| id | movie    | description | rating |
| -- | -------- | ----------- | ------ |
| 3  | Parasite | thrilling   | 9.1    |
| 1  | Joker    | exciting    | 8.9    |
| 5  | Soul     | inspiring   | 8.3    |

---

#### **Final Output**

| id | movie    | description | rating |
| -- | -------- | ----------- | ------ |
| 3  | Parasite | thrilling   | 9.1    |
| 1  | Joker    | exciting    | 8.9    |
| 5  | Soul     | inspiring   | 8.3    |

---

### **7. Alternative Approaches**

#### **A. Using `MOD()` Function Instead of `%`**

Some SQL dialects prefer `MOD()` instead of `%` for computing remainders.

```sql
SELECT
    id, movie, description, rating
FROM 
    Cinema
WHERE 
    MOD(id, 2) = 1
    AND description <> 'boring'
ORDER BY 
    rating DESC;
```

Both `%` and `MOD()` behave equivalently in MySQL.

#### **B. Using `LOWER()` for Case-Insensitive Filtering**

If descriptions could appear in mixed case (e.g., “Boring”, “BORING”), we can ensure case-insensitive filtering:

```sql
WHERE 
    id % 2 != 0
    AND LOWER(description) <> 'boring'
```

---

### **8. Key Takeaways**

* **Modulo (`%` or `MOD()`)** is a simple and efficient way to filter even/odd records.
* **Filtering + Sorting** problems often require careful attention to operator precedence — ensure filters are applied before ordering.
* **Case sensitivity** may vary depending on table collation; using `LOWER()` or `UPPER()` ensures consistent filtering.
* Always test for **logical order** — filtering first, then sorting the resulting subset.

---

