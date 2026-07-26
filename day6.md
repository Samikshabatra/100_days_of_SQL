# Day 6 — First-Year Filtering, HAVING & Per-Group Counts

Three problems from LeetCode's Top SQL 50 — one Medium, two Easy.

---

## Problem 1 — Product Sales Analysis III (LeetCode 1070)

**Topic:** per-group `MIN()` subquery · join back on group key + value · **Difficulty:** Medium

### Table: Sales

| Column     | Type |
|------------|------|
| sale_id    | INT  |
| product_id | INT  |
| year       | INT  |
| quantity   | INT  |
| price      | INT  |

`(sale_id, year)` is the primary key. A product may have multiple sales entries in
the same year. `price` is per-unit.

### Task

Find all sales that happened in the **first year** each product was sold. For each
`product_id`, identify its earliest `year`, then return **all** sales entries for
that product in that year. Return `| product_id | first_year | quantity | price |`
in any order.

### Solution

```sql
SELECT s.product_id, s.year AS first_year, s.quantity, s.price
FROM Sales s
JOIN (
    SELECT product_id, MIN(year) AS first_year
    FROM Sales
    GROUP BY product_id
) t
  ON s.product_id = t.product_id
 AND s.year = t.first_year;
```

### Result

| product_id | first_year | quantity | price |
|------------|------------|----------|-------|
| 100        | 2008       | 10       | 5000  |
| 200        | 2011       | 15       | 9000  |

### Notes

- The subquery `t` gets each product's **earliest** year (`MIN(year)`).
- Joining on **both** `product_id` **and** `year = first_year` keeps every original
  row from that first year — so if a product had two sales in its first year, both
  come back (the task says return *all* entries).
- You can't just `SELECT ... MIN(year), quantity` in one grouped query —
  `quantity`/`price` aren't functionally tied to `MIN(year)`. The self-join back to
  `Sales` is what recovers the real rows.

---

## Problem 2 — Classes With at Least 5 Students (LeetCode 596)

**Topic:** `GROUP BY` + `HAVING` on an aggregate · **Difficulty:** Easy

### Table: Courses

| Column  | Type    |
|---------|---------|
| student | VARCHAR |
| class   | VARCHAR |

`(student, class)` is the primary key — each row is a student enrolled in a class.

### Task

Find all classes that have **at least 5 students**. Return `| class |` in any order.

### Solution

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(student) >= 5;
```

### Result

| class |
|-------|
| Math  |

### Notes

- `HAVING` filters **after** grouping — it's the aggregate counterpart of `WHERE`.
  You can't use `WHERE COUNT(student) >= 5` because `WHERE` runs before the rows are
  grouped and aggregates don't exist yet.
- The primary key `(student, class)` guarantees no duplicate student-in-class rows,
  so `COUNT(student)` is a clean head-count per class.
- In the example only `Math` has ≥ 5 distinct students.

---

## Problem 3 — Find Followers Count (LeetCode 1729)

**Topic:** `COUNT()` per group · `GROUP BY` + `ORDER BY` · **Difficulty:** Easy

### Table: Followers

| Column      | Type |
|-------------|------|
| user_id     | INT  |
| follower_id | INT  |

`(user_id, follower_id)` is the primary key. Each row means `follower_id` follows
`user_id`.

### Task

For each user, return the number of followers. Return `| user_id | followers_count |`
ordered by `user_id` ascending.

### Solution

```sql
SELECT user_id, COUNT(follower_id) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

### Result

| user_id | followers_count |
|---------|-----------------|
| 0       | 1               |
| 1       | 1               |
| 2       | 2               |

### Notes

- Straight `GROUP BY user_id` with `COUNT(follower_id)` — each group is one user,
  and the count is how many rows (followers) they have.
- No `DISTINCT` needed: the primary key `(user_id, follower_id)` already forbids the
  same follower appearing twice for a user.
- `ORDER BY user_id` is required by the prompt (ascending) — grouping alone doesn't
  guarantee sorted output.
