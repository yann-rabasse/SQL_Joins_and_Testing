# Takeaways
Data coming from different data sources may have different levels of details, i.e. data granularity
Joining tables of different granularity can cause severe calculation errors
Identifying the appropriate JOIN type, and data granularity will help prevent these errors
Always triangulate the results of JOINs with calculations from the raw tables as a sanity check that JOINs have been done appropriately.
Case Study
You’ve been hired as a consultant for a general store at the airport to understand how foot traffic affects the store revenue.

To do this, you’ve been provided with multiple CSV files by your client.

## Case Study

![01-CaseStudy](https://github.com/user-attachments/assets/70d04b28-53df-4428-8b8e-8bb7bcdedc5b)

### Data: Visitor
[Download Here](#https://docs.google.com/spreadsheets/d/1GvXefYfRmbelaGnQhcCwpvsu2vs5WNcKGNtAMOrXT0Y/edit?gid=1313584518#gid=1313584518)

The airport has hired staff members to count the number of people entering and exiting the building.

At any given time, the main entrance / exit point at the airport is manned by two staff members. One counting the number of people entering the airport, the other counting the number of people exiting the airport.

However, data is only recorded into a spreadsheet once a day.

#### Visitors Sample

![02-Visitor](https://github.com/user-attachments/assets/04396943-6a70-45a9-857f-6ce11b4dbb99)


### Data: Sales

[Download Here](#https://docs.google.com/spreadsheets/d/1zSqIGIooYoCTqFnArUG6Y950Syv49xYwqtuHnRSNWzA/edit?gid=1162345981#gid=1162345981)

Sales data were exported directly from the point of sales (POS) system. As such, it records data at a transaction level.

Each row represents one sales transaction that has been successfully paid for.

#### Sales Sample

![02-Sales](https://github.com/user-attachments/assets/55b3db38-b407-4c2f-8c0b-3a8a41c7a556)


## Task 1: Understanding data granularity

Before we start writing queries, let’s first understand the granularity of our tables.

![03-Task1](https://github.com/user-attachments/assets/5c4f9b08-5ed4-4da5-947c-c3e0eba47cf4)


#### Prompt

What does each row in this table represent?

The goal here is to encourage students to discuss their understanding of data granularity, using the example datasets provided.

This is an opportunity for Teachers to guide discussions, helping students recognize that sales data are recorded at a transactional level, while visitor data are recorded at a daily level, meaning they have different granularities.

## Task 2: Write an analysis plan

To join our two tables correctly, we first need to align them to the same level of granularity.

Before diving into queries, let’s outline an analysis plan. Consider the following:

Our data has different levels of granularity. How can we adjust the tables to match the same granularity?
Which type of join would be most appropriate to relate the two tables?

#### Prompt

The goal is to encourage students to discuss how to manage different levels of granularity.
There are various ways to approach this problem. One option is to align the data granularity at the daily level through aggregation, a concept covered in the previous lesson.

#### Analysis plan
Step 1. …

Step 2. …

Step 3. …

- Example output

1) Summarise sales by day
2) Use LEFT join between Sales by day and Visitor data using day as joining key
3) Plot result on a chart


## Task 3: Write your queries

Now it’s time to write the queries that implement your analysis plan.

### Step 1: Bringing data to daily granularity
As we haven’t covered subquery yet at this stage, we’re going to save the output of the first couple of steps as temporary tables.

Write a query to align the data to the same granularity, according to your analysis plan.
Save the output of this query as a temporary table.

```
SELECT
  DATE(date) AS `date`
  ,ROUND(SUM(amount),2) AS `sales`
FROM `recapjoin.sales`
GROUP BY DATE(date)
ORDER BY DATE(date)

# Save as tmp_sales
```
### Result

<img width="311" alt="2wpicy964qvx7x36v7ruydxtgry6" src="https://github.com/user-attachments/assets/5a7103d5-2fee-45b8-94a9-97325901ddb4" />

### Step 2: Joining tables

Now that our data is at the same granularity, we’re ready to join the tables.

Write a query to join the visitors and daily sales tables.
Save the output of this query as a temporary table.

```
SELECT
  sales.date
  ,sales.sales
  ,traffic.traffic
FROM `recapjoin.tmp_sales` sales
LEFT JOIN `recapjoin.traffic` traffic ON sales.date = traffic.date

# Save as tmp_joined
```
### Result

<img width="433" alt="59fn10564paggihwora44bnzi4ie" src="https://github.com/user-attachments/assets/0309fbce-a78e-4304-85ce-b4781cea8e93" />


### Step 3: Triangulate
As responsible data analysts, we need to ensure our JOINS are correct. One way to do this is by triangulating the results from the raw data with the processed data.

Calculate total sales from the raw sales table.
Calculate total sales from the joined table.
Do the total sales from both queries match? Why or why not?

💡 *Here is a good checkpoint to let the students assist with identifying and reviewing the analysis plan to address the issue.

To prompt the students, teachers can do a SELECT * from both tables and ask “What’s the difference in structure.

We’re looking for students to identify that the granularity of the visitors table is by day and also by exit/entry types. As we have two types of traffic, sales have been double counted.*

```
SELECT
  SUM(amount)
FROM `recapjoin.sales`

# e.g. returns 355,466.27
```

```
SELECT
  SUM(sales)
FROM `recapjoin.tmp_joined`

# e.g. returns 710,932.52
```

### Step 4: Rewrite SQL queries
Let’s now review our analysis plan and rewrite any queries that need adjustment.

Identify which parts of the analysis plan need to be revised.
Rewrite the necessary SQL queries.
Triangulate your queries to ensure that total sales from the raw data match the total sales after the JOIN.

```
SELECT
  date
  ,SUM(traffic) AS `traffic`
FROM `recapjoin.traffic`
GROUP BY date

# Save as tmp_traffic
```

```
SELECT
  sales.date
  ,sales.sales
  ,visitors.amount
FROM tmp_sales sales
LEFT JOIN tmp_visitors visitors ON sales.date = visitors.date

# Save as tmp_joined
```

```
SELECT
  SUM(amount)
FROM `recapjoin.sales`

# e.g. returns 355,466.27
```

```
SELECT
  SUM(sales)
FROM `recapjoin.tmp_joined`

# e.g. returns 355,466.26 (due to rounding)
```















