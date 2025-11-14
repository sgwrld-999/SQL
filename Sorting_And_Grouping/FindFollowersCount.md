### **Problem Name: Find Followers Count**

**Problem Link:** [LeetCode – 1729. Find Followers Count](https://leetcode.com/problems/find-followers-count/)

---

### **1. Problem Understanding**

We are given a table `Followers` with the following schema:

| Column      | Description                    |
| ----------- | ------------------------------ |
| user_id     | The user being followed        |
| follower_id | The user who follows `user_id` |

**Primary Key:** `(user_id, follower_id)`
This guarantees that the same follower cannot follow the same user more than once.

**Goal:**
Return each `user_id` along with the number of followers they have.

---

### **2. The Core Logic: Grouping + Counting**

Since each row in the `Followers` table represents a unique follower relationship:

* `user_id` is the entity we summarize.
* `follower_id` represents each individual follower.
* We simply need to count how many followers each user has.

| Task                      | SQL Operation          |
| ------------------------- | ---------------------- |
| Summarize by user         | `GROUP BY user_id`     |
| Count number of followers | `COUNT(follower_id)`   |
| Sort results by user      | `ORDER BY user_id ASC` |

Note:
`COUNT(user_id)` works because `user_id` is never null, but using `COUNT(follower_id)` is semantically more correct since we are counting followers.

---

### **3. Correct Query**

```sql
SELECT 
    user_id,
    COUNT(follower_id) AS followers_count
FROM 
    Followers
GROUP BY
    user_id
ORDER BY 
    user_id ASC;
```

---

### **4. Why This Query Is Correct**

1. **Each row corresponds to one follower relationship.**
   Therefore, counting rows per `user_id` yields the exact number of followers.

2. **Primary key ensures no duplicates.**
   `(user_id, follower_id)` prevents double-counting.

3. **Grouping by `user_id`** returns one output row per user.

4. **Sorting ensures ordered results** as required.

---

### **5. Example Walkthrough**

Assume the `Followers` table:

| user_id | follower_id |
| ------- | ----------- |
| 1       | 3           |
| 1       | 4           |
| 2       | 1           |
| 2       | 3           |
| 2       | 4           |

**Step 1:** Group by `user_id`
**Step 2:** Count distinct followers

**Output:**

| user_id | followers_count |
| ------- | --------------- |
| 1       | 2               |
| 2       | 3               |

---

### **6. Key Takeaways**

* Always group by `user_id` when summarizing followers.
* Use `COUNT(follower_id)` to count follower relationships.
* The primary key ensures each follower-user pair is unique.
* Sorting ensures an ordered, readable output.

---
