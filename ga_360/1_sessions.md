## Sessions calculation

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Session dimensions
-
### Session metrics

- Sessions
- Bounces
- Bounce Rate
- Avg. Session Duration

```SQL
SELECT
  -- Sessions (metric)
  COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) AS Sessions,
  -- Bounces (metric)
  COUNT(DISTINCT
    CASE
      WHEN totals.bounces = 1 THEN CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))
    ELSE
    NULL
  END
    ) AS Bounces,
  -- Bounce Rate (metric)
  COUNT(DISTINCT
    CASE
      WHEN totals.bounces = 1 THEN CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))
    ELSE
    NULL
  END
    ) / COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) AS Bounce_Rate,
  -- Average Session Duration (metric)
  SUM(totals.timeOnSite) / COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) AS Average_Session_Duration
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
  ```
