## Delete duplicates

```SQL
DELETE
FROM duplicates AS d
WHERE (SELECT ROW_NUMBER() OVER (PARTITION BY id ORDER BY loadTime DESC)
       FROM `duplicates` AS d2
       WHERE d.id = d2.id AND d.loadTime = d2.loadTime) > 1;
```
## Alternative

```SQL
SELECT
  *
FROM (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY Transaction_ID) row_number
  FROM
    `table`)
WHERE
  row_number = 1
  ```
