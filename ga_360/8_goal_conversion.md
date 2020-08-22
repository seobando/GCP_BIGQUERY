## Goal conversion calculation

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Goal Conversions dimensions
- Goal Completion Location
- Goal Previous Step-1
- Goal Previous Step-2
- Goal Previous Step-3
### Goal Conversions metrics
- Goal XX Completions

```SQL
SELECT
  Goal_Completion_Location,
  Goal_Previous_Step_1,
  Goal_Previous_Step_2,
  Goal_Previous_Step_3,
  -- GoalXX Completions (metric)
  COUNT(DISTINCT(
    IF
      (REGEXP_CONTAINS(Goal_Completion_Location, '/ordercompleted'),
        Session,
        NULL))) AS GoalXX_Completions,
  -- GoalXY Completions (metric)
  COUNT(DISTINCT(
    IF
      (REGEXP_CONTAINS(Goal_Completion_Location, '/basket.html'),
        Session,
        NULL))) AS GoalXY_Completions
FROM (
  SELECT
    -- Goal Completion Location (dimension)
    hits.page.pagePath AS Goal_Completion_Location,
    -- Goal Previous Step 1 (dimension)
    LAG(hits.page.pagePath, 1) OVER (PARTITION BY fullvisitorId, visitStartTime ORDER BY hits.hitNumber ASC) AS Goal_Previous_Step_1,
    -- Goal Previous Step 2 (dimension)
    LAG(hits.page.pagePath, 2) OVER (PARTITION BY fullvisitorId, visitStartTime ORDER BY hits.hitNumber ASC) AS Goal_Previous_Step_2,
    -- Goal Previous Step 3 (dimension)
    LAG(hits.page.pagePath, 3) OVER (PARTITION BY fullvisitorId, visitStartTime ORDER BY hits.hitNumber ASC) AS Goal_Previous_Step_3,
    CONCAT(CAST(fullvisitorid AS string),CAST(visitStartTime AS string)) AS Session
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST(hits) AS hits
  WHERE
    _table_suffix BETWEEN '20170801'
    AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
    AND totals.visits = 1 )
GROUP BY
  1,
  2,
  3,
  4
HAVING
  Goal_Completion_Location NOT IN (Goal_Previous_Step_1,
    Goal_Previous_Step_2,
    Goal_Previous_Step_3)
  AND GoalXX_Completions > 0
  OR GoalXY_Completions > 0
ORDER BY
  5 DESC,
  6 DESC
LIMIT
  10
  
