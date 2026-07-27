# Day 7 — Singles via HAVING & "Bought Everything" with a Count Match

Two problems from LeetCode's Top SQL 50 — one Easy, one Medium.

---

## Problem 1 — Biggest Single Number (LeetCode 619)

**Topic:** `GROUP BY` + `HAVING COUNT(*) = 1` · outer `MAX()` for the NULL case · **Difficulty:** Easy

### Table: MyNumbers

| Column | Type |
|--------|------|
| num    | INT  |

May contain duplicates (no primary key). Each row holds an integer.

### Task

A **single number** appears exactly once in `MyNumbers`. Find the **largest**
single number. If there is none, report `null`.

### Solution

```sql
SELECT MAX(num) AS num
FROM (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
) AS t;
```

### Result

| num |
|-----|
| 6   |

### Notes

- The inner query keeps only values that occur **once**: `GROUP BY num HAVING COUNT(*) = 1`.
- Wrapping it in `MAX(...)` does double duty: it returns the biggest single **and**
  handles the "no singles" case cleanly — `MAX` over zero rows yields `NULL`, which
  is exactly the required output.
- If you selected `num ... ORDER BY num DESC LIMIT 1` instead, an empty set would
  return **no row** rather than a `NULL` row — so the `MAX` wrapper is the safer
  pattern for this problem.
