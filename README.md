# -Data-Analysis-SQL-Project-


# 🛒 Walmart Sales Data Analysis (SQL Project)
## Project Overview

This project analyzes Walmart sales data using PostgreSQL.
The goal is to answer business questions related to products, sales, branches, and customers.

Dataset: <a href=https://https://www.kaggle.com/datasets/antaesterlin/walmart-commerce-data> Dataset: Kaggle Walmart Commerce Data </a> <br />

# 📂 Data Setup 

## 🛢️ -- Create Database
```
CREATE DATABASE walmartsalesdata;

-- Drop Table if already exists
DROP TABLE IF EXISTS walmart_sales;

-- Create Table
CREATE TABLE walmart_sales (
    invoice_id VARCHAR(30) PRIMARY KEY,
    branch VARCHAR(20),
    city VARCHAR(30),
    customer_type VARCHAR(30),
    gender VARCHAR(30),
    product_line VARCHAR(30),
    unit_price NUMERIC(6,2) CHECK (unit_price >= 0),
    quantity INT,
    vat NUMERIC(8,2),
    total NUMERIC(12,2),
    dtme DATE,
    tme TIME,
    payment_method VARCHAR(30),
    cogs NUMERIC(12,2),
    gross_margin_pct NUMERIC(12,2),
    gross_income NUMERIC(12,2),
    rating NUMERIC(3,1) CHECK (rating BETWEEN 0 AND 10),
    time_of_day VARCHAR(30),
    day_name VARCHAR(30),
    month_name VARCHAR(30)
);
```
## 📂  -- Import Data
```
COPY walmart_sales (
    "invoice_id","branch","city","customer_type","gender","product_line",
    "unit_price","quantity","vat","total","dtme","tme","payment_method","cogs",
    "gross_margin_pct","gross_income","rating","time_of_day","day_name","month_name")
FROM 'D:\sql data\WalmartSQL repository.csv'
DELIMITER ';' CSV HEADER QUOTE '"';
```

## -- Alter Columns
```
ALTER TABLE walmart_sales RENAME COLUMN product_line TO product_type;
ALTER TABLE walmart_sales RENAME COLUMN dtme TO sale_date;
ALTER TABLE walmart_sales RENAME COLUMN tme TO sale_time;
```
# 🛒 Product Analysis

## 1️⃣ Which product types generate the highest total revenue?
```
SELECT product_type, 
       SUM(total) AS revenue  
FROM walmart_sales
GROUP BY product_type
ORDER BY revenue DESC
LIMIT 1;
```
## 2️⃣ Which product types generate the highest gross income (profit)?
```
SELECT product_type,
       SUM(gross_income) AS total_income  
FROM walmart_sales
GROUP BY product_type
ORDER BY total_income DESC
LIMIT 1;
```
## 3️⃣ What is the average unit price and quantity sold for each product type?
```
SELECT product_type,
       ROUND(AVG(unit_price),2) AS avg_unit_price,
       SUM(quantity) AS quantity_sold
FROM walmart_sales
GROUP BY product_type
ORDER BY quantity_sold DESC;
```
## 4️⃣ Which product type is most popular in each city?
```
SELECT city, product_type, total_sold
FROM (
    SELECT city, product_type,
           SUM(quantity) AS total_sold,
           RANK() OVER (PARTITION BY city ORDER BY SUM(quantity) DESC) AS rnk
    FROM walmart_sales
    GROUP BY city, product_type
) ranked
WHERE rnk = 1
ORDER BY city;
```
## 5️⃣ Which branch sells the highest quantity of each product type?
```
SELECT product_type, branch, total_sold
FROM (
    SELECT product_type,
           branch,
           SUM(quantity) AS total_sold,
           ROW_NUMBER() OVER (PARTITION BY product_type ORDER BY SUM(quantity) DESC) AS rn
    FROM walmart_sales
    GROUP BY product_type, branch
) t
WHERE rn = 1;
```
## 6️⃣ What are the top 3 best-selling product types in terms of revenue and quantity?
```
SELECT product_type, 
       SUM(total) AS total_revenue,
       SUM(quantity) AS total_quantity
FROM walmart_sales
GROUP BY product_type
ORDER BY total_revenue DESC, total_quantity DESC
LIMIT 3;
```
## 7️⃣ Which product type has the highest average rating from customers?
```
SELECT product_type, 
       ROUND(AVG(rating),2) AS average_rating
FROM walmart_sales
GROUP BY product_type
ORDER BY average_rating DESC
LIMIT 1;
```
## 8️⃣ Which product type has the lowest profit margin?
```
SELECT product_type,
       MIN(gross_margin_pct) AS min_margin
FROM walmart_sales
GROUP BY product_type
ORDER BY min_margin ASC
LIMIT 1;
```
## 9️⃣Retrieve each product_type and add a column product_category, indicating 'Good' or 'Bad,'based on whether its sales are above the average.
```
ALTER TABLE walmart_sales
ADD COLUMN product_category VARCHAR(30);

UPDATE walmart_sales
SET product_category =

CASE 
    WHEN total >= (SELECT AVG(total) FROM walmart_sales) THEN 'Good'
	ELSE 'Bad'
END;
```
# 💰 Sales Analysis
## 1️⃣ What is the total number of transactions, total sales, total gross income, and average rating overall?

```SELECT 
      COUNT(invoice_id) AS total_number_of_transactions, 
      SUM(total) AS total_sales, 
      SUM(gross_income) AS total_gross_income,
      ROUND(AVG(rating),2) AS average_rating
FROM walmart_sales;
```
## 2️⃣ What is the monthly trend of sales and profit?
```
SELECT month_name,
       SUM(total) AS monthly_sales,
       SUM(gross_income) AS monthly_profit
FROM walmart_sales
GROUP BY month_name
ORDER BY monthly_sales DESC;
```
## 3️⃣ How do sales differ by day of the week?
```
SELECT day_name,
       SUM(total) AS total_sales,
       SUM(gross_income) AS total_profit,
       COUNT(invoice_id) AS transactions
FROM walmart_sales
GROUP BY day_name
ORDER BY CASE day_name
           WHEN 'Sunday'    THEN 1
           WHEN 'Monday'    THEN 2
           WHEN 'Tuesday'   THEN 3
           WHEN 'Wednesday' THEN 4
           WHEN 'Thursday'  THEN 5
           WHEN 'Friday'    THEN 6
           WHEN 'Saturday'  THEN 7
         END;
```
## 4️⃣ Which time of day (morning, afternoon, evening) has the highest sales?
```
ALTER TABLE walmart_sales
ADD COLUMN time_of_day VARCHAR(30);

UPDATE walmart_sales
SET time_of_day = 
    CASE
        WHEN sale_time BETWEEN '00:00:00' AND '12:00:00' THEN 'morning'
        WHEN sale_time BETWEEN '12:01:00' AND '16:00:00' THEN 'afternoon'
        ELSE 'evening'
    END;

SELECT time_of_day, SUM(total) AS total_sales 
FROM walmart_sales
GROUP BY time_of_day
ORDER BY total_sales DESC
LIMIT 1;
```
## 5️⃣ What is the distribution of payment methods (Cash, Card, Ewallet, etc.)?
```
SELECT payment_method,
       ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (),2) AS percentage_share
FROM walmart_sales
GROUP BY payment_method;
```
## 6️⃣ Which branch and city generates the most sales and profit?
```
SELECT city, branch,
       SUM(total) AS total_sales,
       SUM(gross_income) AS profit
FROM walmart_sales
GROUP BY city, branch
ORDER BY total_sales DESC
LIMIT 1;
```
## 7️⃣ Which month generated the highest revenue and profit overall?
```
SELECT month_name,
       SUM(total) AS total_sales,
       SUM(gross_income) AS profit
FROM walmart_sales
GROUP BY month_name
ORDER BY total_sales DESC
LIMIT 1;
```
## 8️⃣ Number of sales made in each time of the day per weekday?
```
SELECT day_name,
       COUNT(*) FILTER (WHERE time_of_day = 'morning')   AS morning_sales,
       COUNT(*) FILTER (WHERE time_of_day = 'afternoon') AS afternoon_sales,
       COUNT(*) FILTER (WHERE time_of_day = 'evening')   AS evening_sales,
       COUNT(*) AS no_of_sales
FROM walmart_sales
WHERE day_name NOT IN ('Saturday', 'Sunday')
GROUP BY day_name
ORDER BY day_name;
```
 # 👥 Customer Analysis 
 
## 1️⃣ Which customer type (Member / Normal) spends more money?
```
SELECT customer_type,
       SUM(total) AS total_sales 
FROM walmart_sales
GROUP BY customer_type
ORDER BY total_sales DESC;
```
## 2️⃣ Do males or females spend more overall?
```
SELECT gender,
       SUM(total) AS total_sales 
FROM walmart_sales
GROUP BY gender
ORDER BY total_sales DESC
LIMIT 1;
```
## 3️⃣ Which product types are most popular by gender?
```
WITH gender_product_sales AS (
    SELECT gender,
           product_type,
           SUM(quantity) AS total_sold
    FROM walmart_sales
    GROUP BY gender, product_type
)
SELECT gender, product_type, total_sold
FROM (
    SELECT gender, product_type, total_sold,
           RANK() OVER (PARTITION BY gender ORDER BY total_sold DESC) AS rnk
    FROM gender_product_sales
) ranked
WHERE rnk = 1
ORDER BY gender;
```
## 4️⃣ Which customer type gives the highest average rating?
```
SELECT customer_type,
       ROUND(AVG(rating),2) AS average_rating 
FROM walmart_sales
GROUP BY customer_type
ORDER BY average_rating DESC
LIMIT 1;
```
## 5️⃣ What is the average rating by payment method?
```
SELECT payment_method,
       ROUND(AVG(rating),2) AS average_rating 
FROM walmart_sales
GROUP BY payment_method
ORDER BY average_rating DESC;
```
## 6️⃣ Which gender + customer type combination spends the most?
```
SELECT gender, customer_type,
       SUM(total) AS total_sales 
FROM walmart_sales
GROUP BY gender, customer_type
ORDER BY total_sales DESC
LIMIT 1;
```
## 7️⃣ What is the distribution of high-value transactions (>300 total) by branch?
```
SELECT branch, 
       COUNT(invoice_id) AS no_of_transactions
FROM walmart_sales
WHERE total > 300
GROUP BY branch
ORDER BY no_of_transactions DESC;
```
## ⚙️ Tools & Technologies
 <ul>
<li>PostgreSQL</li> 
<li>Kaggle Walmart Commerce Data</li> 
<li>Data Cleaning & Transformation with SQL</li> 
<li>Window Functions for advanced analysis</li> 
</ul>

## 📌 Key Insights
 <ul>
<li>Identified most popular product types by city, branch, and gender.</li> 
<li>Found lowest margin products impacting profitability.</li> 
<li>Analyzed monthly sales & profit trends.</li> 
<li>Measured payment method distribution.</li> 
<li>Highlighted branch contributions to overall sales.</li> 
</ul>

# Feedback & Collaboration
  Have suggestions or ideas to improve? <br />
  Open an issue or fork this project and contribute. I’m always open to feedback, improvements, or collaboration on data-driven projects!

# Let’s Connect
<ul> <li> LinkedIn: https://www.linkedin.com/in/suraj-kumar-785133243/ <br /> </li>
 <li> Email: <a href="mailto:9kumarsuraj@gmail.com">9kumarsuraj@gmail.com</a>
 </li> </ul>

<h3>Project by: Suraj Kumar </h3>
