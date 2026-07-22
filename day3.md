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
