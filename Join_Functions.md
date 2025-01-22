# Objective
The objective is to join financial data tables to obtain an overview of daily costs and revenue, and to track the business’s profitability over time.

In this exercise, you will:

- Practice joining tables
- Better understand the needs of the finance team

Joins in SQL combine data from two or more tables into a single set by matching rows based on shared columns.

SQL compares each row in one table with every row in another to find matches. When rows match, they are combined into a single set, which can then be filtered, sorted, or manipulated as needed.

To join tables, you must know each table’s primary key and the foreign keys that connect them. Without identifying these keys, you can’t join tables.

### Sources:
Copy the gwz_product table into the new dataset. [Here](# )
Copy the gwz_sales table into the new dataset. [Here](# )
Copy the gwz_ship table into the new dataset. [Here](# )

To help with the task, we’ve summarized the information about the keys:

#### Product
Primary key - products_id
Foreign_key - product_id in gwz_sales

#### Sales
Primary key - orders_id, products_id
Foreign_keys - orders_id in gwz_ship and product_id in gwz_product

#### Ship
Primary key - orders_id
Foreign_key - orders_id in gwz_sales

# 1) Join queries - Building Joins
In the following steps, you’ll join tables to create a financial tracker.

## 1.1) Sales and Product Join
The goal is to set up a financial tracker comparing revenue to costs. To do this, we need to calculate the product margin. Product margin is the difference between the sale price (turnover) and the purchase cost (purchase price of a single product * quantity). We’ll join gwz_sales and gwz_product to calculate the purchase cost and margin.



1) On which key would you join the gwz_sales and gwz_product tables to calculate the purchase cost and margin?
Answer: _We can join gwz_sales and gwz_product on products_id to add purchase_price to gwz_sales. Then, multiply purchase_price by qty to calculate purchase_cost. Finally, subtract purchase_cost from turnover to get the margin.


2) Perform a LEFT JOIN between the gwz_sales and gwz_product tables to add the purchase_price column to the gwz_sales table. Save the result as gwz_sales_margin💾.
```
SELECT
  s.date_date
  ### Key ###
  ,s.orders_id
  ,s.products_id
  ###########
  -- sales columns --
  ,s.turnover
  ,s.qty
  -- product column --
  ,p.purchase_price
FROM `course16.gwz_sales` AS s
LEFT JOIN `course16.gwz_product` AS p ON s.products_id=p.products_id -- USING (products_id)
```


3) Create a test on the primary key for the new gwz_sales_margin table. Save it as gwz_sales_margin_pk💾. A primary key should be a unique identifier—verify that this is the case.
```
SELECT
  ### Key ###
  orders_id
  ,products_id
  ###########
  ,COUNT(*) AS nb
FROM `course16.gwz_sales_margin`
GROUP BY
  orders_id
  ,products_id
HAVING nb>=2
ORDER BY nb DESC
```


4) Add purchase_cost and margin columns to the new gwz_sales_margin table and save the changes.

- To compute the purchase_cost, multiply the quantity by the purchase price.
- To compute the margin, subtract the purchase cost from the turnover.

```
SELECT
  s.date_date
  ### Key ###
  ,s.orders_id
  ,s.products_id
  ###########
  -- qty --
  ,s.qty
  -- revenue --
  ,s.turnover
  -- cost --
  ,p.purchase_price
  ,ROUND(s.qty*p.purchase_price,2) AS purchase_cost
  -- margin --
  ,s.turnover - ROUND(s.qty*p.purchase_price,2) AS margin
FROM `course16.gwz_sales` AS s
LEFT JOIN `course16.gwz_product` AS p ON s.products_id = p.products_id
```

5) Count the number of rows in gwz_sales_margin. Edit your query to change the LEFT JOIN to a RIGHT JOIN. How many rows are returned now? How do you interpret the difference? Discuss with your buddy🧑‍🤝‍🧑 first. 🚨Do not save or update gwz_sales_margin.🚨
```
SELECT
  s.date_date
  ### Key ###
  ,s.orders_id
  ,s.products_id
  ###########
  -- qty --
  ,s.qty
  -- revenue --
  ,s.turnover
  -- cost --
  ,p.purchase_price
  ,ROUND(s.qty*p.purchase_price,2) AS purchase_cost
  -- margin --
  ,s.turnover - ROUND(s.qty*p.purchase_price,2) AS margin
FROM `course16.gwz_sales` AS s
RIGHT JOIN `course16.gwz_product` AS p ON s.products_id = p.products_id
```
Note: We have 1375630 rows with the left join. But we have 1375632 rows with the right join. It means that 2 products_id present in gwz_product do not exist in gwz_sales. They are in the catalog but not in the sales. It means they have never been sold, either because no one is interested or products are recent.


6) To ensure no sales data is lost, select all rows from gwz_sales, even if the product is no longer in gwz_product. Which JOIN —LEFT, RIGHT, or INNER— would be best in this case? Discuss with your buddy👩🏾‍🤝‍👩🏼 how these JOIN types would affect the results.

LEFT JOIN is the best solution here because we want to retain all data from the gwz_sales table. If a product is missing from the catalog (gwz_product), it doesn’t matter since our focus is on tracking sales over time. An INNER JOIN would only select products_id present in both tables, which could lead to losing valuable sales data. We know that 2 products_id in gwz_product don’t exist in gwz_sales (i.e., they are in the catalog but haven't been sold). A RIGHT JOIN wouldn’t be suitable because it could introduce missing values into our sales data..


7) What if some products_id appear in gwz_sales but not in gwz_product? Would this affect margin calculations? How can you quickly detect this issue?

If a product_id is missing in gwz_product, we won’t know the purchase price, and therefore, cannot calculate the margin. To detect this issue, we can create a column test on purchase_price in gwz_sales_margin to ensure it is not null.


8) Create a test for gwz_sales_margin to ensure purchase_price is not NULL, as this would result in missing margin values. The test should return no data if it passes. Save it as gwz_sales_margin_pp_not_null💾.

```
SELECT
   ### Key ###
   orders_id
   ,products_id
   ###########
   ,purchase_price
 FROM `course16.gwz_sales_margin`
 WHERE purchase_price IS NULL
```


## 1.2) Sales and Shipping Join
We want to calculate the operating margin:

#### operating margin = product margin + shipping revenue (shipping_fee) - operating costs (ship_cost + log_cost)



1) Can you join gwz_sales and gwz_ship to calculate the operating margin? On which key? What issue might arise?
We can join gwz_sales and gwz_ship on orders_id, but this is an N:1 relationship. This would duplicate gwz_ship rows, leading to duplicated shipping_fee, ship_cost, and log_cost values for each orders_id. As a result, we wouldn’t be able to calculate the correct operating margin.

To calculate the operating margin, we must first group the gwz_sales table by orders_id. The resulting table can then be joined with gwz_ship to calculate the operating margin.


2) Sum the gwz_sales_margin metrics (qty, turnover, purchase_cost, and margin) for each orders_id. Also, keep the date_date column for later use. Save the result as gwz_orders💾.

```
  SELECT
    date_date
    ### Key ###
    ,orders_id
    ###########
    ,ROUND(SUM(qty),2) AS qty
    ,ROUND(SUM(turnover),2) AS turnover
    ,ROUND(SUM(purchase_cost),2) AS purchase_cost
    ,ROUND(SUM(margin),2) AS margin
  FROM `course16.gwz_sales_margin`
  GROUP BY
    date_date
    ,orders_id
  ORDER BY
    date_date
    ,orders_id
```

3) Add a primary key test for gwz_orders. Save it as gwz_orders_pk💾

```
 SELECT
    ### Key ###
    orders_id
    ###########
    ,COUNT(*) AS nb
  FROM `course16.gwz_orders`
  GROUP BY
    orders_id
  HAVING nb>=2
  ORDER BY nb DESC
```


4) Perform a LEFT JOIN to add the columns from gwz_ship to gwz_orders. Save the result as gwz_orders_operational💾.

```
  SELECT
    o.date_date
    ### Key ###
    ,o.orders_id
    ###########
    -- orders infos --
    ,o.qty
    ,o.turnover
    ,o.purchase_cost
    ,o.margin
    -- ship infos --
    ,sh.shipping_fee
    ,sh.log_cost
    ,sh.ship_cost
  FROM `course16.gwz_orders` AS o
  LEFT JOIN `course16.gwz_ship` AS sh ON o.orders_id = sh.orders_id
  ORDER BY
    date_date
    ,orders_id
```


5) Run two tests on the gwz_orders_operational table:

- A primary key test
- A test to check for no NULL values in the shipping_fee, ship_cost, or log_cost columns

```
SELECT
   ### Key ###
   orders_id
   ###########
   ,COUNT(*) AS nb
 FROM `course16.gwz_orders_operational`
 GROUP BY
   orders_id
 HAVING nb>=2
 ORDER BY nb DESC
```
_Test_- gwz_orders_operationnal_column-ship

```
 SELECT
    ### Key ###
    orders_id
    ###########
    ,shipping_fee
    ,log_cost
    ,ship_cost
 FROM `course16.gwz_orders_operationnal`
 WHERE TRUE
    AND shipping_fee IS NULL
    OR log_cost IS NULL
    OR ship_cost IS NULL
 ORDER BY orders_id
```


6) Add the operational_margin column to gwz_orders_operational and overwrite the table with the new result.

#### Note: operating margin = product margin + shipping revenue (shipping_fee) - operating costs (ship_cost + log_cost)

```
 SELECT
  o.date_date
   ### Key ###
   ,o.orders_id
   ###########
   -- orders infos --
   ,o.qty
   ,o.turnover
   ,o.purchase_cost
   ,o.margin
   -- ship infos --
   ,sh.shipping_fee
   ,sh.log_cost
   ,sh.ship_cost
   -- operationnal margin --
   ,o.margin + sh.shipping_fee - (sh.log_cost + sh.ship_cost) AS operational_margin
 FROM `course16.gwz_orders` AS o
 LEFT JOIN `course16.gwz_ship` AS sh ON o.orders_id = sh.orders_id
 ORDER BY
   date_date
   ,orders_id
```



## 1.3) Aggregation for Daily Financial Monitoring


1) Aggregate the financial KPIs from the gwz_orders_operational table for each day, and order the data by the most recent KPIs. Save the result as gwz_finance_day💾.

```
 SELECT
   ### Key ###
   date_date
   ###########
   -- orders infos --
   ,SUM(qty) AS qty
   ,ROUND(SUM(turnover),0) AS turnover
   ,ROUND(SUM(purchase_cost),0) AS purchase_cost
   ,ROUND(SUM(margin),0) AS margin
   -- ship infos --
   ,ROUND(SUM(shipping_fee),0) AS shipping_fee
   ,ROUND(SUM(log_cost),0) AS log_cost
   ,ROUND(SUM(ship_cost),0) AS ship_cost
   -- operationnal margin --
  ,ROUND(SUM(operational_margin),0) AS operational_margin
 FROM `course16.gwz_orders_operationnal`
 GROUP BY
   date_date
 ORDER BY
   date_date DESC
```

2) Add the following financial KPIs to the gwz_finance_day table and overwrite it with the new results:

- Number of transactions
- Average basket value, for average basket, you have 2 methods:
  - average turnover
  - sum(turnover) / count(orders_id)
 
```
 SELECT
   ### Key ###
   date_date
   ###########
   -- orders infos --
   ,COUNT(orders_id) AS nb_transaction
   ,SUM(qty) AS qty
   ,ROUND(SUM(turnover),0) AS turnover
   ,ROUND(AVG(turnover),1) AS average_basket
   ,ROUND(SUM(turnover)/COUNT(orders_id),1) AS average_basket_bis
   ,ROUND(SUM(purchase_cost),0) AS purchase_cost
   ,ROUND(SUM(margin),0) AS margin
   -- ship infos --
   ,ROUND(SUM(shipping_fee),0) AS shipping_fee
   ,ROUND(SUM(log_cost),0) AS log_cost
   ,ROUND(SUM(ship_cost),0) AS ship_cost
   -- operationnal margin --
   ,ROUND(SUM(operational_margin),0) AS operational_margin
 FROM `course16.gwz_orders_operationnal`
 GROUP BY
   date_date
 ORDER BY
   date_date DESC
```
