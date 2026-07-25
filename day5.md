# Day 5 — Day-After Retention, DISTINCT Counts & Date-Window Filtering

Three problems from LeetCode's Top SQL 50 — one Medium, two Easy.

---

## Problem 1 — Game Play Analysis IV (LeetCode 550)

**Topic:** first-login CTE · `DATE_ADD` day-after self-join · fraction over `DISTINCT` total · **Difficulty:** Medium

### Table: Activity

| Column       | Type |
|--------------|------|
| player_id    | INT  |
| device_id    | INT  |
| event_date   | DATE |
| games_played | INT  |

`(player_id, event_date)` is the primary key. Each row is a player logging in and
playing some games on a given day.

### Task

Report the **fraction** of players that logged in **again on the day right after**
their first login, rounded to 2 decimals. (Players who returned the next day ÷
total players.)

### Solution

```sql
WITH first_login AS (
    SELECT player_id, MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id
)
SELECT ROUND(
         COUNT(DISTINCT a.player_id)
         / (SELECT COUNT(DISTINCT player_id) FROM Activity),
         2
       ) AS fraction
FROM first_login f
JOIN Activity a
  ON f.player_id = a.player_id
 AND a.event_date = DATE_ADD(f.first_login, INTERVAL 1 DAY);
```

### Result

| fraction |
|----------|
| 0.33     |

### Notes

- The `first_login` CTE gets each player's **earliest** `event_date` via
  `MIN(event_date)`.
- The join keeps only rows where the player has activity on
  `DATE_ADD(first_login, INTERVAL 1 DAY)` — i.e. they came back the **very next
  day**. `COUNT(DISTINCT a.player_id)` counts those returning players.
- Denominator is the total distinct players: `(SELECT COUNT(DISTINCT player_id) FROM Activity)`.
- In the example only player 1 returned the next day → `1 / 3 = 0.33`.
- `DATE_ADD(date, INTERVAL 1 DAY)` is the clean way to express "the next calendar
  day" — no manual string math.

---

## Problem 2 — Number of Unique Subjects Taught by Each Teacher (LeetCode 2356)

**Topic:** `COUNT(DISTINCT ...)` · `GROUP BY` · **Difficulty:** Easy

### Table: Teacher

| Column     | Type |
|------------|------|
| teacher_id | INT  |
| subject_id | INT  |
| dept_id    | INT  |

`(subject_id, dept_id)` is the primary key. Each row means the teacher teaches a
subject in a department — so the same subject can repeat across departments.

### Task

Calculate the number of **unique subjects** each teacher teaches. Return
`| teacher_id | cnt |` in any order.

### Solution

```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) AS cnt
FROM Teacher
GROUP BY teacher_id;
```

### Result

| teacher_id | cnt |
|------------|-----|
| 1          | 2   |
| 2          | 4   |

### Notes

- The whole problem is the **`DISTINCT` inside `COUNT`**: a teacher can teach the
  same subject in two departments (two rows), but that's still **one** unique
  subject.
- Teacher 1 has subject 2 (in depts 3 and 4) plus subject 3 → `COUNT(DISTINCT ...) = 2`.
- Plain `COUNT(subject_id)` would wrongly count 3 for teacher 1 — the duplicate
  subject-2 rows would both be counted.

---

## Problem 3 — User Activity for the Past 30 Days I (LeetCode 1141)

**Topic:** date-range `BETWEEN` filter · daily active users via `COUNT(DISTINCT)` · **Difficulty:** Easy

### Table: Activity

| Column        | Type |
|---------------|------|
| user_id       | INT  |
| session_id    | INT  |
| activity_date | DATE |
| activity_type | ENUM |

May have duplicate rows. `activity_type` is one of
`open_session` / `end_session` / `scroll_down` / `send_message`.

### Task

Find the **daily active user count** for the 30-day period **ending 2019-07-27**
(inclusive). A user is active on a day if they made at least one activity that day.
Return `| day | active_users |` in any order.

### Solution

```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
GROUP BY activity_date;
```

### Result

| day        | active_users |
|------------|--------------|
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |

### Notes

- The 30-day window ending **2019-07-27** inclusive starts at **2019-06-28**
  (27 − 30 + 1). `BETWEEN` is inclusive on both ends, which is exactly what's
  needed.
- `COUNT(DISTINCT user_id)` is essential — the table has multiple rows per user
  per day (open/scroll/end), but each user counts **once** toward that day's
  active count.
- Rows outside the window (e.g. 2019-06-25) are dropped by the `WHERE`, so they
  never reach the group.
