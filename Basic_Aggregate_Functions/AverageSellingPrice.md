## 1. Problem Name: Average Selling Price

Problem: LeetCode 1251. Average Selling Price

We are given two tables:

### Prices

Records the active price of each product for specific date ranges.

| Column     | Description                                    |
| ---------- | ---------------------------------------------- |
| product_id | Product identifier                             |
| start_date | Date when this price became active (inclusive) |
| end_date   | Date when this price stopped being active      |
| price      | Price per unit during that date range          |

### UnitsSold

Records how many units of each product were sold on a given date.

| Column        | Description                       |
| ------------- | --------------------------------- |
| product_id    | Product identifier                |
| purchase_date | Date of sale                      |
| units         | Number of units sold on that date |

---

## 2. Goal

For each product, compute its average selling price over the observed sales.

Definition:

average_price = SUM(price × units) / SUM(units)

Where:

* The correct price must be chosen based on the purchase date.
* If a product has no sales at all, its average price should be reported as 0.00.
* The result must be rounded to 2 decimal places.
* Output should include:

  * product_id
  * average_price

---

## 3. Core Logic

The key difficulty is that prices can change over time. So you cannot just join on product_id alone. You must match each sale in UnitsSold to the specific active price interval for that product.

That means:

* For a row in UnitsSold with (product_id = X, purchase_date = D)
* We must find the row in Prices where:

  * product_id = X
  * start_date ≤ D ≤ end_date

Once each sale is matched with its valid price, we can compute total revenue and total units sold per product.

Finally, average_price is total_revenue / total_units.

---

## 4. Query Breakdown

### Step 1. Join Prices to UnitsSold using both product_id and date range

```sql
FROM Prices p
LEFT JOIN UnitsSold us
ON p.product_id = us.product_id
AND us.purchase_date BETWEEN p.start_date AND p.end_date
```

Why LEFT JOIN:

* We want to include all products from Prices even if there were no matching sales in UnitsSold.
* If there are no sales for a product, the joined UnitsSold columns will be NULL.

Why the date condition:

* Ensures that each sale is matched to the correct price at the time of that sale.

---

### Step 2. Compute weighted revenue and total units

* `p.price * us.units` gives revenue for that sale row.
* `SUM(p.price * us.units)` gives total revenue for the product.
* `SUM(us.units)` gives total units sold.

Then average_price = total_revenue / total_units.

---

### Step 3. Handle products with no sales

If a product never sold any units:

* `SUM(us.units)` will be NULL.
* Division would produce NULL.
  So we wrap the division in `IFNULL(..., 0)`.

---

### Step 4. Round to two decimal places

Use `ROUND(value, 2)` for final formatting.

---

## 5. Final SQL Query

```sql
SELECT 
    p.product_id,
    IFNULL(
        ROUND(SUM(p.price * us.units) / SUM(us.units), 2),
        0.00
    ) AS average_price
FROM 
    Prices p
LEFT JOIN 
    UnitsSold us
ON 
    p.product_id = us.product_id
    AND us.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY 
    p.product_id;
```

---

## 6. Dry Run Example

Suppose the data is:

Prices:

| product_id | start_date | end_date   | price |
| ---------- | ---------- | ---------- | ----- |
| 1          | 2019-02-17 | 2019-02-28 | 5     |
| 1          | 2019-03-01 | 2019-03-22 | 20    |
| 2          | 2019-02-01 | 2019-02-20 | 15    |

UnitsSold:

| product_id | purchase_date | units |
| ---------- | ------------- | ----- |
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |

After applying the LEFT JOIN with the date condition:

| p.product_id | p.price | us.purchase_date | us.units | matched?                                        |
| ------------ | ------- | ---------------- | -------- | ----------------------------------------------- |
| 1            | 5       | 2019-02-25       | 100      | 2019-02-25 is between 2019-02-17 and 2019-02-28 |
| 1            | 20      | 2019-03-01       | 15       | 2019-03-01 is between 2019-03-01 and 2019-03-22 |
| 2            | 15      | 2019-02-10       | 200      | 2019-02-10 is between 2019-02-01 and 2019-02-20 |

Now compute revenue per joined row:

For product 1:

* Row 1: price 5 × 100 units = 500
* Row 2: price 20 × 15 units = 300
  Total revenue = 800
  Total units = 100 + 15 = 115
  Average price = 800 / 115 = 6.956521... → 6.96 after rounding

For product 2:

* Row: price 15 × 200 units = 3000
  Total revenue = 3000
  Total units = 200
  Average price = 3000 / 200 = 15.00

Final aggregated result:

| product_id | average_price |
| ---------- | ------------- |
| 1          | 6.96          |
| 2          | 15.00         |

If a product existed in Prices but had no matching rows in UnitsSold:

* The revenue terms would all be NULL.
* The denominator SUM(us.units) would also be NULL.
* The division would return NULL.
* `IFNULL(..., 0.00)` converts that to 0.00.

---

## 7. Alternative Approach Using a Subquery First

Another valid way to solve this is to:

1. First match sales to prices in a derived table.
2. Then aggregate over that derived table.

Example:

```sql
SELECT
    product_id,
    IFNULL(
        ROUND(SUM(price * units) / SUM(units), 2),
        0.00
    ) AS average_price
FROM (
    SELECT 
        p.product_id,
        p.price,
        us.units
    FROM Prices p
    LEFT JOIN UnitsSold us
    ON p.product_id = us.product_id
    AND us.purchase_date BETWEEN p.start_date AND p.end_date
) AS matched
GROUP BY product_id;
```

Notes:

* The inner query handles the date alignment and brings price and units together.
* The outer query just performs the revenue/units aggregation per product.
* This structure can improve readability in more complex analytics queries because it separates the matching logic from the math.

---

## 8. Key Takeaways

1. This problem is a date-range join problem, not just a simple equality join.
2. You must associate each sale with the valid price in effect on that purchase date.
3. The correct aggregation is weighted average price:
   SUM(price × units) / SUM(units), grouped by product.
4. Use LEFT JOIN to avoid dropping products that had no sales.
5. Use IFNULL to turn NULL division results into 0.00.
6. Apply rounding at the end, not on partial values.