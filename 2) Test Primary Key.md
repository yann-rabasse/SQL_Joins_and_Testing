# Instructions
The challenge is considered complete when you submit the URL of the document you’ve been working on. Ensure the URL is set to share mode and is accessible by the teacher.
If there is no submission option on the exercise, click the I'm done button.
__

## Testing your queries is essential!
As you develop increasingly complex transformation pipelines and collaborate with colleagues, it’s easy to make mistakes. Since data transformation steps build on one another, a single error in a query can impact the entire pipeline and cause it to fail. Therefore, never underestimate the importance of testing your work!

In this exercise, we will create some tests. For now, let’s focus on:

- Ensuring that the primary key is respected → essential
- Verifying that certain columns adhere to a defined format → optional
#### Let’s get started! 

## 1) Basic Data Testing
In the challenge “Data Request - Circle Inventory Management” from the Aggregation, Date & Time Functions module, you created several queries to calculate useful KPIs for the purchasing team based on inventory and sales data. Now, let’s create some tests to ensure your queries return the expected results.

There are several ways to create tests:
- Run a query that must return TRUE or a specific value
- Run two queries and check that the values are similar




1) Open the course15 dataset in your BigQuery project, which you created during the Aggregation, Date & Time Functions day. Ensure that the following tables are saved in this dataset: circle_stock_cat, circle_stock, circle_stock_kpi, circle_sales_daily.

If these tables are not in the course15 dataset, copy them to your BigQuery project from here.



2) Write a test query to check if product_id is a primary key in the circle_stock_cat table. Save the query under the name cc_stock_pk💾. Your saved query will appear at the top of your BigQuery project under the Saved queries section.

Note: To create a test that checks if a column is a primary key, you need to check for duplicate and NULL values within that column. Remember how we identified primary keys using the GROUP BY and HAVING clauses. Your test should return no data (no rows) if the column is indeed a suitable primary key for the table.

```
SELECT
  ### Key ###
  product_id
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock_cat`
GROUP BY
  product_id
HAVING nb>=2
ORDER BY nb DESC
```


3) Create a test query named cc_stock_pk-bis to check if the combination of model, color, and size columns serves as a primary key for the circle_stock table.
```
SELECT
  ### Key ###
  model AS product_model
  ,color AS product_color
  ,size AS product_size
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock`
GROUP BY
  model
  ,color
  ,size
HAVING nb>=2
ORDER BY nb DESC
```


4) Create a test query called cc_stock_column-model_type to validate that model_type has no null values in the circle_stock_kpi table.
```
SELECT
  ### Key ###
  product_id
  ###########
  ,model_type
FROM `course15.circle_stock_kpi`
WHERE
  model_type IS NULL
ORDER BY product_id
```

5) Create a new table named cc_stock_model_type from the circle_stock_kpi table, which will contain aggregated metrics (nb_products, nb_products_in_stock, shortage_rate, total_stock_value) at the model_type level.
Then, create a cc_stock_model_type_pk test query to check if model_type is the primary key of the new cc_stock_model_type table.
```
SELECT
  ### Key ###
  model_type
  ###########
  ,COUNT(in_stock) AS nb_products
  ,SUM(in_stock) AS nb_products_in_stock
  ,ROUND(AVG(1-in_stock),3) AS shortage_rate
  ,SUM(stock_value) AS total_stock_value
FROM `course15.circle_stock_kpi`
GROUP BY model_type
ORDER BY model_type
```
Test - cc_stock_model_type_pk

```
SELECT
  ### Key ###
  model_type
  ###########
  ,count(*) AS nb
FROM `course15.cc_stock_model_type`
GROUP BY
  model_type
HAVING nb>=2
ORDER BY nb DESC
```

6) Write and save a cc_sales_daily_pk test query to check if product_id is the primary key of circle_sales_daily.
```
SELECT
  ### Key ###
  product_id
  ###########
  ,count(*) AS nb
FROM `course15.cc_sales_daily`
GROUP BY
  product_id
HAVING nb>=2
ORDER BY nb DESC
```


7) Create a cc_stock_column-product_name test to ensure that product_name has no null values in circle_stock_kpi.

```
SELECT
  ### Key ###
  product_id
  ###########
  ,product_name
FROM `course15.circle_stock_kpi`
WHERE
  product_name IS NULL
ORDER BY product_id
```


8) Create a cc_stock_column-stock_value test to check that the stock_value in circle_stock_kpi is always positive, 0, or NULL. Did your test return any rows? If it did, the test has failed, indicating there are negative values in stock_value. How can you make the test pass?

Note: To make the test pass, you would need to update the table to correct the values that caused the test to fail.

- Identify the problematic rows: Run a query to find all rows where the stock_value is negative.
- Update the values: Ideally, if stock_value is negative, you would want to set it to NULL.
- Rerun the test: After making the updates, rerun the test query to check if it now passes (i.e., returns no data).

If you are dealing with other conditions in different tests, apply similar corrective actions based on what the test is checking for (e.g., removing duplicates, filling in missing values, etc.).

If you update the circle_stock_kpi table with this rule and rerun the cc_stock_column-stock_value test, does it pass (i.e., return no data)?

```
SELECT
  ### Key ###
  product_id
  ###########
  ,stock_value
FROM `course15.circle_stock_kpi`
WHERE
  stock_value < 0
ORDER BY product_id
```

- Corrected table - circle_stock_kpi

```
SELECT
  ### Key ###
  CONCAT(t.model,"_",t.color,"_",IFNULL(t.size,"no-size")) AS product_id
  ###########
  ,t.model AS product_model
  ,t.color AS product_color
  ,t.size AS product_size
  -- category
  ,CASE
    WHEN REGEXP_CONTAINS(LOWER(t.model_name),'t-shirt') THEN 'T-shirt'
    WHEN REGEXP_CONTAINS(LOWER(t.model_name),'short') THEN 'Short'
    WHEN REGEXP_CONTAINS(LOWER(t.model_name),'legging') THEN 'Legging'
    WHEN REGEXP_CONTAINS(LOWER(REPLACE(t.model_name,"è","e")),'brassiere|crop-top') THEN 'Crop-top'
    WHEN REGEXP_CONTAINS(LOWER(t.model_name),'débardeur|haut') THEN 'Top'
    WHEN REGEXP_CONTAINS(LOWER(t.model_name),'tour de cou|tapis|gourde') THEN 'Accessories'
    ELSE NULL
  END AS model_type
  -- name
  ,t.model_name
  ,t.color_name AS color_description
  ,CONCAT(t.model_name," ",t.color_name,IF(t.size IS NULL,"",CONCAT(" - Taille ",size))) AS product_name
  -- product info --
  ,t.new AS product_new
  -- stock metrics --
  ,t.forecast_stock AS forecasted_stock_level
  ,t.stock AS current_stock_level
  ,IF(t.stock>0,1,0) AS in_stock
  -- value
  ,t.price
  ,IF(t.stock<0,NULL,ROUND(t.stock*t.price,2)) AS stock_value
FROM `course15.circle_stock` AS t
WHERE TRUE
ORDER BY product_id
```









