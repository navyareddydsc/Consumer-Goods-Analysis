# Create a README.md content as a markdown string based on the formatted structure above

readme_content = """
# Consumer Goods Analytics – SQL Project

##  Overview

This project simulates solving real business problems for a global hardware company, **Atliq Hardwares**, which produces and distributes computer hardware like laptops, peripherals, and accessories. The company's management lacked actionable insights to make data-driven decisions in a competitive market.

Using SQL, I addressed 10 ad-hoc business questions to support operational, marketing, and strategic planning decisions.

---

##  Problem Statement

**Atliq Hardwares** needed detailed insights to:
- Understand customer and product performance
- Optimize inventory and sales strategy
- Make faster, data-informed decisions across regions and product segments

---

##  Tools & Technologies

- **SQL (MySQL Workbench)**
- Dataset: Simulated database with tables like `dim_customer`, `dim_product`, `fact_sales_monthly`, etc.
- **Business context analysis** through SQL queries

---

##  Key Objectives

- Solve 10 real-world ad-hoc business requests
- Use SQL to extract, manipulate, and analyze data
- Derive meaningful **business insights** from the results

---

##  Dataset Description

Database: `gdb023`

| Table | Description |
|-------|-------------|
| `dim_customer` | Customer information |
| `dim_product` | Product catalog |
| `fact_sales_monthly` | Monthly sales data |
| `fact_gross_price` | Gross pricing information |
| `fact_manufacturing_cost` | Product manufacturing costs |
| `fact_pre_invoice_deduction` | Pre-invoice discounts and adjustments |

---

##  Ad-Hoc Business Queries Solved

# 📁 SQL Queries – Consumer Goods Analytics

This file contains the SQL queries used to solve 10 business ad-hoc requests for Atliq Hardwares. The queries are written in MySQL and were executed using MySQL Workbench.

---

## 🔍 Query 1: Markets where Atliq Exclusive operates in APAC
```sql
SELECT DISTINCT market 
FROM dim_customer 
WHERE customer = 'Atliq Exclusive' AND region = 'APAC';

## Query 2: Percentage of unique product increase in 2021 vs 2020
```sql

WITH unique_products_2020 AS (
    SELECT COUNT(DISTINCT product_code) AS product_2020
    FROM fact_sales_monthly
    WHERE fiscal_year = 2020
),
unique_products_2021 AS (
    SELECT COUNT(DISTINCT product_code) AS product_2021
    FROM fact_sales_monthly
    WHERE fiscal_year = 2021
)
SELECT product_2020, product_2021,
       ROUND(((product_2021 - product_2020) / product_2020) * 100, 2) AS percentage_increase
FROM unique_products_2020, unique_products_2021;

## Query 3: Unique product counts for each segment
```sql
SELECT segment, COUNT(DISTINCT product_code) AS unique_product_count
FROM dim_product
GROUP BY segment
ORDER BY unique_product_count DESC;

##  Query 4: Segment with most increase in unique products (2021 vs 2020)
```sql
WITH products_2020 AS (
    SELECT segment, COUNT(DISTINCT product_code) AS count_2020
    FROM dim_product dp
    JOIN fact_sales_monthly fs ON dp.product_code = fs.product_code
    WHERE fiscal_year = 2020
    GROUP BY segment
),
products_2021 AS (
    SELECT segment, COUNT(DISTINCT product_code) AS count_2021
    FROM dim_product dp
    JOIN fact_sales_monthly fs ON dp.product_code = fs.product_code
    WHERE fiscal_year = 2021
    GROUP BY segment
)
SELECT p2021.segment, p2021.count_2021 - p2020.count_2020 AS increase
FROM products_2020 p2020
JOIN products_2021 p2021 ON p2020.segment = p2021.segment
ORDER BY increase DESC;

## Query 5: Products with highest and lowest manufacturing costs
```sql
SELECT product_code, manufacturing_cost
FROM fact_manufacturing_cost
ORDER BY manufacturing_cost DESC
LIMIT 1;

SELECT product_code, manufacturing_cost
FROM fact_manufacturing_cost
ORDER BY manufacturing_cost ASC
LIMIT 1;

## Query 6: Top 5 customers with highest average pre-invoice discount in 2021 (India market)
```sql
SELECT customer_code, AVG(pre_invoice_discount_pct) AS avg_discount
FROM fact_pre_invoice_deduction fd
JOIN dim_customer dc ON fd.customer_code = dc.customer_code
WHERE fiscal_year = 2021 AND market = 'India'
GROUP BY customer_code
ORDER BY avg_discount DESC
LIMIT 5;

## Query 7: Monthly gross sales for Atliq-Exclusive
```sql
SELECT month(date) AS month, SUM(gross_price * sold_quantity) AS gross_sales
FROM fact_sales_monthly fs
JOIN dim_customer dc ON fs.customer_code = dc.customer_code
JOIN fact_gross_price gp ON fs.product_code = gp.product_code
WHERE dc.customer = 'Atliq Exclusive'
GROUP BY month
ORDER BY month;

## Query 8: Quarter with maximum total_sold_quantity in 2020
```sql
SELECT 
    CASE
        WHEN MONTH(date) IN (9,10,11) THEN 'Q1'
        WHEN MONTH(date) IN (12,1,2) THEN 'Q2'
        WHEN MONTH(date) IN (3,4,5) THEN 'Q3'
        ELSE 'Q4'
    END AS quarter,
    SUM(sold_quantity) AS total_sold
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY quarter
ORDER BY total_sold DESC;

##  Query 9: Channel with highest gross sales and percentage contribution (2021)
```sql
WITH total_sales AS (
    SELECT channel, SUM(gross_price * sold_quantity) AS total_gross_sales
    FROM fact_sales_monthly fs
    JOIN dim_customer dc ON fs.customer_code = dc.customer_code
    JOIN fact_gross_price gp ON fs.product_code = gp.product_code
    WHERE fiscal_year = 2021
    GROUP BY channel
),
grand_total AS (
    SELECT SUM(total_gross_sales) AS grand_total FROM total_sales
)
SELECT ts.channel, ts.total_gross_sales,
       ROUND((ts.total_gross_sales / gt.grand_total) * 100, 2) AS percentage_contribution
FROM total_sales ts, grand_total gt
ORDER BY total_gross_sales DESC;

##  Query 10: Top 3 products in each division with high total_sold_quantity (2021)
```sql
SELECT division, product_code, SUM(sold_quantity) AS total_sold
FROM fact_sales_monthly fs
JOIN dim_product dp ON fs.product_code = dp.product_code
WHERE fiscal_year = 2021
GROUP BY division, product_code
ORDER BY division, total_sold DESC;


## 🧠 Recommendations

Based on the insights derived from the SQL analysis, the following recommendations can be made to Atliq Hardwares:

1. **Focus Marketing on High-Performing Channels**  
   The *Retailer* channel accounts for 75% of gross sales in FY21. Strategic marketing and promotional efforts should continue to focus here, while exploring potential growth opportunities in the *Direct* and *Distributor* channels.

2. **Optimize Inventory Around Peak Quarters**  
   Sales peaked in **Q1 of FY20** and **November 2021**, indicating seasonality. Inventory planning and promotional campaigns should be aligned with these peak periods to avoid stockouts and maximize revenue.

3. **Expand in Top-Performing Segments**  
   The **Accessories** and **Notebook** segments showed the highest growth in unique product sales. Consider expanding SKUs or bundling products in these segments to further boost revenue.

4. **Reevaluate Manufacturing Costs**  
   Products with significantly higher manufacturing costs should be reviewed for profitability. Consider supply chain optimization or alternative vendors for cost-intensive products.

5. **Refine Discount Strategy by Customer**  
   Customers like *Flipkart* received higher average pre-invoice discounts. Assess whether the discount levels are justified by the sales volume and margins to prevent revenue leakage.

6. **Leverage High-Performing Products in Each Division**  
   Products like **AQ Pen Drive DRC** and **AQ Maxima MS** performed exceptionally in their divisions. Use these insights to guide promotional focus, bundling, and cross-selling opportunities.

7. **Increase Product Variety in Underperforming Segments**  
   Segments like *Networking* and *Storage* had the lowest unique product counts. Evaluate customer demand and consider product diversification to increase competitiveness.

---

These recommendations support smarter, data-driven decisions that align with Atliq’s strategic goals of improving operational efficiency, customer targeting, and revenue growth.





