
# 🗃️ E-Commerce Sales Analysis with SQL

_A structured SQL project analyzing an online retail database to uncover trends in revenue, customer behavior, and regional performance._

## 📚 Table of Contents

- [Project Overview](#project-overview)  
- [Database Schema](#database-schema)  
- [Tools Used](#tools-used)  
- [Business Questions](#business-questions)  
- [Sample Queries](#sample-queries)  
- [Key Insights](#key-insights)  
- [How to Use](#how-to-use)  
- [License](#license)

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
	YEAR(date) as year, 
	MONTH(date) as month, 
	SUM(transaction_amount) as total_amount
FROM food
GROUP BY year(date), month(date), item_name
ORDER BY year, item_name
)
SELECT 
	*, 
	ROUND(AVG(total_amount) OVER(partition by item_name, year),2) as monthly_avg, 
	total_amount-round(AVG(total_amount) OVER(partition by item_name, year),2) as avg_diff,
CASE WHEN total_amount-round(AVG(total_amount) OVER(partition by item_name, year),2)<0 THEN 'bad'
     WHEN total_amount-round(AVG(total_amount) OVER(partition by item_name, year),2)=0 THEN 'avg'
     WHEN total_amount-round(AVG(total_amount) OVER(partition by item_name, year),2)>0 THEN 'good'
END AS sales_perf
FROM monthly_total_amount ;
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
SELECT item_name, 
	   transaction_amount, 
CASE WHEN month(date) = 2 THEN 'spring'
	 WHEN month(date) between 3 and 5 THEN 'summer'
	 WHEN month(date) between  6 and 9 THEN 'Monsoon'
	 WHEN month(date) between 10 and 11 THEN 'Pre-Winter'
ELSE 'Winter'
END AS season,
	YEAR (date) as year, 
	SUM(transaction_amount) OVER(partition by CASE WHEN month(date) = 2 THEN 'spring'
	 WHEN month(date) between 3 and 5 THEN 'summer'
	 WHEN month(date) between  6 and 9 THEN 'Monsoon'
	 WHEN month(date) between 10 and 11 THEN 'Pre-Winter'
ELSE 'Winter'
end, item_name) AS total_seasonal_sales
FROM food
ORDER BY item_name, total_seasonal_sales DESC ;
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

- 🇺🇸 USA contributed to **40%** of total sales revenue.  
- 📦 _Product A_ is the top-selling item with over 3,000 units sold.  
- 🕒 Sales spike consistently in Q4 due to holiday seasonality.  
- 👵 Customers aged 35–44 spend the most on average per order.

## ▶️ How to Use

1. Import `ecommerce_schema.sql` to create tables  
2. Run `insert_sample_data.sql` to populate the database  
3. Open `queries/` to view and execute analysis scripts  

## 📜 License

This project is licensed under the MIT License.
