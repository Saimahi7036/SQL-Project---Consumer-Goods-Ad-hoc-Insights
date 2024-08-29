# Consumer Goods Ad-hoc Insights
## SQL Project - Codebasics SQL Project Challenge

This project involves solving various ad-hoc business queries related to the consumer goods industry using SQL. Below are the tasks and corresponding SQL queries that were executed as part of this challenge.

---

### Task 1: Retrieve Distinct Markets in the APAC Region
#### Query:
```sql
SELECT DISTINCT market 
FROM dim_customer
WHERE region = 'APAC' AND customer = 'Atliq Exclusive';

Description: This query retrieves the distinct markets in the APAC region where the customer is 'Atliq Exclusive'.

Task 2: Calculate the Percentage Change in Unique Products
Query:
sql
Copy code
WITH unique_products AS (
    SELECT fiscal_year, COUNT(DISTINCT product_code) AS unique_products 
    FROM fact_gross_price 
    GROUP BY fiscal_year
)
SELECT 
    up_2020.unique_products AS unique_products_2020,
    up_2021.unique_products AS unique_products_2021,
    ROUND((up_2021.unique_products - up_2020.unique_products) / up_2020.unique_products * 100, 2) AS percentage_change
FROM unique_products up_2020
JOIN unique_products up_2021
ON up_2020.fiscal_year = 2020 AND up_2021.fiscal_year = 2021;
Description: This query calculates the percentage change in unique products between the fiscal years 2020 and 2021.

Task 3: Count of Products by Segment
Query:
sql
Copy code
SELECT segment, COUNT(DISTINCT product_code) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
Description: This query returns the count of products for each segment, ordered by the segment with the highest number of products.

Task 4: Product Count Difference by Segment Between 2020 and 2021
Query:
sql
Copy code
WITH temp_table AS (
    SELECT p.segment, s.fiscal_year, COUNT(DISTINCT s.product_code) AS product_count
    FROM fact_sales_monthly s
    JOIN dim_product p ON s.product_code = p.product_code
    GROUP BY p.segment, s.fiscal_year
)
SELECT 
    t1.segment,
    t1.product_count AS product_count_2020,
    t2.product_count AS product_count_2021,
    t2.product_count - t1.product_count AS difference
FROM temp_table t1
JOIN temp_table t2
ON t1.segment = t2.segment AND t1.fiscal_year = 2020 AND t2.fiscal_year = 2021
ORDER BY difference DESC;
Description: This query identifies the difference in the product count by segment between the years 2020 and 2021.

Task 5: Retrieve Products with Minimum and Maximum Manufacturing Costs
Query:
sql
Copy code
SELECT m.product_code, CONCAT(p.product, ' (', p.variant, ')') AS product, cost_year, manufacturing_cost
FROM fact_manufacturing_cost m
JOIN dim_product p ON m.product_code = p.product_code
WHERE manufacturing_cost = 
      (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
UNION
SELECT m.product_code, CONCAT(p.product, ' (', p.variant, ')') AS product, cost_year, manufacturing_cost
FROM fact_manufacturing_cost m
JOIN dim_product p ON m.product_code = p.product_code
WHERE manufacturing_cost = 
      (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost)
ORDER BY manufacturing_cost DESC;
Description: This query retrieves the products with the minimum and maximum manufacturing costs, along with their corresponding product details.

Task 6: Top 5 Customers with the Highest Average Pre-Invoice Discount Percentage in 2021 (India Market)
Query:
sql
Copy code
SELECT c.customer_code, c.customer, ROUND(AVG(pre_invoice_discount_pct), 4) AS average_discount_percentage
FROM fact_pre_invoice_deductions d
JOIN dim_customer c ON d.customer_code = c.customer_code
WHERE c.market = 'India' AND fiscal_year = 2021
GROUP BY customer_code
ORDER BY average_discount_percentage DESC
LIMIT 5;
Description: This query retrieves the top 5 customers in the India market with the highest average pre-invoice discount percentage in 2021.

Task 7: Gross Sales by Month for 'Atliq Exclusive'
Query:
sql
Copy code
WITH temp_table AS (
    SELECT c.customer, MONTHNAME(date) AS months, MONTH(date) AS month_number, YEAR(date) AS year,
           (sold_quantity * gross_price) AS gross_sales
    FROM fact_sales_monthly s
    JOIN fact_gross_price g ON s.product_code = g.product_code
    JOIN dim_customer c ON s.customer_code = c.customer_code
    WHERE customer = 'Atliq Exclusive'
)
SELECT months, year, CONCAT(ROUND(SUM(gross_sales) / 1000000, 2), 'M') AS gross_sales 
FROM temp_table
GROUP BY year, months
ORDER BY year, month_number;
Description: This query calculates the gross sales by month for the customer 'Atliq Exclusive', displaying the sales in millions.

Task 8: Total Sold Quantity in Millions by Quarter for 2020
Query:
sql
Copy code
WITH temp_table AS (
    SELECT date, MONTH(DATE_ADD(date, INTERVAL 4 MONTH)) AS period, fiscal_year, sold_quantity 
    FROM fact_sales_monthly
)
SELECT 
    CASE 
        WHEN period <= 3 THEN 'Q1'
        WHEN period <= 6 THEN 'Q2'
        WHEN period <= 9 THEN 'Q3'
        ELSE 'Q4' 
    END AS quarter,
    ROUND(SUM(sold_quantity) / 1000000, 2) AS total_sold_quantity_in_millions 
FROM temp_table
WHERE fiscal_year = 2020
GROUP BY quarter
ORDER BY total_sold_quantity_in_millions DESC;
Description: This query calculates the total sold quantity in millions by quarter for the year 2020.

Task 9: Gross Sales by Sales Channel in 2021
Query:
sql
Copy code
WITH temp_table AS (
    SELECT c.channel, SUM(s.sold_quantity * g.gross_price) AS total_sales
    FROM fact_sales_monthly s 
    JOIN fact_gross_price g ON s.product_code = g.product_code
    JOIN dim_customer c ON s.customer_code = c.customer_code
    WHERE s.fiscal_year = 2021
    GROUP BY c.channel
    ORDER BY total_sales DESC
)
SELECT 
    channel,
    ROUND(total_sales / 1000000, 2) AS gross_sales_in_millions,
    ROUND(total_sales / SUM(total_sales) OVER() * 100, 2) AS percentage 
FROM temp_table;
Description: This query retrieves the gross sales by sales channel for the year 2021, displaying the sales in millions and their corresponding percentage contribution.

Task 10: Top 3 Products by Division in Terms of Sold Quantity in 2021
Query:
sql
Copy code
WITH temp_table AS (
    SELECT division, s.product_code, CONCAT(p.product, '(', p.variant, ')') AS product, 
           SUM(sold_quantity) AS total_sold_quantity,
           RANK() OVER (PARTITION BY division ORDER BY SUM(sold_quantity) DESC) AS rank_order
    FROM fact_sales_monthly s
    JOIN dim_product p ON s.product_code = p.product_code
    WHERE fiscal_year = 2021
    GROUP BY division, s.product_code, p.product, p.variant
)
SELECT * FROM temp_table
WHERE rank_order IN (1, 2, 3);
Description: This query retrieves the top 3 products by division in terms of sold quantity for the year 2021.



