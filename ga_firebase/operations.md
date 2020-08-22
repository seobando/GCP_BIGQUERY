## Events Detail View
```SQL
SELECT
  event_name,
  event_params.key,
  event_params.value.string_value,
  event_params.value.int_value,
  event_params.value.float_value
FROM
  `your_firebase_table`,
  UNNEST(event_params) AS event_params
```

## Transposición de columnas

```SQL
SELECT
  event_name,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = "TEST) AS screen_name,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE REGEXP_CONTAINS(key, "TEXT")) AS parameter
FROM
  `TABLE` AS session
WHERE
  event_name = "TEXT"  
GROUP BY 
1,2,3
ORDER BY 
1
```

## Sessions
```SQL
#https://stackoverflow.com/questions/42546815/how-to-calculate-session-and-session-duration-in-firebase-analytics-raw-data
SELECT
  user_pseudo_id,
  sess_id,
  MIN(min_time) sess_start,
  MAX(max_time) sess_end,
  COUNT(*) records,
  MAX(sess_id) OVER(PARTITION BY user_pseudo_id) total_sessions,
  (ROUND((MAX(max_time)-MIN(min_time))/(1000*1000),1)) sess_length_seconds
FROM (
  SELECT
    *,
    SUM(session_start) OVER(PARTITION BY user_pseudo_id ORDER BY min_time) sess_id
  FROM (
    SELECT
      *,
    IF
      (previous IS NULL
        OR (min_time-previous) > (20*60*1000*1000),
        1,
        0) session_start
    FROM (
      SELECT
        *,
        LAG(max_time, 1) OVER(PARTITION BY user_pseudo_id ORDER BY max_time) previous
      FROM (
        SELECT
          user_pseudo_id,
          MIN(event_timestamp) AS min_time,
          MAX(event_timestamp) AS max_time
        FROM
          `Table`
        GROUP BY
          user_pseudo_id) ) ) )
GROUP BY
  1,
  2
```

## AVG. Screen Time
```SQL
SELECT
  PARSE_DATE("%Y%m%d",CAST(event_date AS STRING)) AS particion, 
  event_date,
  userid,
  previuos_session_id,
  session_id,
  previuos_screen_name,
  screen_name,
  start_time,
  end_time,
  CASE 
    WHEN previuos_session_id IS NULL THEN avg_time
    WHEN previuos_session_id != session_id THEN 0
    WHEN avg_time >= 1800 THEN 0
  ELSE avg_time END AS avg_time,
  exit,
  event_count
FROM(
SELECT
  event_date,
  userid,
  LAG(session_id) OVER (PARTITION BY userid ORDER BY start_time) AS previuos_session_id,
  session_id,
  LAG(screen_name) OVER (PARTITION BY userid ORDER BY start_time) AS previuos_screen_name,
  screen_name,
  TIMESTAMP_MICROS(start_time) AS start_time,
  TIMESTAMP_MICROS(end_time) AS end_time,
  IF(end_time IS NULL,0,TIMESTAMP_DIFF(TIMESTAMP_MICROS(end_time),TIMESTAMP_MICROS(start_time),SECOND)) AS avg_time,
  IF(end_time IS NULL,"SI", "NO") AS exit,
  event_count
FROM(
SELECT
  event_date,
  user_pseudo_id AS userid,
  (SELECT value.int_value FROM UNNEST(event_params) WHERE key = "ga_session_id") AS session_id,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = "firebase_screen") AS screen_name,
  event_timestamp AS start_time,
  LEAD(event_timestamp) OVER (PARTITION BY user_pseudo_id ORDER BY event_timestamp) AS end_time,
  COUNT(*) AS event_count
FROM
  `your_firebase_table`
WHERE
  event_name = "screen_view"
GROUP BY
  1,
  2,
  3,
  4,
  5
HAVING
  session_id IS NOT NULL AND	screen_name IS NOT NULL
ORDER BY
  user_pseudo_id,event_timestamp))


```
