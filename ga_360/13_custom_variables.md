## Calculate Custom Variables

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Custom Dimensions
- Custom Dimension XX (User)
- Custom Dimension XX (Session)
- Custom Dimension XX (Hit)
- Custom Dimension XX (Product)
### Custom Metrics
- Custom Metric XX Value (Hit)
- Custom Metric XX Value (Product)

```SQL
-- Some sample set custom dimensions return null values
SELECT
  -- Custom Dimension XX (User)
  (
  SELECT
    value
  FROM
    UNNEST(session.customDimensions)
  WHERE
    index = 3
  GROUP BY
    1) AS Custom_Dimension_XX_User,
  -- Custom Dimension XX (Session)
  (
  SELECT
    value
  FROM
    UNNEST(session.customDimensions)
  WHERE
    index = 4
  GROUP BY
    1) AS Custom_Dimension_XX_Session,
  -- Custom Dimension XX (Hit)
  (
  SELECT
    value
  FROM
    UNNEST(hits.customDimensions)
  WHERE
    index = 2
  GROUP BY
    1) AS Custom_Dimension_XX_Hit,
  -- Custom Dimension XX (Product)
  (
  SELECT
    value
  FROM
    UNNEST(product.customDimensions)
  WHERE
    index = 10
  GROUP BY
    1) AS Custom_Dimension_XX_Product,
  -- Custom Metric XX (Hit)
  SUM((
    SELECT
      value
    FROM
      UNNEST(hits.customMetrics)
    WHERE
      index = 1)) AS Custom_Metric_XX_Hit,
  -- Custom Metric XX (Product)
  SUM((
    SELECT
      value
    FROM
      UNNEST(product.customMetrics)
    WHERE
      index = 2)) AS Custom_Metric_XX_Product
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*` AS session,
  UNNEST(hits) AS hits,
  UNNEST(product) AS product
WHERE
  _table_suffix BETWEEN '20160101'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  2 DESC
LIMIT
  10
```    
## Other way

Refer to: https://medium.com/@JustinCarmony/strategies-for-easier-google-analytics-bigquery-analysis-custom-dimensions-cad8afe7a153

```SQL
WITH
  CD AS (
  SELECT
    WITH_VISIT.fullVisitorId,
    WITH_VISIT.visitId,
    WITH_HIT.hitNumber,
    WITH_CD.index,
    WITH_CD.value
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_*` WITH_VISIT,
    UNNEST(WITH_VISIT.hits) WITH_HIT,
    UNNEST(WITH_HIT.customDimensions) WITH_CD
  WHERE
    WITH_CD.index IN (1,2,4) )
SELECT
  VISIT.visitId,
  CD_1.value AS userid,
  CD_2.value AS segmento,
  CD_4.value AS campaing
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*` VISIT,
  UNNEST(VISIT.hits) HIT
LEFT JOIN
  CD CD_1
ON
  VISIT.fullVisitorId = CD_1.fullVisitorId
  AND VISIT.visitId = CD_1.visitId
  AND HIT.hitNumber = CD_1.hitNumber
  AND CD_1.index = 1
LEFT JOIN
  CD CD_2
ON
  VISIT.fullVisitorId = CD_2.fullVisitorId
  AND VISIT.visitId = CD_2.visitId
  AND HIT.hitNumber = CD_2.hitNumber
  AND CD_2.index = 2
LEFT JOIN
  CD CD_4
ON
  VISIT.fullVisitorId = CD_4.fullVisitorId
  AND VISIT.visitId = CD_4.visitId
  AND HIT.hitNumber = CD_4.hitNumber
  AND CD_4.index = 4  
GROUP BY
  1,
  2,
  3,
  4
```  
  
