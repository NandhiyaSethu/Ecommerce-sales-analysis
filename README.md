# E-Commerce Sales Analysis
### Identifying Profit Drivers and Regional Performance Using BigQuery SQL and Google Sheets

---

## Project Overview

This project analyses **4 years of retail sales data (2014–2017)** from the Superstore dataset to identify key business insights around profitability, regional performance, and seasonal trends. The goal was to answer a core business question:

> *"Where is the company making money, where is it losing money, and what can be done about it?"*

---

## Tools and Technologies

| Tool | Purpose |
|---|---|
| **BigQuery (SQL)** | Data storage, cleaning, and analysis |
| **Google Sheets** | Data validation, pivot tables, and charts |
| **GitHub** | Version control and portfolio hosting |

---

## Dataset

| Detail | Info |
|---|---|
| Source | Kaggle — Sample Superstore Dataset |
| Rows | 9,994 orders |
| Columns | 21 (Category, Sales, Profit, Discount, Region, Order Date etc.) |
| Period | January 2014 — December 2017 |

---

## Problem Statement

The Superstore company generates $2.29M in total revenue but only retains $286K as profit — an overall margin of 12.5%. This analysis was conducted to find which categories, regions, and time periods are responsible for profit leakage and to provide actionable recommendations.

---

## Key Findings

### 1. Furniture Margin Problem
- Furniture generates **$742K in sales** but only **$18K in profit** — a margin of just **2.5%**
- Technology achieves **17.4% margin** on $836K in sales
- The **Tables sub-category loses $17K** despite $206K in sales — selling more makes losses worse
- **Recommendation:** Review supplier pricing for furniture. Consider discontinuing loss-making sub-categories like Tables and Bookcases.

### 2. Central Region Over-Discounting
- Central region applies an average discount of **24%** — nearly double other regions (10–15%)
- This results in the **lowest profit margin of 7.9%** across all regions
- West region gives only **10.9% discount** and achieves **14.9% margin** — nearly double Central
- **Recommendation:** Cap discounts at 15% for Central region. This policy change alone could recover approximately **$30K in annual profit**

### 3. Seasonal Sales Pattern
- Sales peak **every Q4 (Oct–Dec)** and dip every **Jan–Feb** — consistently across all 4 years
- November 2017 reached **$116K** — the highest single month in the dataset
- January sales regularly drop to **$14–18K** — a 6x difference from peak months
- **Recommendation:** Stock inventory in September, run marketing campaigns in August, and hire temporary support staff in October to capture peak season demand

---

## SQL Queries

### Query 1 — Sales and Profit by Category
```sql
SELECT
  Category,
  COUNT(*) AS total_orders,
  ROUND(SUM(Sales), 2) AS total_sales,
  ROUND(SUM(Profit), 2) AS total_profit,
  ROUND(SUM(Profit) * 100.0 / SUM(Sales), 1)
    AS profit_margin_pct
FROM `e-commerce-analysis-495305.superstore_data.superstore1`
GROUP BY Category
ORDER BY total_sales DESC;
```

### Query 2 — Top and Bottom Sub-Categories by Profit (CTE + RANK)
```sql
WITH SubCategoryProfit AS (
  SELECT
    `Sub-Category`,
    ROUND(SUM(Sales), 2) AS total_sales,
    ROUND(SUM(Profit), 2) AS total_profit,
    RANK() OVER (ORDER BY SUM(Profit) DESC)
      AS profit_rank
  FROM `e-commerce-analysis-495305.superstore_data.superstore1`
  GROUP BY `Sub-Category`
),
total_count AS (
  SELECT COUNT(*) AS cnt FROM SubCategoryProfit
)
SELECT s.*
FROM SubCategoryProfit s, total_count t
WHERE s.profit_rank <= 5
   OR s.profit_rank > t.cnt - 4
ORDER BY s.profit_rank;
```

### Query 3 — Monthly Sales Trend with Running Total
```sql
SELECT
  order_month,
  monthly_sales,
  monthly_profit,
  ROUND(SUM(monthly_sales) OVER (
    ORDER BY order_month
  ), 2) AS running_total_sales
FROM (
  SELECT
    FORMAT_DATE('%Y-%m', `Order Date`) AS order_month,
    ROUND(SUM(Sales), 2) AS monthly_sales,
    ROUND(SUM(Profit), 2) AS monthly_profit
  FROM `e-commerce-analysis-495305.superstore_data.superstore1` 
  GROUP BY FORMAT_DATE('%Y-%m', `Order Date`)
)
ORDER BY order_month;
```

### Query 4 — Region Performance vs Discount Impact
```sql
SELECT 
  Region,
  COUNT(*) AS total_orders,
  ROUND(SUM(Sales), 2) AS total_sales,
  ROUND(SUM(Profit), 2) AS total_profit,
  ROUND(AVG(Discount) * 100, 1) AS avg_discount_pct,
  ROUND(SUM(Profit) * 100.0 / SUM(Sales), 1) 
    AS profit_margin_pct
FROM `e-commerce-analysis-495305.superstore_data.superstore1` 
GROUP BY Region
ORDER BY total_profit DESC;
```

### Query 5 — Top Customers per Segment (Double CTE + PARTITION BY)
```sql
WITH CustomerSales AS (
  SELECT
    Customer_Name,
    Segment,
    ROUND(SUM(Sales), 2) AS total_spent,
    COUNT(DISTINCT Order_ID) AS total_orders,
    ROUND(SUM(Profit), 2) AS total_profit
  FROM `ecommerce-analysis.superstore_data.superstore`
  GROUP BY Customer_Name, Segment
),
RankedCustomers AS (
  SELECT *,
    RANK() OVER (
      PARTITION BY Segment
      ORDER BY total_spent DESC
    ) AS rank_in_segment
  FROM CustomerSales
)
SELECT *
FROM RankedCustomers
WHERE rank_in_segment <= 3
ORDER BY Segment, rank_in_segment;
```

---

## SQL Concepts Demonstrated

| Concept | Used In |
|---|---|
| GROUP BY + Aggregations | Query 1, 4 |
| Common Table Expressions (CTEs) | Query 2, 5 |
| RANK() Window Function | Query 2, 5 |
| PARTITION BY | Query 5 |
| Running Total (SUM OVER) | Query 3 |
| Subquery | Query 3 |
| FORMAT_DATE | Query 3 |
| COUNTIF | Data validation |

---

## Charts Built in Google Sheets

| Chart | Type | Key Insight |
|---|---|---|
| Sales vs Profit by Category | Grouped Bar | Furniture 2.5% vs Technology 17.4% margin |
| Monthly Sales Trend 2014–2017 | Line Chart | Q4 peaks every year — 6x Jan-Feb levels |
| Region: Profit Margin vs Avg Discount | Grouped Bar | Central 24% discount = 7.9% margin |

---

## Results Summary

| Metric | Value |
|---|---|
| Total Revenue | $2,297,200 |
| Total Profit | $286,397 |
| Overall Margin | 12.5% |
| Best Category (Margin) | Technology — 17.4% |
| Worst Category (Margin) | Furniture — 2.5% |
| Best Region (Margin) | West — 14.9% |
| Worst Region (Margin) | Central — 7.9% |
| Peak Sales Month | November 2017 — $116K |
| Estimated Profit Recovery | $50–60K with recommended changes |

---

## Business Recommendations

1. **Discontinue Tables sub-category** — losing $17K annually despite high sales volume
2. **Cap Central region discounts at 15%** — could recover ~$30K in annual profit
3. **Invest in Technology marketing** — highest margin category at 17.4% but only 18.5% of orders
4. **Plan Q4 inventory by September** — seasonal peak is predictable and consistent every year

---

## Repository Structure

```
ecommerce-sales-analysis/
├── README.md
├── queries/
│   ├── 01_category_analysis.sql
│   ├── 02_subcategory_profit.sql
│   ├── 03_monthly_trend.sql
│   ├── 04_region_performance.sql
│   └── 05_customer_segmentation.sql
└── insights/
    └── key_findings.md
```

---

## About Me

This project demonstrates end-to-end data analysis skills — from writing advanced SQL queries in BigQuery to building business insights and visualisations in Google Sheets.

- LinkedIn: [your LinkedIn URL]
- Tableau Public:  (https://public.tableau.com/app/profile/nandhiya.s/viz/HRAttrition_17774538480190/Dashboard2)
- Email: nandhiyadevi2000@gmail.com

---

