### **Problem Name: User Activity for the Past 30 Days I**

**Problem Link:** [LeetCode – 1141. User Activity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/)

---

### **1. Problem Understanding**

We are given a table `Activity` that logs each user’s activity date and type.
Each row represents a single user’s action on a particular day.

| Column        | Type | Description                                      |
| ------------- | ---- | ------------------------------------------------ |
| user_id       | int  | Unique identifier for the user                   |
| activity_date | date | The date when the user performed activity        |
| activity_type | enum | (Not relevant here; can be login, comment, etc.) |

**Goal:**
Find the number of **unique active users per day** during the **last 30 days** from the reference date `'2019-07-27'`.

That is, include users whose `activity_date` falls within **[2019-06-27, 2019-07-27]**.

---

### **2. The Core Logic: Date Filtering + Grouping + Counting**

This is a **time-based aggregation** problem. The key operations are:

1. **Filter**: Keep records from the last 30 days.
2. **Group**: Aggregate by date (`activity_date`).
3. **Count**: Find the number of distinct users for each day.

| Task                 | SQL Operation                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| Filter last 30 days  | `WHERE DATEDIFF('2019-07-27', activity_date) < 30 AND DATEDIFF('2019-07-27', activity_date) >= 0` |
| Group by date        | `GROUP BY activity_date`                                                                          |
| Count distinct users | `COUNT(DISTINCT user_id)`                                                                         |
| Format output        | Rename columns as `day` and `active_users`                                                        |

---

### **3. Correct Query**

```sql
# Write your MySQL query statement below
SELECT 
    activity_date AS day, 
    COUNT(DISTINCT user_id) AS active_users
FROM 
    Activity
WHERE 
    DATEDIFF('2019-07-27', activity_date) < 30 
    AND DATEDIFF('2019-07-27', activity_date) >= 0
GROUP BY 
    activity_date
ORDER BY 
    activity_date;
```

**Why it’s correct:**

* Uses `DATEDIFF` to dynamically compute the 30-day range from `'2019-07-27'`.
* Groups by `activity_date` to compute daily aggregates.
* Counts distinct `user_id` values to avoid double-counting.
* Orders the output chronologically by date.

---

### **4. Common Mistake (Your Initial Query)**

**Incorrect Query:**

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

1. The column `activity_day` does **not exist**; it should be `activity_date`.
2. The filtering logic was valid, but grouping by a non-existent column produces an error.
3. Hardcoding date bounds works, but using `DATEDIFF()` makes it adaptive and cleaner.

**Fixed Query Using Date Difference (Corrected):**

```sql
SELECT 
    activity_date AS day, 
    COUNT(DISTINCT user_id) AS active_users
FROM 
    Activity
WHERE 
    DATEDIFF('2019-07-27', activity_date) < 30 
    AND DATEDIFF('2019-07-27', activity_date) >= 0
GROUP BY 
    activity_date;
```

---

### **5. Example Walkthrough**

**Input Table:**

| user_id | activity_date |
| ------- | ------------- |
| 1       | 2019-06-28    |
| 2       | 2019-06-28    |
| 1       | 2019-06-29    |
| 3       | 2019-07-01    |

**Step 1:** Filter records where date lies between `2019-06-27` and `2019-07-27`.
**Step 2:** Group by each `activity_date`.
**Step 3:** Count distinct users per date.

**Output:**

| day        | active_users |
| ---------- | ------------ |
| 2019-06-28 | 2            |
| 2019-06-29 | 1            |
| 2019-07-01 | 1            |

---

### **6. Key Takeaways**

* Use **`DATEDIFF()`** for flexible date range filtering without hardcoding.
* Always **verify column names** in `GROUP BY` and `SELECT`.
* Use **`COUNT(DISTINCT user_id)`** to count unique active users.
* Always **order by the date** for readable, chronological output.
* For fixed intervals like 30 days, ensure inclusivity using both upper and lower bounds.

---
