## Calculate Page Tracking Variables

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Page Tracking dimensions
- Hostname
- Page
- Previous Page Path
- Page path level 1
- Page path level 2
- Page path level 3
- Page path level 4
- Page Title
- Landing Page
- Second Page
- Exit Page
### Page Tracking metrics
- Entrances
- Pageviews
- Unique Pageviews (see separate example query)
- Pages / Session
- Exits
- % Exit
- Avg. Time on Page (see separate example query)

```SQL
SELECT
  Hostname,
  Page,
  Previous_Page,
  Page_Path_Level_1,
  Page_Path_Level_2,
  Page_Path_Level_3,
  Page_Path_Level_4,
  Page_Title,
  Landing_Page,
  Second_Page,
  Exit_Page,
  -- Entrances (metric)
  SUM(CASE
      WHEN isEntrance = TRUE THEN 1
    ELSE
    0
  END
    ) AS Entrances,
  -- Pageviews (metric)
  COUNT(*) AS Pageviews,
  -- Pages Per Session (metric)
  COUNT(*) / COUNT(DISTINCT CONCAT(fullVisitorId, CAST(visitStartTime AS STRING))) AS Pages_Per_Session,
  -- Exits (metric)
  SUM(CASE
      WHEN isExit = TRUE THEN 1
    ELSE
    0
  END
    ) AS Exits,
  -- Exit Rate (metric)
  SUM(CASE
      WHEN isExit = TRUE THEN 1
    ELSE
    0
  END
    ) / COUNT(*) AS Exit_Rate
FROM (
  SELECT
    -- Hostname (dimension)
    hits.page.hostname AS Hostname,
    -- Page (dimension)
    hits.page.pagePath AS Page,
    -- Previous Page (dimension)
    LAG(hits.page.pagePath, 1) OVER (PARTITION BY fullvisitorId, visitStartTime ORDER BY hits.hitNumber ASC) AS Previous_Page,
    -- Page path level 1 (dimension)
    hits.page.pagePathLevel1 AS Page_Path_Level_1,
    -- Page path level 2 (dimension)
    hits.page.pagePathLevel2 AS Page_Path_Level_2,
    -- Page path level 3 (dimension)
    hits.page.pagePathLevel3 AS Page_Path_Level_3,
    -- Page path level 4 (dimension)
    hits.page.pagePathLevel4 AS Page_Path_Level_4,
    -- Page Title (dimension)
    hits.page.pageTitle AS Page_Title,
    -- Landing Page (dimension)
    CASE
      WHEN hits.isEntrance = TRUE THEN hits.page.pagePath
    ELSE
    NULL
  END
    AS Landing_page,
    -- Second Page (dimension)
    CASE
      WHEN hits.isEntrance = TRUE THEN (LEAD(hits.page.pagePath, 1) OVER (PARTITION BY fullvisitorId, visitStartTime ORDER BY hits.hitNumber ASC))
    ELSE
    NULL
  END
    AS Second_Page,
    -- Exit Page (dimension)
    CASE
      WHEN hits.isExit = TRUE THEN hits.page.pagePath
    ELSE
    NULL
  END
    AS Exit_page,
    hits.isEntrance,
    fullVisitorId,
    visitStartTime,
    hits.isExit
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST(hits) AS hits
  WHERE
    _table_suffix BETWEEN '20160801'
    AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
    AND totals.visits = 1
    AND hits.type = 'PAGE')
GROUP BY
  1,
  2,
  3,
  4,
  5,
  6,
  7,
  8,
  9,
  10,
  11
ORDER BY
  13 DESC
LIMIT
  10
```

### Unique page views:

```SQL
SELECT
  -- Page (dimension)
  Page,
  -- Unique Pageviews (metric)
  SUM(unique_pageviews) AS Unique_Pageviews
FROM (
  SELECT
    hits.page.pagePath AS Page,
    CONCAT(hits.page.pagePath,hits.page.pageTitle) AS page_concat,
    COUNT(DISTINCT CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))) AS unique_pageviews
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST(hits) AS hits
  WHERE
    _table_suffix BETWEEN '20160801'
    AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
    AND totals.visits = 1
    AND hits.type = 'PAGE'
  GROUP BY
    1,
    2 )
GROUP BY
  1
ORDER BY
  2 DESC
LIMIT
  10
```

### Avg. time on page:

```SQL
SELECT
  -- Page (dimension)
  pagePath AS Page,
  -- Avg Time On Page (metric)
  CASE
    WHEN pageviews = exits THEN 0
  ELSE
  total_time_on_page / (pageviews - exits)
END
  AS Avg_Time_On_Page
FROM (
  SELECT
    pagePath,
    COUNT(*) AS pageviews,
    SUM(
    IF
      (isExit IS NOT NULL,
        1,
        0)) AS exits,
    SUM(time_on_page) AS total_time_on_page
  FROM (
    SELECT
      fullVisitorId,
      visitStartTime,
      pagePath,
      hit_time,
      type,
      isExit,
      CASE
        WHEN isExit IS NOT NULL THEN last_interaction - hit_time
      ELSE
      next_pageview - hit_time
    END
      AS time_on_page
    FROM (
      SELECT
        fullVisitorId,
        visitStartTime,
        pagePath,
        hit_time,
        type,
        isExit,
        last_interaction,
        LEAD(hit_time) OVER (PARTITION BY fullVisitorId, visitStartTime ORDER BY hit_time) AS next_pageview
      FROM (
        SELECT
          fullVisitorId,
          visitStartTime,
          pagePath,
          hit_time,
          type,
          isExit,
          last_interaction
        FROM (
          SELECT
            fullVisitorId,
            visitStartTime,
            hits.page.pagePath,
            hits.type,
            hits.isExit,
            hits.time / 1000 AS hit_time,
            MAX(
            IF
              (hits.isInteraction = TRUE,
                hits.time / 1000,
                0)) OVER (PARTITION BY fullVisitorId, visitStartTime) AS last_interaction
          FROM
            `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
            UNNEST(hits) AS hits
          WHERE
            _table_suffix BETWEEN '20160801'
            AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
            AND totals.visits = 1
            AND hits.type = 'PAGE')))
    GROUP BY
      1,
      2,
      3,
      4,
      5,
      6,
      7)
  GROUP BY
    1)
ORDER BY
  2 DESC
LIMIT
  10
```


