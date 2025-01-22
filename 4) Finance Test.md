## Introduction:
As you develop transformation pipelines and collaborate with colleagues, it’s easy to make mistakes. Since data transformations build on one another, a single error can impact the entire pipeline and cause it to fail. So, never underestimate the importance of testing your work!

In the previous course, we learned how to test that:

- The primary key is respected → essential
- Some columns follow a defined format → optional

SQL joins are complex operations that can lead to data loss if not done correctly. It’s crucial to add additional tests to ensure no information is lost when joining source data to the destination aggregate data.

## 1) Error Handling

When performing a join, several types of errors can occur:

Duplication:
- If the join is made on the wrong key
- If there is duplication in one of the source tables

Loss of Information:
- If the join is made on an incorrect key
- If the join type (INNER, LEFT, RIGHT) is not correctly chosen
- If there’s an error in the WHERE condition (e.g., referencing the right table while using a LEFT join)
- If a missing key in one of the source tables prevents the join from working

Some of these problems can be detected using the tests discussed in the previous lesson:

Duplication can be detected with the primary key test.
- Some information loss can be detected with the column test by checking for null values.
- However, if the loss involves a missing row in the join, neither the primary key test nor the column test can detect it. For this, we need a retention test.

#### A retention test checks that a relevant metric from a source table is retained correctly in the join query. The metric can be the number of rows, distinct values, the sum, or the average of a column metric.

Example: gwz_orders_operational
Let’s take an example with the gwz_orders_operational join request created in the previous exercise. This query joins gwz_orders and gwz_ship.

Theoretically, all orders_id should be present in both tables without duplication.

### 1.1) Primary Key Test - gwz_orders_operationnal_pk

This test checks for duplication in the join table.

Run the gwz_orders_operational_pk test and ensure there are no errors.

```
SELECT
  ### Key ###
  orders_id,
  ###########
  COUNT(*) AS number
FROM `course16.gwz_orders_operational`
GROUP BY
  orders_id
HAVING number>=2
ORDER BY number DESC
```



### 1.2) Column Test - gwz_orders_operational_column-ship

This test checks that none of the columns from gwz_ship are null in the join query. The join is a LEFT JOIN from gwz_orders to gwz_ship, so the test verifies that all orders_id in gwz_orders are present in gwz_ship.


1) Run the gwz_orders_operational_column-ship test and ensure there are no errors.
```
SELECT
  sh.orders_id,
  o.orders_id
FROM `course16.gwz_ship` AS sh
LEFT JOIN `course16.gwz_orders_operational` AS o
  USING (orders_id)
WHERE TRUE
  AND s.orders_id IS NULL
```

2) Compare the nb_row of gwz_ship and gwz_orders_operational. Investigate and explain any differences.

#### gwz_ship —> 167552 gwz_orders_operational —> 167551 Explanation —> orders_id = 1002564 is in gwz_ship but not in gwz_orders_operational


```
SELECT
  COUNT(*)
FROM `course16.gwz_ship`;

SELECT
  COUNT(*)
FROM `course16.gwz_orders_operational`;


SELECT
  sh.orders_id,
  o.orders_id
FROM `course16.gwz_ship` AS sh
LEFT JOIN `course16.gwz_orders_operational` AS o
  USING (orders_id)
WHERE TRUE
  AND o.orders_id IS NULL
```

3) What retention check can you think of to detect this error? Consider how to verify that a metric from gwz_ship is retained in gwz_orders_operational.

Count the rows in gwz_ship, count the rows in gwz_orders_operational and check if it is equal Test ship1 - _gwz_orders_operational_conservation-ship1_—> count the rows in gwz_ship

```
SELECT
  COUNT(*) AS nb
FROM `course16.gwz_ship`;
```

Test ship2 - _gwz_orders_operational_conservation-ship2_—> count the rows in gwz_orders_operational


```
SELECT
  COUNT(*)
FROM `course16.gwz_orders_operational`;
```
