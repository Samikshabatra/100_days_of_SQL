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

---

## Problem 2 — Consecutive Numbers (LeetCode 180)

**Topic:** triple self-join on `id + 1` / `id + 2` · consecutive-run detection · **Difficulty:** Medium

### Table: Logs

| Column | Type    |
|--------|---------|
| id     | INT     |
| num    | VARCHAR |

`id` is the primary key, an autoincrement starting at 1 (so rows are contiguous).

### Task

Find all numbers that appear **at least three times consecutively**. Return
`| ConsecutiveNums |`, any order.

### Solution

```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l2.id = l1.id + 1
JOIN Logs l3 ON l3.id = l1.id + 2
WHERE l1.num = l2.num
  AND l2.num = l3.num;
```

### Result

| ConsecutiveNums |
|-----------------|
| 1               |

### Notes

- Because `id` is contiguous, "consecutive rows" = `id`, `id+1`, `id+2`. Join the
  table to itself three times, aligned to those offsets.
- The `WHERE` forces all three aligned rows to share the same `num` — that's a run
  of three in a row.
- `DISTINCT` because a number appearing four+ times in a row would match from
  multiple starting positions, producing duplicates.
