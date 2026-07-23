# Day 4 — Conditional Aggregation & First-Order Filtering

Two **Medium** problems from LeetCode's Top SQL 50.

---

## Problem 1 — Monthly Transactions I (LeetCode 1193)

**Topic:** `DATE_FORMAT` bucketing · conditional aggregation (`CASE` / boolean `SUM`) · `GROUP BY` on two keys · **Difficulty:** Medium

### Table: Transactions

| Column     | Type    |
|------------|---------|
| id         | INT     |
| country    | VARCHAR |
| state      | ENUM    |
| amount     | INT     |
| trans_date | DATE    |

`id` is the primary key. `state` is an enum of `"approved"` / `"declined"`.

### Task

For each **month and country**, report the number of transactions and their total
amount, plus the number of **approved** transactions and their total amount.
Return `| month | country | trans_count | trans_total_amount | approved_count | approved_total_amount |`
in any order.

### Solution

```sql
SELECT DATE_FORMAT(trans_date, '%Y-%m') AS month,
       country,
       COUNT(*) AS trans_count,
       SUM(amount) AS trans_total_amount,
       SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount,
       SUM(state = 'approved') AS approved_count
FROM Transactions
GROUP BY DATE_FORMAT(trans_date, '%Y-%m'), country;
```

### Result

| month   | country | trans_count | trans_total_amount | approved_total_amount | approved_count |
|---------|---------|-------------|--------------------|-----------------------|----------------|
| 2018-12 | US      | 2           | 3000               | 1000                  | 1              |
| 2019-01 | US      | 1           | 2000               | 2000                  | 1              |
| 2019-01 | DE      | 1           | 2000               | 2000                  | 1              |

### Notes

- `DATE_FORMAT(trans_date, '%Y-%m')` buckets each transaction into a year-month;
  it goes in **both** the `SELECT` and the `GROUP BY` (grouping per month + country).
- Two ways to do the "approved-only" math, both used here:
  - **Sum with CASE** → `SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END)`
    adds up only approved amounts.
  - **Boolean SUM** → `SUM(state = 'approved')`. In MySQL the condition is `1`/`0`,
    so summing it *counts* the approved rows — shorter than `COUNT(CASE WHEN ...)`.
- `COUNT(*)` / `SUM(amount)` cover **all** rows in the group (approved + declined).

---

## Problem 2 — Immediate Food Delivery II (LeetCode 1174)

**Topic:** first-order filtering via `MIN()` subquery · conditional percentage · `JOIN` on two keys · **Difficulty:** Medium

### Table: Delivery

| Column                     | Type |
|----------------------------|------|
| delivery_id                | INT  |
| customer_id                | INT  |
| order_date                 | DATE |
| customer_pref_delivery_date| DATE |

`delivery_id` is the primary key. An order is **immediate** if
`order_date = customer_pref_delivery_date`, else **scheduled**. A customer's
**first order** is their earliest `order_date` (exactly one per customer).

### Task

Find the percentage of **immediate** orders among the **first orders** of all
customers, rounded to 2 decimals.

### Solution

```sql
SELECT ROUND(
         SUM(CASE WHEN d.order_date = d.customer_pref_delivery_date THEN 1 ELSE 0 END)
         * 100.0 / COUNT(*), 2
       ) AS immediate_percentage
FROM Delivery d
JOIN (
    SELECT customer_id, MIN(order_date) AS first_order
    FROM Delivery
    GROUP BY customer_id
) f
  ON d.customer_id = f.customer_id
 AND d.order_date  = f.first_order;
```

### Result

| immediate_percentage |
|----------------------|
| 33.33                |

### Notes

- The subquery `f` finds each customer's **first order** (`MIN(order_date)`); the
  `JOIN ... ON d.customer_id = f.customer_id AND d.order_date = f.first_order`
  keeps only those first-order rows.
- Then over just those rows: `SUM(CASE WHEN order_date = pref_date THEN 1 ELSE 0 END)`
  counts the immediate ones, `/ COUNT(*)` turns it into a fraction, `* 100.0`
  (float, to avoid integer truncation) makes it a percentage.
- In the example, only customer 2's first order is immediate → `1 / 3 = 33.33`.
- Joining on the **date** as well as the customer is what restricts the average to
  first orders — without it you'd be averaging over every delivery.
