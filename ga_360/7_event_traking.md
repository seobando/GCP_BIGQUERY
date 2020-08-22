## Calculate event variables

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Event Tracking dimensions
- Event Category
- Event Action
- Event Label
### Event Tracking metrics
- Total Events
- Unique Events
- Event Value
- Avg. Value
- Sessions with Event
- Events / Session with Event

```SQL
SELECT
  -- Event Category (dimension)
  hits.eventInfo.eventCategory AS Event_Category,
  -- Event Action (dimension)
  hits.eventInfo.eventAction AS Event_Action,
  -- Event Label (dimension)
  hits.eventInfo.eventLabel AS Event_Label,
  -- Total Events (metric)
  COUNT(*) AS Total_Events,
  -- Unique Events (metric),
  COUNT(DISTINCT CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))) AS Unique_Events,
  -- Event Value (metric)
  SUM(hits.eventInfo.eventValue) AS Event_Value,
  -- Avg. Value (metric)
  SUM(hits.eventInfo.eventValue) / COUNT(*) AS Avg_Value,
  -- Sessions With Events (metric)
  COUNT(DISTINCT
    CASE
      WHEN hits.type = 'EVENT' THEN CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))
    ELSE
    NULL
  END
    ) AS Sessions_With_Events,
  -- Events / Session With Event (metric)
  COUNT(*) / COUNT(DISTINCT
    CASE
      WHEN hits.type = 'EVENT' THEN CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))
    ELSE
    NULL
  END
    ) AS Events_Session_With_Event
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
  AND hits.type = 'EVENT'
GROUP BY
  1,
  2,
  3
HAVING
  Event_Category IS NOT NULL
ORDER BY
  4 DESC
LIMIT
  10

