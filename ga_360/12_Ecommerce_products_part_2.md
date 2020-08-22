## Calculate Ecommerce product variables part 2

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Enhanced Ecommerce dimensions
- Product SKU
### Enhanced Ecommerce metrics
- Buy-to-Detail Rate
- Cart-to-Detail Rate
- Product Adds To Cart
- Product Checkouts
- Product Detail Views
- Product Refunds
- Product Removes From Cart
- Refund Amount

```SQL
SELECT
  -- Product SKU (dimension)
  productSKU AS Product_SKU,
  -- Unique Purchases (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '6' THEN hits.transaction.transactionId
    ELSE
    NULL
  END
    ) AS Unique_Purchases,
  -- Cart-to-Detail Rate (metric)
  CASE
    WHEN COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
    ELSE
    NULL
  END
    ) = 0 THEN 0
  ELSE
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '3' THEN fullVisitorId
    ELSE
    NULL
  END
    ) / COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
    ELSE
    NULL
  END
    )
END
  AS Cart_To_Detail_Rate,
  -- Buy-to-Detail Rate (metric)
  CASE
    WHEN COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
    ELSE
    NULL
  END
    ) = 0 THEN 0
  ELSE
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '6' THEN hits.transaction.transactionId
    ELSE
    NULL
  END
    ) / COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
    ELSE
    NULL
  END
    )
END
  AS Buy_To_Detail_Rate,
  -- Product Detail Views (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
    ELSE
    NULL
  END
    ) AS Product_Detail_Views,
  -- Product Adds To Cart (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '3' THEN fullVisitorId
    ELSE
    NULL
  END
    ) AS Product_Adds_To_Cart,
  -- Product Removes From Cart (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '4' THEN fullVisitorId
    ELSE
    NULL
  END
    ) AS Product_Removes_From_Cart,
  -- Product Checkouts (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '5' THEN fullVisitorId
    ELSE
    NULL
  END
    ) AS Product_Checkouts,
  -- Product Refunds (metric)
  COUNT(CASE
      WHEN hits.eCommerceAction.action_type = '7' THEN fullVisitorId
    ELSE
    NULL
  END
    ) AS Product_Refunds
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
  UNNEST(hits) AS hits,
  UNNEST(product) AS product
WHERE
  _table_suffix BETWEEN '20160801'
  AND FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))
  AND totals.visits = 1
  AND (isImpression IS NULL
    OR isImpression = FALSE)
GROUP BY
  1
ORDER BY
  2 DESC
LIMIT
  10
