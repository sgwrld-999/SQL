## **Problem Name: Rising Temperature**

**Problem Link:** [LeetCode 197 – Rising Temperature](https://leetcode.com/problems/rising-temperature/)

---

### **1. Problem Understanding**

We are given a single table named **`Weather`**, which records the temperature of each day.

| Column        | Type | Description                                   |
| ------------- | ---- | --------------------------------------------- |
| `id`          | int  | Primary key (unique identifier of the record) |
| `recordDate`  | date | The date of the temperature record            |
| `temperature` | int  | The temperature value on that date            |

**Task:**
Find all the `id`s of dates where the **temperature was higher than the previous day’s** temperature.

This means:
For a given date `D₁`, if there exists a date `D₀` such that `D₁ = D₀ + 1` and `temperature(D₁) > temperature(D₀)`, we should return the `id` of the record corresponding to `D₁`.

---

### **2. Core Concept**

The problem revolves around **comparing each day with its previous day**.
We can achieve this by **joining the table to itself** (a *self-join*) — one instance representing the current day (`w1`) and the other representing the previous day (`w2`).

#### Key Operations:

1. **Self-Join Condition:**
   Match records where the date difference between `w1` and `w2` is exactly one day.
2. **Temperature Comparison:**
   Retain only those records where today’s temperature (`w1.temperature`) is greater than yesterday’s (`w2.temperature`).
3. **Projection:**
   Output only the `id` of the warmer day.

---

### **3. Step-by-Step Query Logic**

#### **Step 1: Self-Join**

```sql
FROM Weather w1
JOIN Weather w2
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
```

This creates all possible pairs of consecutive days (where the date difference is exactly one day).

#### **Step 2: Filter Rising Temperatures**

```sql
WHERE w1.temperature > w2.temperature
```

Keeps only the pairs where today’s temperature is higher than yesterday’s.

#### **Step 3: Output Result**

```sql
SELECT w1.id
```

Retrieves the ID of the warmer day.

---

### **4. Dry Run Example**

**Input Table (`Weather`):**

| id | recordDate | temperature |
| -- | ---------- | ----------- |
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |

---

#### **Step 1: Self-Join Result**

| w1.id | w1.recordDate | w1.temp | w2.id | w2.recordDate | w2.temp | DateDiff |
| ----- | ------------- | ------- | ----- | ------------- | ------- | -------- |
| 2     | 2015-01-02    | 25      | 1     | 2015-01-01    | 10      | 1        |
| 3     | 2015-01-03    | 20      | 2     | 2015-01-02    | 25      | 1        |
| 4     | 2015-01-04    | 30      | 3     | 2015-01-03    | 20      | 1        |

---

#### **Step 2: Apply Filter `WHERE w1.temperature > w2.temperature`**

| w1.id | w1.recordDate | w1.temp | w2.recordDate | w2.temp |
| ----- | ------------- | ------- | ------------- | ------- |
| 2     | 2015-01-02    | 25      | 2015-01-01    | 10      |
| 4     | 2015-01-04    | 30      | 2015-01-03    | 20      |

---

#### **Step 3: Final Output**

| id |
| -- |
| 2  |
| 4  |

Hence, the output corresponds to the days where temperature increased compared to the previous day.

---

### **5. SQL Solutions**

#### **Query-1 (Using `DATEDIFF`)**

```sql
# Write your MySQL query statement below
SELECT w1.id
FROM Weather w1
JOIN Weather w2
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

**Explanation:**

* `DATEDIFF(w1.recordDate, w2.recordDate) = 1` ensures consecutive days.
* The comparison `w1.temperature > w2.temperature` filters for rising temperatures.

---

#### **Query-2 (Using `SUBDATE`)**

```sql
# Alternate approach
SELECT w1.id
FROM Weather w1
JOIN Weather w2
ON w1.recordDate = SUBDATE(w2.recordDate, 1)
WHERE w2.temperature > w1.temperature;
```

**Explanation:**

* `SUBDATE(w2.recordDate, 1)` shifts `w2`’s date back by one day.
* This join condition aligns `w1` with the record that occurred exactly one day later.
* The temperature comparison logic is inverted: here, `w2` represents the *later day*, so the condition `w2.temperature > w1.temperature` captures the same logic.

---

### **6. Key Takeaways**

* **Self-Join Usage:**
  When comparing records within the same table (e.g., current vs. previous), self-joins are an effective pattern.

* **Date Difference Handling:**
  Both `DATEDIFF()` and `SUBDATE()` can identify consecutive dates — choose whichever is more intuitive or optimized for your database engine.

* **Output Specificity:**
  We only need to output the IDs of dates meeting the rising temperature condition.

* **Query Equivalence:**
  Both Query-1 and Query-2 produce the same logical result, though their join directions differ slightly.

---
