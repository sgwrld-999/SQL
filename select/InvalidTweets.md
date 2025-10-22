### **Problem Name: Invalid Tweets**

**Problem Link:** [LeetCode](https://leetcode.com/problems/invalid-tweets/)

-----

### **1. Problem Understanding** 

We have a `Tweets` table containing a `tweet_id` and the `content` of the tweet.

**Task:**
The goal is to find the `tweet_id` for all "invalid" tweets. A tweet is defined as invalid if the number of characters in its content is **strictly greater than 15**.

**Example:**
*Input:*
`Tweets` table:

```
+----------+----------------------------------+
| tweet_id | content                          |
+----------+----------------------------------+
| 1        | Vote for Biden                   |
| 2        | Let us make America great again! |
+----------+----------------------------------+
```

*Output:*

```
+----------+
| tweet_id |
+----------+
| 2        |
+----------+
```

*Explanation:* The content of tweet 2 has a length of 32 characters, which is greater than 15, so it's invalid. Tweet 1 has a length of 14, which is not greater than 15.

-----

### **2. The Core Logic: Using Functions in Filters**

The key to this problem is realizing that a `WHERE` clause can do more than just compare column values directly. You can apply **functions** to a column's value and then use the result of that function in your comparison.

In this case, the condition isn't about the text of the tweet itself, but about a *property* of the text—its length. SQL provides the `LENGTH()` function (or `CHAR_LENGTH()` in some dialects) to calculate the number of characters in a string.

The logic is simple: for each row, the query first calculates `LENGTH(content)`, gets a number, and then checks if that number is `> 15`.

-----

### **3. Constructing the Query (Step-by-Step)**

1.  **`SELECT tweet_id`**: We start by specifying that we only need the `tweet_id` in our result.
2.  **`FROM Tweets`**: We identify the `Tweets` table as our data source.
3.  **`WHERE LENGTH(content) > 15`**: This is the crucial filtering step. For each row, the database:
      * Takes the value from the `content` column.
      * Applies the `LENGTH()` function to it.
      * Compares the resulting number to 15.
      * Keeps the row only if the length is strictly greater than 15.

-----

### **4. Dry Run Example**

Let's trace the logic on the example table.
**Condition:** `LENGTH(content) > 15`

| tweet\_id | content | `LENGTH(content)` | `LENGTH(content) > 15`? | Keep/Discard |
| :--- | :--- | :--- | :--- | :--- |
| **1** | 'Vote for Biden' | 14 | 14 \> 15 → FALSE | **Discard** |
| **2** | 'Let us make America great again\!' | 32 | 32 \> 15 → TRUE | **Keep** |

The query correctly keeps only `tweet_id` 2.

-----

### **5. SQL Query**

```sql
SELECT
    tweet_id
FROM
    Tweets
WHERE
    LENGTH(content) > 15;
```

-----

### **6. Key Takeaways** 

  * **Functions in `WHERE`**: Don't forget that you can use built-in SQL functions (for strings, dates, numbers, etc.) inside a `WHERE` clause to create much more powerful and flexible filters.
  * **String Manipulation**: `LENGTH()` is one of the most common string functions, essential for validating data based on size constraints.
  * **Translating Rules to Code**: This problem is a great example of translating a simple business rule ("tweets must not be longer than 15 characters") into a concise line of SQL.