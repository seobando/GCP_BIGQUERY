## Calculate Ecommerce Variables

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### (Enhanced) Ecommerce dimensions
- Transaction ID
### (Enhanced) Ecommerce metrics
- Transactions
- Ecommerce Conversion Rate
- Revenue
- Avg. Order Value
- Per Session Value
- Shipping
- Tax
- Revenue per User
- Transactions per User

```SQL
SELECT
  -- Transaction ID (dimension)
  hits.transaction.transactionId AS Transaction_ID,
  -- Transactions (metric)
  COUNT(DISTINCT hits.transaction.transactionId) AS Transactions,
  -- Revenue (metric)
  SUM(hits.transaction.transactionRevenue)/1000000 AS Revenue,
  -- Ecommerce Conversion Rate
  COUNT(DISTINCT hits.transaction.transactionId) / COUNT(DISTINCT CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))) AS Ecommerce_Conversion_Rate,
  -- Avg. Order Value
  (SUM(hits.transaction.transactionRevenue)/1000000)/COUNT(DISTINCT hits.transaction.transactionId) AS Avg_Order_Value,
  -- Per Session Value
  (SUM(hits.transaction.transactionRevenue)/1000000) / COUNT(DISTINCT CONCAT(CAST(fullVisitorId AS STRING), CAST(visitStartTime AS STRING))) AS Per_Session_Value,
  -- Shipping
  SUM(hits.transaction.transactionShipping)/1000000 AS Shipping,
  -- Tax
  SUM(hits.transaction.transactionTax)/1000000 AS Tax,
  -- Revenue Per User
  (SUM(hits.transaction.transactionRevenue)/1000000) / COUNT(DISTINCT fullVisitorId) AS Revenue_Per_User,
  -- Transactions Per User
  COUNT(DISTINCT hits.transaction.transactionId) / COUNT(DISTINCT fullVisitorId) AS Transactions_Per_User
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
GROUP BY
  1
HAVING
  hits.transaction.transactionId IS NOT NULL
ORDER BY
  1 DESC
LIMIT
  10
```  
