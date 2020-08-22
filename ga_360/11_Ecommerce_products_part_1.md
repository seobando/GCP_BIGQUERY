## Calculate Ecommerce product variables part 1

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Enhanced Ecommerce dimensions
- Product SKU
- Product
- Product Category (Enhanced Ecommerce)
- Product Brand
- Product Variant
### Enhanced Ecommerce metrics
- Quantity
- Unique Purchases
- Product Revenue
- Avg. Price
- Avg. QTY

```SQL
SELECT
  -- Product SKU (dimension)
  productSKU AS Product_SKU,
  -- Product (dimension)
  v2ProductName AS Product,
  -- Product Variant (dimension)
  productVariant AS Product_Variant,
  -- Product Brand (dimension)
  productBrand AS Product_Brand,
  -- Product Category (Enhanced Ecommerce) (dimension)
  v2ProductCategory AS Product_Category_Enhanced_Ecommerce,
  -- Unique Purchases (metric)
  COUNT(hits.transaction.transactionId) AS Unique_Purchases,
  -- Quantity (metric)
  SUM(productQuantity) AS Quantity,
  -- Product Revenue (metric)
  SUM(productRevenue)/1000000 AS Product_Revenue,
  -- Avg. Price
  SUM(productRevenue)/1000000 / SUM(productQuantity) AS Avg_Price,
  -- Avg. QTY
  SUM(productQuantity) / COUNT(hits.transaction.transactionId) AS Avg_Quantity
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits,
  UNNEST(product) AS product
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
  AND hits.eCommerceAction.action_type = '6'
GROUP BY
  1,
  2,
  3,
  4,
  5
ORDER BY
  6 DESC
LIMIT
  10
  
