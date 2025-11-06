### **Problem Name: User Activity for the Past 30 Days I**

**Problem Link:** [LeetCode – 1141. User Activity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/)

---

### **1. Problem Understanding**

We are given a table `Activity` containing user login or interaction data.
Each row records a user’s activity on a specific date.

| Column        | Type | Description                                           |
| ------------- | ---- | ----------------------------------------------------- |
| user_id       | int  | Unique identifier for the user                        |
| activity_date | date | The date when the user performed activity             |
| activity_type | enum | (Not relevant here, may include login, comment, etc.) |

**Goal:**
Find, for each day in a **given date range**, how many **unique users** were active (performed at least one activity).

**Given Range:**
From `'2019-06-27'` to `'2019-07-27'`.

---

### **2. The Core Logic: Filtering + Grouping + Counting**

The problem is a **date-based aggregation** task:

1. **Filter** records to only include the desired date range.
2. **Group** by date (`activity_date`).
3. **Count** the number of **distinct users** for each day.

| Task                | SQL Operation                                               |
| ------------------- | ----------------------------------------------------------- |
| Limit by date range | `WHERE activity_date BETWEEN '2019-06-27' AND '2019-07-27'` |
| Group by each date  | `GROUP BY activity_date`                                    |
| Count unique users  | `COUNT(DISTINCT user_id)`                                   |
| Format output       | Rename columns as `day` and `active_users`                  |

---

### **3. Correct Query**

```sql
SELECT 
    activity_date AS day,
    COUNT(DISTINCT user_id) AS active_users
FROM 
    Activity
WHERE 
    activity_date BETWEEN '2019-06-27' AND '2019-07-27'
GROUP BY 
    activity_date
ORDER BY 
    activity_date;
```

**Why it’s correct:**

* Uses `BETWEEN` for an inclusive date range filter.
* Groups by `activity_date` to get one row per day.
* Counts distinct users so multiple activities by the same user on the same day count once.
* Orders by `activity_date` for chronological readability.

---

### **4. Common Mistake (Your Initial Query)**

```sql
SELECT 
    activity_date AS day,
    COUNT(DISTINCT user_id) AS active_users
FROM 
    Activity
WHERE 
    activity_date >= '2019-06-27' AND activity_date <= '2019-07-27'
GROUP BY 
    activity_day;
```

**Issues:**

1. `activity_day` does **not exist** — correct column is `activity_date`.
2. The query logic was fine, but grouping by a non-existent column causes an error.
3. While using `>=` and `<=` works, `BETWEEN` is a cleaner, equivalent alternative.

---

### **5. Example Walkthrough**

| user_id | activity_date |
| ------- | ------------- |
| 1       | 2019-06-28    |
| 2       | 2019-06-28    |
| 1       | 2019-06-29    |
| 3       | 2019-07-01    |

**Step 1:** Filter dates between `2019-06-27` and `2019-07-27`.
**Step 2:** Group by each date.
**Step 3:** Count unique users.

| day        | active_users |
| ---------- | ------------ |
| 2019-06-28 | 2            |
| 2019-06-29 | 1            |
| 2019-07-01 | 1            |

---

### **6. Key Takeaways**

* **Always verify column names** used in `GROUP BY` and `SELECT`.
* **BETWEEN** is inclusive and cleaner than using `>=` and `<=`.
* Use **`COUNT(DISTINCT ...)`** whenever you’re asked for *unique users*.
* **ORDER BY date** to keep results in chronological order for readability.

---
