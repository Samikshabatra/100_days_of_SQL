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
