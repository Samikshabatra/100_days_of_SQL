# Day 1 — SELECT / WHERE / ORDER BY + JOINS

Two problems from LeetCode's Top SQL 50 study plan.

---

## Problem 1 — 620. Not Boring Movies

**Source:** LeetCode (Top SQL 50) · **Difficulty:** Easy
**Link:** https://leetcode.com/problems/not-boring-movies/

### Table: Cinema

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| movie       | varchar |
| description | varchar |
| rating      | float   |

`id` is the primary key. `rating` is a float in range [0, 10].

### Task

Report the movies with an odd-numbered `id` and a `description` that is not
`"boring"`. Order the result by `rating` in descending order.

### Solution

```sql
SELECT *
FROM Cinema
WHERE id % 2 != 0 AND description != "boring"
ORDER BY rating DESC;
```

### Notes

- Accepted on first try, 10/10 testcases passed. Runtime 262ms (beats 79.97%).
- Key idea: `id % 2 != 0` filters odd IDs, combined with a simple inequality
  filter on `description`. No joins/subqueries needed — good warm-up.

---

## Problem 2 — 1251. Average Selling Price

**Source:** LeetCode (Top SQL 50) · **Difficulty:** Easy
**Link:** https://leetcode.com/problems/average-selling-price/

### Table: Prices

| Column Name | Type |
|-------------|------|
| product_id  | int  |
| start_date  | date |
| end_date    | date |
| price       | int  |

`(product_id, start_date, end_date)` is the primary key. Each row gives the
price of `product_id` for the period `start_date` to `end_date`. No two periods
for the same product overlap.

### Table: UnitsSold

| Column Name   | Type |
|---------------|------|
| product_id    | int  |
| purchase_date | date |
| units         | int  |

May contain duplicate rows.

### Task

Find the average selling price for each product, rounded to 2 decimal places.
If a product has no units sold, its average price is assumed to be 0.

### Solution

```sql
SELECT p.product_id,
       ROUND(IFNULL(SUM(p.price * u.units) / SUM(u.units), 0), 2) AS average_price
FROM Prices p
LEFT JOIN UnitsSold u
       ON p.product_id = u.product_id
      AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

### Notes

- Accepted, 18/18 testcases passed. Runtime 848ms (beats 48.32%).
- `LEFT JOIN` (not `INNER`) is essential — products with zero units sold still
  need to appear, with `average_price = 0` via `IFNULL`.
- The join condition isn't just `product_id` — `purchase_date` must fall
  `BETWEEN` the price's `start_date` and `end_date` to apply the right price.
- Weighted average = `SUM(price * units) / SUM(units)`, not a plain `AVG(price)`.
