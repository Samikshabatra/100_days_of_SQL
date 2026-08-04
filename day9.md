# Day 9 — CASE Expressions & Consecutive Rows via Self-Join

Two problems from LeetCode's Top SQL 50 — one Easy, one Medium.

---

## Problem 1 — Triangle Judgement (LeetCode 610)

**Topic:** `CASE WHEN` for a per-row label · triangle inequality · **Difficulty:** Easy

### Table: Triangle

| Column | Type |
|--------|------|
| x      | INT  |
| y      | INT  |
| z      | INT  |

`(x, y, z)` is the primary key. Each row holds the lengths of three line segments.

### Task

For every three segments, report whether they can form a triangle. Return
`| x | y | z | triangle |` (`'Yes'`/`'No'`), any order.

### Solution

```sql
SELECT x, y, z,
       CASE WHEN x + y > z AND y + z > x AND x + z > y
            THEN 'Yes' ELSE 'No' END AS triangle
FROM Triangle;
```

### Result

| x  | y  | z  | triangle |
|----|----|----|----------|
| 13 | 15 | 30 | No       |
| 10 | 20 | 15 | Yes      |

### Notes

- Triangle inequality: three sides form a triangle only when **every** pair sums to
  more than the third side — all three conditions must hold.
- `CASE WHEN ... THEN 'Yes' ELSE 'No' END` labels each row; no `GROUP BY` since it's
  a per-row computation.
- `13, 15, 30` fails because `13 + 15 = 28 < 30`.
