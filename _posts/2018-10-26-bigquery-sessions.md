---
layout: post
title: BigQuery Window Function Calculate User Sessions
tags: [bigquery, session, log, timestamp]
---

Combine stream of user actions with a timestamps into a windowed sessions example:

```sql
WITH data AS (
  -- user 1, session 1, morning
  SELECT TIMESTAMP('2001-01-01 07:01:00') AS timestamp,
                                        1 AS user,
                                   'view' AS action
  UNION ALL SELECT TIMESTAMP('2001-01-01 07:02:00'), 1, 'view'
  UNION ALL SELECT TIMESTAMP('2001-01-01 07:03:00'), 1, 'view'
  UNION ALL SELECT TIMESTAMP('2001-01-01 07:04:00'), 1, 'apply'

  -- user 1, session 2, midday
  UNION ALL SELECT TIMESTAMP('2001-01-01 12:01:00'), 1, 'view'
  UNION ALL SELECT TIMESTAMP('2001-01-01 12:02:00'), 1, 'view'
  UNION ALL SELECT TIMESTAMP('2001-01-01 12:03:00'), 1, 'apply'

  -- user 1, session 3, evening
  UNION ALL SELECT TIMESTAMP('2001-01-01 19:01:00'), 1, 'view'



  -- user 2, session 1, evening
  UNION ALL SELECT TIMESTAMP('2001-01-01 07:01:00'), 2, 'view'

  -- user 2, session 2, evening
  UNION ALL SELECT TIMESTAMP('2001-01-01 18:01:00'), 2, 'view'
  UNION ALL SELECT TIMESTAMP('2001-01-01 18:01:00'), 2, 'apply'

/*+---------------------+------+--------+
  |      timestamp      | user | action |
  +---------------------+------+--------+
  | 2001-01-01 07:01:00 |    1 | view   |
  | 2001-01-01 07:02:00 |    1 | view   |
  | 2001-01-01 07:03:00 |    1 | view   |
  | 2001-01-01 12:01:00 |    1 | view   |
  | 2001-01-01 12:02:00 |    1 | view   |
  | 2001-01-01 19:01:00 |    1 | view   |
  | 2001-01-01 07:04:00 |    1 | apply  |
  | 2001-01-01 12:03:00 |    1 | apply  |
  | 2001-01-01 07:01:00 |    2 | view   |
  | 2001-01-01 18:01:00 |    2 | view   |
  | 2001-01-01 18:01:00 |    2 | apply  |
  +---------------------+------+--------+*/
), data_with_previous_timestamp AS (
  SELECT
  -- POI: previous timestamp if any
  LAG(timestamp, 1) OVER (PARTITION BY user ORDER BY timestamp) AS previous_timestamp,
  *
  FROM data

/*+---------------------+---------------------+------+--------+
  | previous_timestamp  |      timestamp      | user | action |
  +---------------------+---------------------+------+--------+
  |                NULL | 2001-01-01 07:01:00 |    1 | view   | <- I have NO previous action
  | 2001-01-01 07:01:00 | 2001-01-01 07:02:00 |    1 | view   | <- I HAVE previous timestamp
  | 2001-01-01 07:02:00 | 2001-01-01 07:03:00 |    1 | view   |
  | 2001-01-01 07:03:00 | 2001-01-01 07:04:00 |    1 | apply  |
  | 2001-01-01 07:04:00 | 2001-01-01 12:01:00 |    1 | view   |
  | 2001-01-01 12:01:00 | 2001-01-01 12:02:00 |    1 | view   |
  | 2001-01-01 12:02:00 | 2001-01-01 12:03:00 |    1 | apply  |
  | 2001-01-01 12:03:00 | 2001-01-01 19:01:00 |    1 | view   |
  |                NULL | 2001-01-01 07:01:00 |    2 | view   | <- I have no previous action
  | 2001-01-01 07:01:00 | 2001-01-01 18:01:00 |    2 | view   |
  | 2001-01-01 18:01:00 | 2001-01-01 18:01:00 |    2 | apply  |
  +---------------------+---------------------+------+--------+*/
), data_with_is_new_session AS (
  SELECT
  -- POI: 30 min window
  CASE WHEN TIMESTAMP_DIFF(timestamp, previous_timestamp, MINUTE) >= 30 OR previous_timestamp IS NULL THEN 1 ELSE 0 END AS is_new_session,
  *
  FROM data_with_previous_timestamp

/*+----------------+---------------------+---------------------+------+--------+
  | is_new_session | previous_timestamp  |      timestamp      | user | action |
  +----------------+---------------------+---------------------+------+--------+
  |              1 |                NULL | 2001-01-01 07:01:00 |    1 | view   | <- user 1, session 1
  |              0 | 2001-01-01 07:01:00 | 2001-01-01 07:02:00 |    1 | view   |
  |              0 | 2001-01-01 07:02:00 | 2001-01-01 07:03:00 |    1 | view   |
  |              0 | 2001-01-01 07:03:00 | 2001-01-01 07:04:00 |    1 | apply  |
  |              1 | 2001-01-01 07:04:00 | 2001-01-01 12:01:00 |    1 | view   | <- user 1, session 2
  |              0 | 2001-01-01 12:01:00 | 2001-01-01 12:02:00 |    1 | view   |
  |              0 | 2001-01-01 12:02:00 | 2001-01-01 12:03:00 |    1 | apply  |
  |              1 | 2001-01-01 12:03:00 | 2001-01-01 19:01:00 |    1 | view   | <- user 1, session 3
  |              1 |                NULL | 2001-01-01 07:01:00 |    2 | view   | <- user 2, session 1
  |              1 | 2001-01-01 07:01:00 | 2001-01-01 18:01:00 |    2 | view   | <- user 2, session 2
  |              0 | 2001-01-01 18:01:00 | 2001-01-01 18:01:00 |    2 | apply  |
  +----------------+---------------------+---------------------+------+--------+*/
), data_with_sessions AS (
  SELECT
  -- POI: will be 1, 2, 3, 4, 5 - e.g. overall sessions
  SUM(is_new_session) OVER (ORDER BY user, timestamp) AS global_session_id,
  -- POI: will be 1, 2, 3 for user 1 and 1, 2 for user 2 - e.g. sessions per user
  SUM(is_new_session) OVER (PARTITION BY user ORDER BY timestamp) AS user_session_id,
  *
  FROM data_with_is_new_session

/*+-------------------+-----------------+----------------+---------------------+---------------------+------+--------+
  | global_session_id | user_session_id | is_new_session | previous_timestamp  |      timestamp      | user | action |
  +-------------------+-----------------+----------------+---------------------+---------------------+------+--------+
  |                 1 |               1 |              1 |                NULL | 2001-01-01 07:01:00 |    1 | view   |
  |                 1 |               1 |              0 | 2001-01-01 07:01:00 | 2001-01-01 07:02:00 |    1 | view   |
  |                 1 |               1 |              0 | 2001-01-01 07:02:00 | 2001-01-01 07:03:00 |    1 | view   |
  |                 1 |               1 |              0 | 2001-01-01 07:03:00 | 2001-01-01 07:04:00 |    1 | apply  |
  |                 2 |               2 |              1 | 2001-01-01 07:04:00 | 2001-01-01 12:01:00 |    1 | view   |
  |                 2 |               2 |              0 | 2001-01-01 12:01:00 | 2001-01-01 12:02:00 |    1 | view   |
  |                 2 |               2 |              0 | 2001-01-01 12:02:00 | 2001-01-01 12:03:00 |    1 | apply  |
  |                 3 |               3 |              1 | 2001-01-01 12:03:00 | 2001-01-01 19:01:00 |    1 | view   |
  |                 4 |               1 |              1 |                NULL | 2001-01-01 07:01:00 |    2 | view   |
  |                 5 |               2 |              1 | 2001-01-01 07:01:00 | 2001-01-01 18:01:00 |    2 | view   |
  |                 5 |               2 |              0 | 2001-01-01 18:01:00 | 2001-01-01 18:01:00 |    2 | apply  |
  +-------------------+-----------------+----------------+---------------------+---------------------+------+--------+*/
)
SELECT AVG(views) AS avg_views_per_session FROM (
  SELECT
  -- user,
  -- user_session_id,
  COUNT(*) AS views
  FROM data_with_sessions
  WHERE action = 'view'
  GROUP BY user, user_session_id
)
/*+-----------------------+
  | avg_views_per_session |
  +-----------------------+
  |                   1.6 |
  +-----------------------+*/
```

[Main source](https://blog.modeanalytics.com/finding-user-sessions-sql/)

Note: to get ascii art use `bq query --nouse_legacy_sql "select * from acme"`
