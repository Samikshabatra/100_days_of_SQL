# Day 3 — Aggregation, JOIN + GROUP BY & Conditional Averages

Three problems from LeetCode's **Top SQL 50** study plan.

---

## Problem 1 — Project Employees I (LeetCode 1075)

**Topic:** `JOIN` + `GROUP BY` + `AVG()` · `ROUND()` · **Difficulty:** Easy

### Tables

**Project**

| Column      | Type |
|-------------|------|
| project_id  | INT  |
| employee_id | INT  |

`(project_id, employee_id)` is the primary key. `employee_id` is a foreign key to
`Employee`.

**Employee**

| Column           | Type    |
|------------------|---------|
| employee_id      | INT     |
| name             | VARCHAR |
| experience_years | INT     |

`employee_id` is the primary key; `experience_years` is never NULL.

### Task

Report the **average experience years** of all employees on each project, rounded
to 2 digits. Return `| project_id | average_years |` in any order.

### Solution

```sql
SELECT project_id, ROUND(AVG(experience_years), 2) AS average_years
FROM Project p
JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY project_id;
```

### Result

| project_id | average_years |
|------------|---------------|
| 1          | 2.00          |
| 2          | 2.50          |

### Notes

- Project 1 has employees 1, 2, 3 → `(3 + 2 + 1) / 3 = 2.00`; project 2 has
  employees 1, 4 → `(3 + 2) / 2 = 2.50`.
- The `JOIN` pulls each project row's `experience_years` from `Employee`, then
  `GROUP BY project_id` collapses them so `AVG()` runs per project.
- `ROUND(..., 2)` is required — the problem asks for 2 decimal digits.

---

## Problem 2 — Percentage of Users Attended a Contest (LeetCode 1633)

**Topic:** Scalar subquery · percentage · `GROUP BY` + `ORDER BY` tiebreak · **Difficulty:** Easy

### Tables

**Users**

| Column    | Type    |
|-----------|---------|
| user_id   | INT     |
| user_name | VARCHAR |

`user_id` is the primary key.

**Register**

| Column     | Type |
|------------|------|
| contest_id | INT  |
| user_id    | INT  |

`(contest_id, user_id)` is the primary key — each row is a user registered for a
contest.

### Task

Find the **percentage of all users** registered in each contest, rounded to 2
decimals. Order by `percentage DESC`, breaking ties by `contest_id ASC`.

### Solution

```sql
SELECT contest_id,
       ROUND(COUNT(r.user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) AS percentage
FROM Users u
JOIN Register r ON u.user_id = r.user_id
GROUP BY contest_id
ORDER BY percentage DESC, contest_id ASC;
```

### Result

| contest_id | percentage |
|------------|------------|
| 208        | 100.00     |
| 209        | 100.00     |
| 210        | 66.67      |
| 215        | 66.67      |

### Notes

- The denominator is the **total** number of users, so it's a scalar subquery
  `(SELECT COUNT(*) FROM Users)` — a constant, not affected by the `GROUP BY`.
- `* 100.0` (not `* 100`) forces floating-point division; with integers MySQL
  would truncate before `ROUND` could help.
- `COUNT(r.user_id)` per group counts how many users registered for that contest;
  dividing by 3 total users gives the percentage.
- The tie between 208 and 209 (both 100.00) is resolved by `contest_id ASC`, which
  is exactly why the `ORDER BY` has a second key.
