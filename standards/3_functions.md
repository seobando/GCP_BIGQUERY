
## Functions:

 * Count uniques
 ```SQL
 COUNT(DISTINCT column)
 ```
 * SHA256
 ```SQL
 TO_HEX(SHA256("Hello"))
 ```
 
 * SPLIT
```SQL
SELECT SPLIT(path, '/')[OFFSET(0)] part1,
       SPLIT(path, '/')[OFFSET(1)] part2,
       SPLIT(path, '/')[OFFSET(2)] part3
FROM (SELECT "/a/b/aaaa?c" path)
```
 
