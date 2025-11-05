# **Problem Name: Percentage of Users Attended a Contest**

**(LeetCode #1633)**

---

## **1. Problem Understanding** 

We are given two tables:

* **`Users`** – contains unique user IDs.
* **`Register`** – maps each contest (`contest_id`) to the users (`user_id`) who registered for it.
  A single contest can have multiple registered users.

---

### **Task**

Find the **percentage of users** who registered for each contest.

**Formula:**
[
\text{percentage} = \frac{\text{number of users registered for the contest}}{\text{total number of users}} \times 100
]

**Output requirements:**

* Round the percentage to **two decimal places**.
* Sort results by:

  1. `percentage DESC`
  2. `contest_id ASC`

---

## **2. The Core Logic: Counting, Division, and Rounding**

The problem involves three main logical components:

1. **Counting registered users per contest**

   * Using `COUNT(r.user_id)` counts how many users attended each contest.

2. **Calculating the total number of users**

   * `(SELECT COUNT(user_id) FROM Users)` gives the **total number of users**, used as the denominator in our percentage formula.

3. **Computing percentage**

   * Multiply the fraction by `100.0` to convert it to a percentage.
   * Use `ROUND(..., 2)` to round to two decimal places.

4. **Sorting the results**

   * Use `ORDER BY percentage DESC, contest_id ASC` to match problem requirements.

---

## **3. Constructing the Query (Step-by-Step)**

1. **Start from the `Register` table**
   We have one row per user per contest, so it’s ideal for grouping.

   ```sql
   FROM Register r
   ```

2. **Group by contest**

   ```sql
   GROUP BY r.contest_id
   ```

3. **Count the number of users per contest**

   ```sql
   COUNT(r.user_id)
   ```

4. **Find the total number of users**

   ```sql
   (SELECT COUNT(user_id) FROM Users)
   ```

5. **Compute and round the percentage**

   ```sql
   ROUND(COUNT(r.user_id) * 100.0 / (SELECT COUNT(user_id) FROM Users), 2)
   ```

6. **Apply ordering**

   ```sql
   ORDER BY percentage DESC, r.contest_id ASC
   ```

---

## **4. Dry Run Example**

### **`Users` Table**

| user_id |
| :-----: |
|    6    |
|    2    |
|    1    |
|    3    |

### **`Register` Table**

| contest_id | user_id |
| :--------: | :-----: |
|     208    |    6    |
|     208    |    2    |
|     208    |    1    |
|     209    |    2    |
|     209    |    3    |
|     210    |    6    |
|     210    |    1    |
|     210    |    3    |
|     210    |    2    |

---

### **Step 1: Group by contest**

| contest_id | users_in_contest |
| :--------: | :--------------: |
|     208    |         3        |
|     209    |         2        |
|     210    |         4        |

### **Step 2: Total users**

[
\text{total users} = 4
]

### **Step 3: Calculate percentage**

| contest_id | users_in_contest | total_users |      percentage      |
| :--------: | :--------------: | :---------: | :------------------: |
|     208    |         3        |      4      |  (3/4) × 100 = 75.00 |
|     209    |         2        |      4      |  (2/4) × 100 = 50.00 |
|     210    |         4        |      4      | (4/4) × 100 = 100.00 |

---

### **Step 4: Sort Results**

* First by `percentage DESC`
* Then by `contest_id ASC`

**Final Output**

| contest_id | percentage |
| :--------: | :--------: |
|     210    |   100.00   |
|     208    |    75.00   |
|     209    |    50.00   |

---

## **5. SQL Query**

```sql
SELECT 
    r.contest_id,
    ROUND(
        COUNT(r.user_id) * 100.0 / (SELECT COUNT(user_id) FROM Users),
        2
    ) AS percentage
FROM 
    Register r
GROUP BY 
    r.contest_id
ORDER BY 
    percentage DESC, 
    r.contest_id ASC;
```

---

## **6. Key Takeaways** 

* **Percentage formula:**
  `COUNT(r.user_id) * 100.0 / (SELECT COUNT(user_id) FROM Users)`
* **Subquery usage:**
  Efficient for fixed denominators (total users constant across contests).
* **Avoid unnecessary joins:**
  The `Register` table already contains all needed data.
* **Sorting:**
  Always separate multiple sort criteria using commas, not conditional statements.
* **Rounding:**
  Use `ROUND(..., 2)` to match the precision required by LeetCode.

---
