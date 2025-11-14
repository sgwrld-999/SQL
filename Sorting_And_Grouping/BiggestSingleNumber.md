### **Problem Name: Biggest Single Number**

**Problem Link:** [LeetCode – 619. Biggest Single Number](https://leetcode.com/problems/biggest-single-number/)

---

### **1. Problem Understanding**

We are given a table `MyNumbers` containing a single column:

| Column | Description               |
| ------ | ------------------------- |
| num    | A number from the dataset |

Numbers may appear multiple times.

**Goal:**
Find the **largest number** that appears **exactly once** in the table.

Example input:

| num |
| --- |
| 8   |
| 8   |
| 3   |
| 3   |
| 1   |
| 4   |
| 5   |
| 6   |

The numbers that appear only once are 1, 4, 5, and 6.
The largest among them is **6**.

---

### **2. The Core Logic: Grouping + Filtering Unique Values + Selecting Maximum**

To solve the problem, break it into the following steps:

1. **Group by num** to compute how many times each number appears.
2. **Filter** only those numbers with `COUNT(num) = 1`.
3. **Select the largest number** among those unique numbers using `MAX(num)`.

| Task                               | SQL Operation                 |
| ---------------------------------- | ----------------------------- |
| Count occurrences of each number   | `GROUP BY num`                |
| Filter to numbers that appear once | `HAVING COUNT(num) = 1`       |
| Select largest number              | `MAX(num)` on filtered values |

---

### **3. Incorrect Query and Why It Fails**

Your attempt:

```sql
SELECT 
    MAX(num) as num
FROM 
    MyNumbers
WHERE (num) IN (
    SELECT COUNT(num) as num
    FROM MyNumbers
    GROUP BY num
) 
GROUP BY 
    num
```

**Issues:**

1. Inner query returns **counts**, not the original numbers.
   Example counts: `2, 2, 1, 1, 1, 1`.

2. The outer query tries to compare:
   `num IN (1, 2)`
   but `num` contains values such as 8, 3, 1, 4, 5, 6.

3. `GROUP BY num` in the outer query conflicts with `MAX(num)`, producing incorrect records.

Therefore, the logic breaks because it compares numbers against counts rather than comparing numbers against numbers.

---

### **4. Correct Query**

```sql
SELECT 
    MAX(num) AS num
FROM 
    MyNumbers
WHERE num IN (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(num) = 1
);
```

---

### **5. Why This Query Works**

1. **Inner Query**

```sql
SELECT num
FROM MyNumbers
GROUP BY num
HAVING COUNT(num) = 1
```

* Collects all numbers that appear exactly once.

2. **Outer Query**

```sql
SELECT MAX(num)
```

* Selects the maximum among these unique numbers.

This approach compares numbers to numbers, which is correct.

---

### **6. Example Walkthrough**

Given:

| num |
| --- |
| 8   |
| 8   |
| 3   |
| 3   |
| 1   |
| 4   |
| 5   |
| 6   |

**Step 1:** Group by `num`

| num | count |
| --- | ----- |
| 8   | 2     |
| 3   | 2     |
| 1   | 1     |
| 4   | 1     |
| 5   | 1     |
| 6   | 1     |

**Step 2:** Filter where count = 1
Remaining numbers → 1, 4, 5, 6.

**Step 3:** Take the maximum
Output → **6**

---

### **7. Key Takeaways**

* Always compare values of the same type: numbers should be compared to numbers, not counts.
* Use `HAVING COUNT(...) = 1` to filter groups based on aggregated values.
* When asked for the largest or smallest value among filtered groups, use `MAX()` or `MIN()`.
* Avoid unnecessary `GROUP BY` in the outer query unless aggregate logic requires it.

---
