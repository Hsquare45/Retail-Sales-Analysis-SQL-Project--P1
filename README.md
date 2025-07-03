# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `p1_retail_db`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project is ideal for those who are starting their journey in data analysis and want to build a solid foundation in SQL.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values. Also, renamed an incorrect column.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE sql_project_p1_D1;

CREATE TABLE retail_sale
	(
transactions_id INT,
sale_date DATE,
sale_time TIME,
customer_id	 INT,
gender	VARCHAR(10),
age	INT,
category VARCHAR(15),
quantiy INT,
price_per_unit FLOAT,
cogs FLOAT,
total_sale FLOAT
	);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.
- **Rename an incorrect column (Quantiy)

```sql
SELECT COUNT(*) FROM retail_sale;
SELECT DISTINCT category FROM retail_sale;

SELECT *
FROM retail_sale
WHERE 
	transactions_id IS NULL
	OR sale_date IS NULL
 	OR sale_time IS NULL
	OR customer_id IS NULL
	OR gender IS NULL
	OR category IS NULL
	OR quantiy IS NULL
	OR price_per_unit IS NULL
	OR cogs IS NULL
	OR total_sale IS NULL;

DELETE
FROM retail_sale
WHERE 
	transactions_id IS NULL
	OR sale_date IS NULL
 	OR sale_time IS NULL
	OR customer_id IS NULL
	OR gender IS NULL
	OR category IS NULL
	OR quantiy IS NULL
	OR price_per_unit IS NULL
	OR cogs IS NULL
	OR total_sale IS NULL;

ALTER TABLE retail_sale
RENAME column quantiy to quantity
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
SELECT *
FROM retail_sale
WHERE sale_date = '2022-11-05';
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 2 in the month of Nov-2022**:
```sql
SELECT *
FROM retail_sale
WHERE 
	category = 'Clothing' 
	AND 
	quantity > 2 
	AND 
	TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sale
GROUP BY 1
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT ROUND(AVG(age),3) avg_age, category
FROM retail_sale
WHERE category = 'Beauty'
GROUP BY 2
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT *
FROM retail_sale
WHERE 
	total_sale > 1000
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT COUNT(*), gender, category
FROM retail_sale
GROUP BY 2,3
ORDER BY 1 DESC
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**: (in Subquery form)
```sql
SELECT year, month, avg_sale
FROM
(
	SELECT 
		EXTRACT(YEAR FROM sale_date) as year,
		EXTRACT(MONTH FROM sale_date) as month,
		ROUND(AVG(total_sale)::numeric,2) avg_sale,
		RANK() OVER
		(
		PARTITION BY EXTRACT(YEAR FROM sale_date) 
		ORDER BY ROUND(AVG(total_sale)::numeric,2) desc
		)
		AS rank	
			
	FROM retail_sale 
	GROUP BY 1, 2
)
WHERE rank =1
```
** (In CTE form)
```sql
WITH avg_sales_ranked AS(
	SELECT 
		EXTRACT(YEAR FROM sale_date) as year,
		EXTRACT(MONTH FROM sale_date) as month,
		ROUND(AVG(total_sale)::numeric,2) as avg_sale,
		RANK() OVER(
		PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY ROUND(AVG(total_sale)::numeric,2) desc
		) AS rank
	FROM retail_sale
	GROUP BY 1,2	
)
SELECT year, month, avg_sale
FROM avg_sales_ranked
WHERE rank = 1
```

8. **Which month has the most avg_sales in the 2 years combined?
```sql
SELECT 
	TO_CHAR(sale_date, 'month') as month_name,
	EXTRACT(MONTH FROM sale_date) as month,
	ROUND(AVG(total_sale)::numeric,2) as avg_sale
FROM retail_sale
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5
```
9. **Calculate the monthly sales for each year. Find the month with the most sales
```sql
WITH sales_ranked AS(
	SELECT 
		SUM(total_sale) as Tot_sale,
		EXTRACT(MONTH FROM sale_date) as month,
		EXTRACT(YEAR FROM sale_date) as year,
		TO_CHAR(sale_date, 'month') as Month_name,
		RANK() OVER
		(
		PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY SUM(total_sale) DESC
		) AS rank
	FROM retail_sale
	GROUP BY 2,3,4
	ORDER BY 1 DESC)

SELECT month_name, year, tot_sale
FROM sales_ranked
ORDER BY 3 desc
LIMIT 6
```
10. **Calculate the month that has had the most sales overall
```sql
SELECT TO_CHAR(sale_date, 'month') month_name,
		SUM(total_sale) Tot_sale
FROM retail_sale
GROUP BY 1
ORDER BY 2 DESC
```
11. **Find the number of unique customers who purchased items from each category
```sql
SELECT COUNT(DISTINCT customer_id), category
FROM retail_sale
GROUP BY 2
ORDER BY 1 DESC
```
12. **Create each shift and number of orders example (morning < 12, afternoon >12 and <17, evening)
```sql
WITH hourly_orders AS
(	SELECT *,
		CASE 
		WHEN EXTRACT(HOUR FROM sale_time) <= 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon' 
		ELSE 'Evening' 
		END AS periods
		
	FROM retail_sale
)
SELECT COUNT(*), periods
FROM hourly_orders
GROUP BY 2
ORDER BY 1 DESC
```

13. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sale
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.


