## Instructions
The challenge is considered complete when you submit the URL of the document you’ve been working on. Ensure the URL is set to share mode and is accessible by the teacher.

If there is no submission option on the exercise, click the I'm done button.

The objective of this exercise is to learn basic SQL join operations by combining tables from a product catalog.

The green_catalog dataset consists of several product-related tables:

- green_product: Base table representing the product catalog
- green_pdt_segment: Provides the segment of each product
- green_promo: Information about promotions
- green_price: Information about product prices
- green_stock: Information about product stock
- green_categories: Information about product categories
- green_sales: Information about the quantity of products sold in the last 3 months
Schema

![01-Greenweez-Catalog-Basic-Join-asset-1-Untitled](https://github.com/user-attachments/assets/84a6be2a-d93e-417e-a4ee-386a3bbb97bf)


## 1) Join Queries - Building Joins


1) Join the green_product and green_pdt_segment tables to return the pdt_name and pdt_segment columns. Choose an INNER JOIN based on the schema and analysis goals. Specify the column (i.e.products_id) to join ON.
```
SELECT
  p.pdt_name
  ,sgt.pdt_segment
FROM `green_catalog.green_product` AS p
INNER JOIN `green_catalog.green_pdt_segment` AS sgt ON p.products_id = sgt.products_id
```


2) Add stock information to the main product details by joining the green_product and green_stock tables. Use alias names for each table so that the SELECT clause can unambiguously identify which table a specific column comes from.
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  ,p.products_status
  ,p.categories_id
  ,p.promo_id
  -- stock table
  ,st.stock
  ,st.stock_forecast
FROM `green_catalog.green_product` AS p
INNER JOIN `green_catalog.green_stock` AS st ON p.products_id = st.pdt_id
```


3) SELECT products_id, pdt_name, promo_name, and promo_pourcent from the green_product and green_promo tables. Ensure the result includes product rows without a promotion. Experiment with different JOIN types to return all products, both with and without promotions.
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  -- promo table
  ,pr.promo_name
  ,pr.promo_pourcent
FROM `green_catalog.green_product` AS p
LEFT JOIN `green_catalog.green_promo` AS pr ON p.promo_id = pr.promo_id
```


4) Add price information from green_price to green_product.
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  ,p.products_status
  ,p.categories_id
  ,p.promo_id
  -- price table
  ,pr.pd_cat
  ,pr.ps_cat
FROM `green_catalog.green_product` AS p
INNER JOIN `green_catalog.green_price` AS pr USING (products_id)
```


5) Return the products_id and pdt_name with all their associated category information.
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  ,p.categories_id
  -- price table
  ,c.category_1
  ,c.category_2
  ,c.category_3
FROM `green_catalog.green_product` AS p
INNER JOIN `green_catalog.green_categories` AS c USING (categories_id)
```


6) Return the products_id, pdt_name, and their qty sold in the last 3 months. Include all products from green_product, even if they were not sold in the last 3 months. (Remember, the green_sales table only contains sales data from the past 3 months.)
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  -- price table
  ,IFNULL(s.qty,0) AS qty
FROM `green_catalog.green_product` AS p
LEFT JOIN `green_catalog.green_sales` AS s ON p.products_id = s.pdt_id
```
- Use the left join to avoid losing products that have not been sold in the last 3 months.
- If there are no rows in green_sales, it means that the sales are equal to 0 over the last 3 months. We use the IFNULL function to assign 0 to the qty column instead of NULL.

7) Return the same result by using RIGHT JOIN.
```
SELECT
  -- product table --
  p.products_id
  ,p.pdt_name
  -- price table
  ,s.qty
FROM `green_catalog.green_sales` AS s
RIGHT JOIN `green_catalog.green_product` AS p ON p.products_id = s.pdt_id
```













