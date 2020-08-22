## Calculate Platform or device dimensiones

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Platform or Device dimensions
- Browser
- Browser Version
- Operating System
- Operating System Version
- Mobile Device Branding
- Mobile Device Model
- Mobile Input Selector
- Mobile Device Info
- Mobile Device Marketing Name
- Device Category
- Browser Size
- Data Source
### Platform or Device metrics
-

```SQL
SELECT
  -- Browser (dimension)
  device.browser AS Browser,
  -- Browser Version (dimension)
  device.browserVersion AS Browser_Version,
  -- Operating System (dimension)
  device.operatingSystem AS Operating_System,
  -- Operating System Version (dimension)
  device.operatingSystemVersion AS Operating_System_Version,
  -- Mobile Device Branding (dimension)
  device.mobileDeviceBranding AS Mobile_Device_Branding,
  -- Mobile Device Model (dimension)
  device.mobileDeviceModel AS Mobile_Device_Model,
  -- Mobile Input Selector (dimension)
  device.mobileInputSelector AS Mobile_Input_Selector,
  -- Mobile Device Info (dimension)
  device.mobileDeviceInfo AS Mobile_Device_Info,
  -- Mobile Device Marketing Name (dimension)
  device.mobileDeviceMarketingName AS Mobile_Device_Marketing_Name,
  -- Device Category (dimension)
  device.deviceCategory AS Device_Category,
  -- Browser Size (dimension)
  device.browserSize AS Browser_Size,
  -- Data Source (dimension)
  MAX(hits.dataSource) AS Data_Source
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
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
LIMIT
  10
