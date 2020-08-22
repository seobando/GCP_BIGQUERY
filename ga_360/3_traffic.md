## Calculate Traffic

Refer to: https://towardsdatascience.com/how-to-query-and-calculate-google-analytics-data-in-bigquery-cab8fc4f396

### Traffic sources dimensions
- Referral Path
- Full Referrer
- Default Channel Grouping
- Campaign
- Source
- Medium
- Source / Medium
- Keyword
- Ad Content
- Social Network
- Social Source Referral
- Campaign Code
### Traffic sources metrics
-

```SQL
SELECT
  -- Referral Path (dimension)
  trafficSource.referralPath AS Referral_Path,
  -- Full Referrer (dimension)
  CONCAT(trafficSource.source,trafficSource.referralPath) AS Full_Referrer,
  -- Default Channel Grouping
  channelGrouping AS Default_Channel_Grouping,
  -- Campaign (dimension)
  trafficSource.campaign AS Campaign,
  -- Source (dimension)
  trafficSource.source AS Source,
  -- Medium (dimension)
  trafficSource.medium AS Medium,
  -- Source / Medium (dimension)
  CONCAT(trafficSource.source," / ",trafficSource.medium) AS Source_Medium,
  -- Keyword (dimension)
  trafficSource.keyword AS Keyword,
  -- Ad Content (dimension)
  trafficSource.adContent AS Ad_Content,
  -- Social Network (dimension)
  MAX(hits.social.socialNetwork) AS Social_Network,
  -- Social Source (dimension)
  MAX(hits.social.hasSocialSourceReferral) AS Social_Source,
  -- Campaign Code (dimension)
  trafficSource.campaignCode AS Campaign_Code
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
  12
LIMIT
  10
```
