## Objective
In the previous exercise, Greenweez Finance - Join, you created a financial report that includes operational costs. Now, we want to add advertising costs to the report. Let’s get started!

## 1) Ads Cost

### 1.1) Data import

Copy the gwz_campaign table into the course16 dataset in your BigQuery project.


### 1.2) Include Ad Cost in the Financial Report
The goal is to add the ads_cost column to the gwz_finance_day table created in the previous exercise and calculate ads_margin as operational_margin - ads_cost.



1) Determine the primary key of the gwz_campaign table and create a test for it.
```
SELECT
  ### Key ###
  date_date
  ,campaign_name
  ###########
  ,COUNT(*) AS nb
FROM `course16.gwz_campaign`
GROUP BY
  date_date
  ,campaign_id
HAVING nb>=2
ORDER BY nb DESC
```


2) Can you directly join the gwz_finance_day and gwz_campaign tables? What issue might arise? Consider a strategy to overcome this issue.

We could join gwz_campaign and gwz_finance_day on date_date. However, this is an N:1 relationship. Thus, the rows of gwz_campaign would be duplicated resulting in duplicated ads_cost values. This would lead to incorrect ads_margin values. As we want to calculate ads_margin, we must first aggregate gwz_campaign on date_date to ensure that there would be a single value for each date. Then we could join the two tables on date_date and calculate the correct ads_margin.


3) Aggregate gwz_campaign by date_date and save the result as gwz_campaign_day.

```
SELECT
  ### Key ###
  date_date
  ###########
  ,SUM(ads_cost) AS ads_cost
FROM `course16.gwz_campaign`
GROUP BY
  date_date
ORDER BY
  date_date
```


4) Perform a LEFT JOIN between gwz_finance_day and gwz_campaign_day to add the ads_cost column to the financial report. Save the result as gwz_finance_ads.

```
SELECT
  ### Key ###
  fi.date_date
  ###########
  -- orders infos --
  ,fi.nb_transaction
  ,fi.qty
  ,fi.turnover
  ,fi.average_basket
  ,fi.purchase_cost
  ,fi.margin
  -- ship infos --
  ,fi.shipping_fee
  ,fi.log_cost
  ,fi.ship_cost
  -- operational margin --
  ,fi.operational_margin
  -- ads --
  ,c.ads_cost
FROM `course16.gwz_finance_day` AS fi
LEFT JOIN `course16.gwz_campaign_day` AS c
  ON fi.date_date = c.date_date
```

5) Add an ads_margin column to the gwz_finance_ads table and save the updated table under the same name.

```
SELECT
  ### Key ###
  fi.date_date
  ###########
  -- orders infos --
  ,fi.nb_transaction
  ,fi.qty
  ,fi.turnover
  ,fi.average_basket
  ,fi.purchase_cost
  ,fi.margin
  -- ship infos --
  ,fi.shipping_fee
  ,fi.log_cost
  ,fi.ship_cost
  -- operational margin --
  ,fi.operational_margin
  -- ads --
  ,c.ads_cost
  ,fi.operational_margin-c.ads_cost AS ads_margin
FROM `course16.gwz_finance_day` AS fi
LEFT JOIN `course16.gwz_campaign_day` AS c ON fi.date_date = c.date_date
```

6) Aggregate the gwz_finance_ads table by month to analyze the indicators for July, August, and September 2021. Calculate the following KPIs and save the results in the gwz_finance_ads_month table:

- turnover
- nb_transaction
- average_basket (weighted by the number of orders each day)
- margin_ord (average margin per order)
- margin_percent (average margin percentage)
- shipping_fee_ord (average shipping fee per order)
- operational_cost_ord (average cost of shipping and logistics per order)
- ads_cost_ord (average ads cost per order)
- ads_margin_ord (average ads margin per order)


![05-Gwz-Finance-Include-Ads-Cost-asset-1-Untitled](https://github.com/user-attachments/assets/8948fa80-4ee9-4a7c-b2c8-00505c8d7fbe)


#### What insights can you draw from the new financial report?


```
   SELECT
     ### Key ###
     EXTRACT(MONTH FROM date_date) AS month
     ###########
     -- orders infos --
     ,SUM(nb_transaction) AS nb_transaction
     ,SUM(turnover) AS turnover
     ,ROUND(SUM(turnover)/SUM(nb_transaction),1) AS average_basket
     -- margin
     ,ROUND(SUM(margin)/SUM(nb_transaction),1) AS margin_ord
     ,ROUND(SUM(margin)/SUM(turnover)*100,1) AS margin_percent
     -- ship infos --
     ,ROUND(SUM(shipping_fee)/SUM(nb_transaction),1) AS shipping_fee_ord
     ,ROUND(SUM(log_cost+ship_cost)/SUM(nb_transaction),1) AS ship_cost_ord
     -- ads --
     ,ROUND(SUM(ads_cost)/SUM(nb_transaction),1) AS ads_cost_ord
     ,ROUND(SUM(ads_margin)/SUM(nb_transaction),1) AS ads_margin_ord
     ,SUM(ads_margin) AS ads_margin
   FROM `course16.gwz_finance_ads_day`
   WHERE date_date BETWEEN '2021-07-01' and '2021-09-30'
   GROUP BY
     month
   ORDER BY
     month DESC
```



There is a decrease in the number of transactions, but an increase in the average basket size, keeping the turnover relatively stable over the 3 months. However, there is a significant drop in both total ads margin and average ads margin per order. This can be attributed to two main factors:

An increase in ads cost per order over the last 3 months
A substantial 2.5% decrease in margin percentage in September
The solution for ads cost is to limit and control the budget. For margin percentage, the decline may be due to either high levels of promotion or significant changes in selling prices that impacted the margin.
