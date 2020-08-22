## Users calculations 

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### User dimensions

- User Type
- Count of Sessions

### User metrics

- Users
- New Users
- % New Sessions
- Number of Sessions per User
- Hits

```SQL
SELECT
  -- User Type (dimension)
  CASE
    WHEN totals.newVisits = 1 THEN 'New Visitor'
  ELSE
  'Returning Visitor'
END
  AS User_Type,
  -- Count of Sessions (dimension)
  visitNumber AS Count_of_Sessions,
  -- Users (metric)
  COUNT(DISTINCT fullVisitorId) AS Users,
  -- New Users (metric)
  COUNT(DISTINCT(
      CASE
        WHEN totals.newVisits = 1 THEN fullVisitorId
      ELSE
      NULL
    END
      )) AS New_Users,
  -- % New Sessions (metric)
  COUNT(DISTINCT(
      CASE
        WHEN totals.newVisits = 1 THEN fullVisitorId
      ELSE
      NULL
    END
      )) / COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) AS Percentage_New_Sessions,
  -- Number of Sessions per User (metric)
  COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) / COUNT(DISTINCT fullVisitorId) AS Number_of_Sessions_per_User,
  -- Hits (metric)
  SUM((
    SELECT
      totals.hits
    FROM
      UNNEST(hits)
    GROUP BY
      1)) AS Hits
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
GROUP BY
  1,
  2
ORDER BY
  2
LIMIT
  10
```  
