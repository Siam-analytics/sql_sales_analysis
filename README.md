
🗃️ Fast Food Sales Analysis with SQL
A structured SQL project analyzing fast food store sales to uncover trends in product popularity, seasonal performance, and customer preferences across gender and time.

## 📚 Table of Contents

- Project Overview
- Database Schema
- Tools Used
- Business Questions
- Sample Queries
- Key Insights
- How to Use
- License

## 📌 Project Overview

This project showcases how SQL can be used to perform data analysis for a fast food outlet, 'Balaji Fast Food'. The analysis includes:

- Total Revenue
- Total quantity sold
- Total orders
- Monthly Revenue trends  
- Seasonal trends of items
- Running sum for progress analysis
- Popularity distribution of items among genders
- Item contribution to whole
- Popular payment methods

## 🧱 Database Schema

**Database: fast_food**
**Table: food**
Columns:
- order_id ()
- date (Date of order)
- item_name (Name of the item)
- item_type(Category of food)
- item_price (Price per unit)
- quantity (Number of items ordered)
- transaction_amount (Total amount)
- transaction_type (Payment methods)
- received_by (Gender of customers)
- time_of_sale (Time of the day when the order was placed)

## 🛠 Tool Used

- **MySQL Workbench**  

## 💼 Business Questions

1. What is the total revenue?
2. What is the total quantity sold?
3. What is the total number of orders?
4. What are the item popularity trends across different genders?
5. How does each item's monthly revenue compare to its overall average monthly sales?
6. What are the most effective time slots for maximizing sales of specific items?
7. Which payment method is most preferred by customers?
8. What is each item's annual sales performance and its share of total yearly sales?
9. In which season does each item generate the highest revenue?
10. which month generated the most revenue for each year?

## 🧾 Sample Queries

```sql
-- Total revenue
SELECT 
    SUM(transaction_amount) AS total_sales
FROM
    food ;
```

```sql
-- Total quantity sold
SELECT 
    SUM(quantity) AS total_sold
FROM
    food ;
```

```sql
-- Total orders
SELECT 
    COUNT(order_id) AS total_orders
FROM
    food ;
```

```sql
-- Popularity of item between the genders
SELECT 
    received_by,
    item_name, 
    SUM(quantity) AS total_bought
FROM
    food
GROUP BY received_by , item_name
ORDER BY received_by , SUM(quantity) DESC ;
```

```sql
-- Monthly Item Performance: Sales vs. Annual Average with Performance Flag
WITH monthly_total_amount AS (
    SELECT 
        item_name, 
        YEAR(date) AS year, 
        MONTH(date) AS month, 
        SUM(transaction_amount) AS total_amount
    FROM food
    GROUP BY YEAR(date), MONTH(date), item_name
),
monthly_performance AS (
    SELECT 
        *, 
        ROUND(AVG(total_amount) OVER(PARTITION BY item_name, year), 2) AS monthly_avg, 
        total_amount - ROUND(AVG(total_amount) OVER(PARTITION BY item_name, year), 2) AS avg_diff,
        CASE 
            WHEN total_amount - ROUND(AVG(total_amount) OVER(PARTITION BY item_name, year), 2) < 0 THEN 'below_avg'
            WHEN total_amount - ROUND(AVG(total_amount) OVER(PARTITION BY item_name, year), 2) = 0 THEN 'avg'
            ELSE 'above_avg'
        END AS sales_comp_with_avg
    FROM monthly_total_amount
),
performance_pct AS (
    SELECT 
        item_name, 
        year,
        sales_comp_with_avg,
        CONCAT(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY item_name, year), ' %') AS pct_status
    FROM monthly_performance
    GROUP BY item_name, year, sales_comp_with_avg
)

SELECT *
FROM performance_pct
ORDER BY item_name, year, sales_comp_with_avg;
```

```sql
-- Sales Distribution: Item Categories by Time Period
SELECT 
    COUNT(order_id) total_orders, item_type, 
    time_of_sale
FROM
    food
GROUP BY item_type , time_of_sale
ORDER BY item_type , total_orders DESC ;
```

```sql
-- Best payout method
SELECT 
    transaction_type, COUNT(transaction_type) as total
FROM
    food
GROUP BY transaction_type
ORDER BY total DESC ;
```

```sql
-- Contribution of each item to the total yearly sales
WITH cte1 as (
SELECT 
	YEAR (date) as year, item_name, SUM(quantity) AS total_sales
FROM food
GROUP BY YEAR (date), item_name
ORDER BY year, total_sales DESC
)

SELECT  
	*, 
	SUM(total_sales) OVER (partition by year) as total_yearly_sales, 
	CONCAT((total_sales/SUM(total_sales) OVER(partition by year)*100), ' %')
	AS total_yearly_sales
FROM cte1 ;
```

```sql
-- Seasonal Sales of each item
WITH seasonal_segmentation as (
SELECT 
	item_name, 
	transaction_amount, 
	CASE WHEN month(date) = 2 THEN 'spring'
		WHEN month(date) between 3 and 5 THEN 'summer'
		WHEN month(date) between  6 and 9 THEN 'Monsoon'
		WHEN month(date) between 10 and 11 THEN 'Pre-Winter'
	ELSE 'Winter'
	END AS season
FROM food
ORDER BY item_name DESC 
)

SELECT item_name, season, sum(transaction_amount) AS total_amount
FROM seasonal_segmentation
group by item_name, season
order by item_name, total_amount DESC;
```

```sql
-- Monthly sales_amount
SELECT 
    YEAR(date) AS year,
    MONTH(date) AS month,
    SUM(transaction_amount) AS sales_amount
FROM
    food
GROUP BY YEAR(date) , MONTH(date)
ORDER BY YEAR(date) , sales_amount DESC ;
```


## 📊 Key Insights

- Panipuri emerged as the most popular item among men with 685 total orders, whereas Cold Coffee was the top choice among women with 687 orders.
- In terms of annual sales, Cold Coffee led in 2022, accounting for 16.35% of the year's total sales. In 2023, Frankie took the lead, contributing 18.84% to the yearly sales.
- During the summer season, Aalopuri and Cold Coffee generated the highest revenue, while all other items reached peak revenue during the monsoon season.
- The percentage of top-performing items—Cold Coffee, Frankie, Panipuri, and Sandwich—classified as above average increased from approximately 45% in 2022 to 66.6% in 2023.
- Aalopuri showed notable improvement, shifting from being mostly below average in 2022 to predominantly above average in 2023. Conversely, Vadapav declined, with its above-average performance dropping from 66% to 33% in 2023.
  
## ▶️ How to Use

1.Create a Database: e.g. 
```sql
CREATE DATABASE fast_food;
```
2.Create a Table: e.g.
```sql
CREATE TABLE sales_data (
    order_id INT PRIMARY KEY,
    date DATE,
    item_name VARCHAR(50),
    item_type VARCHAR(20),
    item_price DECIMAL(10, 2),
    quantity INT,
    transaction_amount DECIMAL(10, 2),
    transaction_type VARCHAR(20),
    received_by VARCHAR(10),
    time_of_sale VARCHAR(20)
);
```
3.To Import:
- Select & Right click on 'food' table
- Click the option 'Table Data Import Wizard',
- browse your csv location, select the file and click next until import

## 🧹 Data Cleaning
```sql
UPDATE food
SET transaction_type = ''
WHERE transaction_type = 'Other';
```

## 📜 License

This project is licensed under the MIT License.
