## Joins

  * Two tables union
  ```SQL
  SELECT
      column_key_table_1,
      column_a,
      column_b
  FROM(
    (SELECT 
      column_key_table_1,
      column_a
    FROM
      `project.dataset.table`) AS table_1
    JOIN|LEFT JOIN|RIGHT JOIN
    (
      SELECT 
        column_key_table_2,
        column_b
      FROM
        `project.dataset.table`      
    ) AS table_2
    ON
      table_1.column_key_table_1 = table_2.column_key_table_2
  
  ```
  * More than two tables union

  ```SQL
  SELECT
      column_key_layer_2,
      column_a,
      column_b,
      column_c
  FROM
  (SELECT
      column_key_layer_0,
      column_a,
      column_b
  FROM(
    (SELECT 
      column_key_table_1,
      column_a
    FROM
      `project.dataset.table`) AS table_1
    JOIN|LEFT JOIN|RIGHT JOIN
    (
      SELECT 
        column_key_table_2,
        column_b
      FROM
        `project.dataset.table`      
    ) AS table_2
    ON
      table_1.column_key_table_1 = table_2.column_key_table_2) AS layer_0
    JOIN|LEFT JOIN|RIGHT JOIN
    (
      SELECT 
        column_key_table_3,
        column_c
      FROM
        `project.dataset.table`      
    ) AS table_3
    ON
      layer_0.column_key_layer_1 = column_key_table_3   
  
  ```
