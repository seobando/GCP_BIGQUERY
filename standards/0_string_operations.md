## String operations:

  * Cast
  ```SQL
  CAST(column AS STRING|INT64|FLOAT64|NUMERIC|BOOL|BYTES|DATE)
  ```
  * Substring
  ```SQL  
  SAFE.SUBSTR(column, position, length) 
  ```  
  * Split
  ```SQL
  SPLIT(column, 'criteria')[SAFE_ORDINAL(position)]
  ``` 
