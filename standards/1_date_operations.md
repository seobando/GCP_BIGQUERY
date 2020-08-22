## Date operations:

 * Get Year|Month|Day|ISOWEEK
 ```SQL
 EXTRACT(YEAR|MONTH|DAY|ISOWEEK FROM date)
```
 * Get difference Between Dates
 ```SQL
 DATE_DIFF(DATE 'YYYY-MM-DD', DATE 'YYYY-MM-DD', DAY|MONT|YEAR|WEEK(MONDAY))
```
 * Get the day of the week when the date type is integer in the format YYYYMMDD
 ```SQL
  CASE
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '1' THEN '1.Monday'
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '2' THEN '2.Tuesday'
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '3' THEN '3.Wednesday'
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '4' THEN '4.Thursday'
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '5' THEN '5.Friday'
   WHEN FORMAT_DATE('%u', PARSE_DATE('%Y%m%d', CAST(date_integer AS STRING))) = '6' THEN '6.Saturday'
 ELSE
 '7.Sunday'
 END
 AS day_week
``` 
