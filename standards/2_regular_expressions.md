## Regulars expressions: 

 * When column_field content value
  ```SQL
  REGEXP_CONTAINS(column, 'value')
  ``` 
 * When column_field start with value
  ```SQL
  REGEXP_CONTAINS(column, '^value.*')
  ``` 
 * When column_field ends with value
  ```SQL
  REGEXP_CONTAINS(column, 'value$')
  ``` 
 * When column_field between value_1 and value_2
  ```SQL
 REGEXP_CONTAINS(column, 'value_1(.*?)value_2')
  ```   
 * More but for DatStudio
  Buenas, por si les pasa que usaron el pipe "|" como separador en una categoria o acción o etiqueta de ventos, creen un campo usando 
 ```
 REGEXP_REPLACE(Etiqueta de evento,"\\|","-") y sobre ese campo con rel REGEXP_EXTRACT sacan la info que tienen
 ```
