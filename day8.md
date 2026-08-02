# Day 8 — Self-Join Report Counts & Primary Department Logic

Two Easy problems from LeetCode's Top SQL 50.

---

## Problem 1 — The Number of Employees Which Report to Each Employee (LeetCode 1731)

**Topic:** self-join on `reports_to` · `COUNT` + `ROUND(AVG())` · `GROUP BY` + `ORDER BY` · **Difficulty:** Easy

### Table: Employees

| Column      | Type    |
|-------------|---------|
| employee_id | INT     |
| name        | VARCHAR |
| reports_to  | INT     |
| age         | INT     |

`employee_id` is unique. `reports_to` is the manager's id (NULL if none).

### Task

A **manager** is anyone with at least 1 direct report. Report each manager's id,
name, number of **direct** reports, and the average age of those reports rounded to
the nearest integer. Return `| employee_id | name | reports_count | average_age |`
ordered by `employee_id`.

### Solution

```sql
SELECT m.employee_id, m.name,
       COUNT(e.employee_id) AS reports_count,
       ROUND(AVG(e.age)) AS average_age
FROM Employees m
JOIN Employees e ON m.employee_id = e.reports_to
GROUP BY m.employee_id, m.name
ORDER BY m.employee_id;
```

### Result

| employee_id | name  | reports_count | average_age |
|-------------|-------|---------------|-------------|
| 9           | Hercy | 2             | 39          |

### Notes

- Self-join: `m` = manager, `e` = report, joined on `m.employee_id = e.reports_to`.
  The `INNER JOIN` drops anyone with zero reports (non-managers) automatically.
- `COUNT(e.employee_id)` = number of direct reports per manager.
- `ROUND(AVG(e.age))` rounds to nearest integer — Hercy's reports are 41 and 36,
  avg 38.5, rounds to **39**.
- Must group by both `m.employee_id` and `m.name` since both are non-aggregated in
  the `SELECT`.

---

## Problem 2 — Primary Department for Each Employee (LeetCode 1789)

**Topic:** `OR` with a `HAVING COUNT = 1` subquery for the single-department case · **Difficulty:** Easy

### Table: Employee

| Column        | Type    |
|---------------|---------|
| employee_id   | INT     |
| department_id | INT     |
| primary_flag  | VARCHAR |

`(employee_id, department_id)` is the primary key. `primary_flag` is `'Y'`/`'N'`.
An employee in only one department has `primary_flag = 'N'`.

### Task

Report every employee with their **primary department**. Employees in multiple
departments have exactly one `'Y'`; employees in a single department report that
one department (even though its flag is `'N'`). Any order.

### Solution

```sql
SELECT employee_id, department_id
FROM Employee
WHERE primary_flag = 'Y'
   OR employee_id IN (
        SELECT employee_id
        FROM Employee
        GROUP BY employee_id
        HAVING COUNT(*) = 1
   );
```

### Result

| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 1             |
| 3           | 3             |
| 4           | 3             |

### Notes

- Two cases, joined with `OR`:
  1. `primary_flag = 'Y'` — the marked primary for multi-department employees.
  2. `employee_id IN (... HAVING COUNT(*) = 1)` — employees who appear in exactly
     one row, i.e. belong to a single department (whose flag is `'N'`).
- The subquery finds the single-department employees by counting their rows per
  `employee_id` and keeping those with exactly one.
- Employee 1 has only dept 1 (flag N) so the subquery catches it; employees 2 and 4
  are caught by their `'Y'` flag.
