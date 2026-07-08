# Day 2 — Window Functions, Self-Join & Running Totals

Three practice problems from my SQL learning path.

---

## Problem 1 — Performance Bucketing with NTILE()

**Topic:** `NTILE()` — bucketing rows into equal-sized tiers · **Difficulty:** Medium

### Scenario

You're a data analyst at a sales company. HR wants to bucket sales reps into
**3 performance tiers** based on their total sales amount.

### Table: sales_reps

| Column      | Type         |
|-------------|--------------|
| rep_id      | INT          |
| rep_name    | VARCHAR(50)  |
| region      | VARCHAR(50)  |
| total_sales | DECIMAL      |

### Task

Bucket all reps into 3 tiers based on `total_sales DESC` (highest sales = Tier 1).
Return `| rep_id | rep_name | total_sales | tier | performance_level |`, where
`performance_level` is `'Top Performer'` (Tier 1), `'Mid Performer'` (Tier 2),
`'Low Performer'` (Tier 3). Order by `total_sales DESC`.

### Solution

```sql
WITH sales_bucketed AS (
    SELECT rep_id, rep_name, total_sales,
           NTILE(3) OVER (ORDER BY total_sales DESC) AS tier
    FROM sales_reps
)
SELECT rep_id, rep_name, total_sales, tier,
       CASE WHEN tier = 1 THEN 'Top Performer'
            WHEN tier = 2 THEN 'Mid Performer'
            WHEN tier = 3 THEN 'Low Performer'
       END AS performance_level
FROM sales_bucketed
ORDER BY total_sales DESC;
```

### Result (5 reps)

| rep_id | rep_name | total_sales | tier | performance_level |
|--------|----------|-------------|------|-------------------|
| 2      | Samiksha | 280000      | 1    | Top Performer     |
| 1      | Sam      | 180000      | 1    | Top Performer     |
| 4      | Samarth  | 120000      | 2    | Mid Performer     |
| 3      | Samantha | 80000       | 2    | Mid Performer     |
| 5      | Sami     | 10000       | 3    | Low Performer     |

### Notes

- `NTILE(3) OVER (ORDER BY total_sales DESC)` splits rows into 3 buckets that are
  as equal as possible. With 5 reps it goes 2 / 2 / 1 — extra rows land in the
  earlier buckets.
- Had to wrap it in a **CTE**: you can't reference the `tier` window alias inside
  a `CASE` in the same SELECT level, so compute `tier` first, then label it.
- The window's `ORDER BY` controls bucketing; the outer `ORDER BY` controls the
  display order.

---

## Problem 2 — Employees Outperforming Their Manager (Self-Join)

**Topic:** Self-join on a table that references itself · **Difficulty:** Medium

### Scenario

You're a data analyst at a tech company. The team lead wants to identify
employees who **joined after their manager** but **earn more than their manager**.

### Table: employees

| Column      | Type         |
|-------------|--------------|
| employee_id | INT          |
| name        | VARCHAR(50)  |
| salary      | DECIMAL      |
| hire_date   | DATE         |
| manager_id  | INT          |

`manager_id` references `employee_id` in the same table.

### Task

Return all employees who were hired **after** their manager
(`hire_date > manager's hire_date`) **and** earn **more** than their manager
(`salary > manager's salary`). Return
`| employee_name | employee_salary | employee_hire_date | manager_name | manager_salary | manager_hire_date |`,
ordered by `employee_salary DESC`.

### Solution

```sql
SELECT e.name, e.salary, e.hire_date,
       m.name, m.salary, m.hire_date
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.hire_date > m.hire_date
  AND e.salary > m.salary
ORDER BY e.salary DESC;
```

### Result

| employee_name | employee_salary | employee_hire_date | manager_name | manager_salary | manager_hire_date |
|---------------|-----------------|--------------------|--------------|----------------|-------------------|
| Samantha      | 280000          | 2026-12-01         | Sam          | 200000         | 2026-09-02        |
| Samarth       | 260000          | 2026-10-01         | Samiksha     | 250000         | 2026-09-01        |

### Notes

- The manager lives in the **same** `employees` table → self-join: alias `e`
  (employee) and `m` (manager), joined on `e.manager_id = m.employee_id`.
- **What I got wrong first:** wrote `JOIN manager m ...` and got
  `Error 1146. Table 'company.manager' doesn't exist`. There is no `manager`
  table — the manager is another row in `employees`. Fixed to `JOIN employees m`.
- The `INNER JOIN` automatically drops the CEO (`manager_id = NULL`), since NULL
  matches no `employee_id`.

---

## Problem 3 — Monthly Deposit Report with Running Total

**Topic:** `COALESCE` · `DATE_FORMAT` · `GROUP BY` + cumulative window `SUM()` · **Difficulty:** Medium

### Scenario

You're a data analyst at a bank. The finance team wants a monthly deposit report
per branch, with a **running total** of deposits. Some branches have missing
region data.

### Table: deposits

| Column       | Type         |
|--------------|--------------|
| deposit_id   | INT          |
| branch_id    | INT          |
| region       | VARCHAR(50)  |
| deposit_date | DATE         |
| amount       | DECIMAL      |

`region` can be `NULL` for some branches.

### Task

Return per branch per month
`| branch_id | region | deposit_month | monthly_total | running_total |`, where:
- `region` — actual region if available, else `'Unassigned'`
- `deposit_month` — `DATE_FORMAT(deposit_date, '%Y-%m')`
- `monthly_total` — `SUM(amount)` per branch per month
- `running_total` — cumulative `SUM` of `monthly_total` per branch ordered by
  `deposit_month ASC`

Order by `branch_id ASC, deposit_month ASC`.

### Solution

```sql
WITH cte AS (
    SELECT branch_id,
           COALESCE(region, 'Unassigned') AS region,
           DATE_FORMAT(deposit_date, '%Y-%m') AS deposit_month,
           SUM(amount) AS monthly_total
    FROM deposits
    GROUP BY branch_id, region, DATE_FORMAT(deposit_date, '%Y-%m')
)
SELECT branch_id, region, deposit_month, monthly_total,
       SUM(monthly_total) OVER (PARTITION BY branch_id
                                ORDER BY deposit_month ASC) AS running_total
FROM cte
ORDER BY branch_id ASC, deposit_month ASC;
```

### Result

| branch_id | region    | deposit_month | monthly_total | running_total |
|-----------|-----------|---------------|---------------|---------------|
| 101       | Bengaluru | 2026-09       | 180000        | 649000        |
| 101       | Sonipat   | 2026-09       | 380000        | 649000        |
| 101       | Panipat   | 2026-09       | 89000         | 649000        |
| 102       | Delhi     | 2026-10       | 280000        | 360000        |
| 102       | Haryana   | 2026-10       | 80000         | 360000        |

### Notes

- `COALESCE(region, 'Unassigned')` replaces `NULL` regions with a label so they
  still show up in the report.
- `DATE_FORMAT(deposit_date, '%Y-%m')` buckets each deposit into a year-month.
- Two-step pattern: the **CTE** does the `GROUP BY` to get `monthly_total`, then
  the outer `SUM() OVER (PARTITION BY branch_id ORDER BY deposit_month)` adds the
  cumulative `running_total` **without collapsing** the rows.
- Quirk worth noting: `GROUP BY` includes `region`, so a branch-month can span
  several rows (branch 101 has Bengaluru/Sonipat/Panipat all in `2026-09`). With
  `ORDER BY deposit_month` and the default `RANGE` frame, every row sharing the
  same month gets the **same** running total (all peers summed together) — that's
  why 101's three September rows all read `649000`. For a strictly row-by-row
  running total you'd add a unique tiebreaker to `ORDER BY` or use a `ROWS` frame.
