## Calculate time dimensiones

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Time dimensions
- Date
- Year
- ISO Year
- Month of Year
- Month of the year
- Week of Year
- Week of the Year
- ISO Week of the Year
- ISO Week of ISO Year
- Day of the month
- Day of Week
- Day of Week Name
- Hour
- Minute
- Hour of Day
- Date Hour and Minute
### Time metrics

```SQL
SELECT
  -- Date (dimension)
  date AS Date,
  -- Year (dimension)
  FORMAT_DATE('%Y', PARSE_DATE("%Y%m%d",
      date)) AS Year,
  -- ISO Year (dimension)
  FORMAT_DATE('%G', PARSE_DATE("%Y%m%d",
      date)) AS ISO_Year,
  -- Month of Year (dimension)
  FORMAT_DATE('%Y%m', PARSE_DATE("%Y%m%d",
      date)) AS Month_of_Year,
  -- Month of the Year (dimension)
  FORMAT_DATE('%m', PARSE_DATE("%Y%m%d",
      date)) AS Month_of_the_Year,
  -- Week of Year (dimension)
  FORMAT_DATE('%Y%U', PARSE_DATE("%Y%m%d",
      date)) AS Week_of_Year,
  -- Week of the Year (dimension)
  FORMAT_DATE('%U', PARSE_DATE("%Y%m%d",
      date)) AS Week_of_the_Year,
  -- ISO Week of the Year (dimension)
  FORMAT_DATE('%W', PARSE_DATE("%Y%m%d",
      date)) AS ISO_Week_of_the_Year,
  -- ISO Week of ISO Year (dimension)
  FORMAT_DATE('%G%W', PARSE_DATE("%Y%m%d",
      date)) AS ISO_Week_of_ISO_Year,
  -- Day of the Month (dimension)
  FORMAT_DATE('%d', PARSE_DATE("%Y%m%d",
      date)) AS Day_of_the_Month,
  -- Day of Week (dimension)
  FORMAT_DATE('%w', PARSE_DATE("%Y%m%d",
      date)) AS Day_of_Week,
  -- Day of Week Name (dimension)
  FORMAT_DATE('%A', PARSE_DATE("%Y%m%d",
      date)) AS Day_of_Week_Name,
  -- Hour of Day(dimension)
  CONCAT(date,CAST(EXTRACT (hour
      FROM
        TIMESTAMP_SECONDS(visitStartTime)) AS String)) AS Hour_of_Day,
  -- Hour (dimension)
  FORMAT("%02d",EXTRACT (hour
    FROM
      TIMESTAMP_SECONDS(visitStartTime))) AS Hour,
  -- Minute (dimension)
  FORMAT("%02d",EXTRACT (Minute
    FROM
      TIMESTAMP_SECONDS(visitStartTime))) AS Minute,
  -- Date Hour and Minute (dimension)
  CONCAT(CONCAT(date,CAST(EXTRACT (hour
        FROM
          TIMESTAMP_SECONDS(visitStartTime)) AS String)),FORMAT("%02d",EXTRACT (hour
      FROM
        TIMESTAMP_SECONDS(visitStartTime))),FORMAT("%02d",EXTRACT (Minute
      FROM
        TIMESTAMP_SECONDS(visitStartTime)))) AS Date_Hour_and_Minute
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
ORDER BY
  16 DESC
LIMIT
  10
