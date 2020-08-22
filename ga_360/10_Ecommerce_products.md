### Calculate Ecommerce product variables

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Ecommerce dimensions
- Product SKU
- Product
- Product Category
### Ecommerce metrics
- Quantity
- Unique Purchases
- Avg. Price
- Product Revenue
- Avg. QTY

```SQL
  -- This query will return no data as there is no standard ecommerce product data available in the sample data set.
SELECT
  -- Product SKU (dimension)
  productSku AS Product_SKU,
  -- Product (dimension)
  v2ProductName AS Product,
  -- Product Category (dimension)
  hits.item.productCategory AS Product_Category,
  -- Quantity
  SUM(hits.item.itemQuantity) AS Quantity,
  -- Unique Purchases (metric)
  CASE
    WHEN hits.item.productSKU IS NOT NULL THEN COUNT(hits.transaction.transactionId)
  ELSE
  NULL
END
  AS Unique_Purchases,
  -- Product Revenue
  SUM(hits.item.itemRevenue)/1000000 AS Product_Revenue,
  -- Avg. Price
  SUM(hits.item.itemRevenue)/1000000 / SUM(hits.item.itemQuantity) AS Avg_Price,
  -- Avg. QTY
  SUM(hits.item.itemQuantity) / COUNT(hits.transaction.transactionId) AS Avg_Quantity
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits
WHERE
  _table_suffix BETWEEN '20170801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
GROUP BY
  1,
  2,
  3
HAVING
  Product_SKU IS NOT NULL
ORDER BY
  4 DESC
LIMIT
  10
```
