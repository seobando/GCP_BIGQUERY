## Table operations:

 * Create Partition Table
  ```SQL
  CREATE TABLE
    `table_destination`
  PARTITION BY
    particion
    CLUSTER BY campaign AS 
  SELECT 
  *,PARSE_DATE("%Y%m%d",CAST(date AS STRING)) AS particion 
  FROM
    `table_origin`
  ```
  * Truncate table
  ```SQL
  DELETE `project.dataset.table` WHERE 1=1
  ```
  * Insert
  ```SQL
  INSERT INTO
  `project.dataset.table`(
    column_1,
    column_n)
   SELECT
    column_1,
    column_n
   FROM `project.dataset.table`
  ```
  * Update
  ```SQL
  UPDATE `project.dataset.table` SET column = 'value' WHERE condition
  ```
  * Alter table [use query settings]
  ```SQL
  SELECT * EXCEPT (column_a, column_b, column_c) FROM `project.dataset.table`
  ```
  * Unnest
  ```SQL
  SELECT
   event_params.value_1
  FROM
   `project.dataset.table`
  CROSS JOIN
  UNNEST(event_params) AS event_params
  GROUP BY
   event_params.value_1
  ```
  * Get Unique values
  ```SQL
  SELECT
    *EXCEPT(row_number)
  FROM (
    SELECT
      *,
      ROW_NUMBER() OVER (PARTITION BY key) row_number
    FROM (
      SELECT
        Date_Hour_and_Minute,
        CONCAT(CONCAT( userid,"-"),product_sku) AS key
      FROM
        `project.dataset.table`
      ORDER BY
        Date_Hour_and_Minute DESC))
  WHERE
    row_number = 1
  ```
  * Count if query return rows
  ```SQL
  SELECT
  IFNULL((
    SELECT
      COUNT(1)
    FROM
      `your_table`
    GROUP BY
      _TABLE_SUFFIX
    HAVING
      _TABLE_SUFFIX = FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)) ),
    0)
  ````
* Control Flow

```SQL
DECLARE
  validacion INT64;
SET
  validacion = (
  SELECT
    IFNULL((
      SELECT
        COUNT(1)
      FROM
        `table_name`
      GROUP BY
        _TABLE_SUFFIX
      HAVING
        _TABLE_SUFFIX = FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)) ),
      0) );
IF
  validacion > 0 THEN
  SELECT
    *
  FROM
    `table_name`
  WHERE
    _TABLE_SUFFIX = FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY));
ELSE
  SELECT
    *
  FROM
    `table_name`
  WHERE
    _TABLE_SUFFIX = FORMAT_DATE('%Y%m%d',DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY));
END IF;
```

* Update one table from another

```SQL
MERGE `novaventa-co.NV_ANALITICA_DIGITAL.UPDATE_TEST` AS TABLE_A
USING `novaventa-co.NV_ANALITICA_DIGITAL.UPDATE_TEST_NEW_STATUS` AS TABLE_B
ON TABLE_A.USERID = TABLE_B.USERID
WHEN MATCHED THEN
  UPDATE SET ESTADO = TABLE_B.ESTADO
WHEN NOT MATCHED THEN
  INSERT ( ID, CICLO, USERID, ESTADO, FECHA_1, FECHA_2) VALUES ( ID, CICLO, USERID, ESTADO, FECHA_1, FECHA_2)
```  


