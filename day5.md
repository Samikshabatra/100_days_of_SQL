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
