# 📦 Klyre – E-Commerce Performance Dashboard (2022–2025)

A multi-year data analysis project to uncover key insights and growth opportunities from 30,000 rows of anonymized e-commerce transactions.

---

## 📌 Project Objective

To extract actionable insights from Klyre’s 4-year e-commerce dataset and translate them into strategies for improving product, marketing, and operational decisions.

---

## 🧾 Dataset Overview

- **Source**: Google Public Dataset (`thelook_ecommerce`)
- **Time Period**: 2022–2025
- **Size**: 30,000 rows (7,500 per year)
- **Sampling Method**: Stratified random sampling to ensure balanced year-over-year distribution  
  → See SQL query in [Jump to SQL Sampling Query](#data-sampling-query)
- **Key Columns**:
  - Order dates & status
  - Delivery duration & return rates
  - Product brand, category, pricing, cost, profit
  - User demographics (age, gender, country)
  - Traffic source

---

## 🛠️ Tools & Methods

- **Google Sheets & Excel** for cleaning and pivot analysis
- **Exploratory Data Analysis (EDA)** with filters and calculated metrics
- **Interactive Dashboard** with slicers and key performance indicators (KPIs)
- **Segmentation** by:
  - Year
  - Customer age group
  - Product category
  - Region

---

## 🔍 What's Inside

This project includes:
- 📊 **Interactive Dashboard** summarizing KPIs, trends, and segments
- 📁 **Insight Reports** (available in `/reports/` folder), including:
  - Sales & Profit Trends
  - Product Performance
  - Customer Demographics
  - Marketing ROI
  - Delivery Efficiency
- 📄 **Executive Summary** for high-level decision-makers
- 📄 PDF Report → [Click here to view the full report](ecommerce_orders_report_2022_2025.pdf)

---

## 🚀 Key Highlights

- 👥 **Young Adults (25–34)** drove over 40% of total orders.
- 👙 **Swimwear** performed well in sales but had the highest return rate—possible sizing or quality issue.
- 🚚 **Delivery time improved** by 29% from 2022 to 2025, correlating with reduced return rates.
- 📈 **Facebook Ads** yielded high conversion value despite lower traffic share.

---

## 📁 Folder Structure
```
📦 Klyre-Ecommerce-Project
├── 📊 dashboard/
│   └── klyre_dashboard.jpg
├── 📄 reports/
│   ├── Klyre_E-Commerce_Report_2022-2025.pdf
│   ├── sales_profit_analysis.xlsx
│   └── customer_demographics.xlsx
├── 📂 data/
│   ├── ecommerce_orders_raw.csv
│   └── ecommerce_cleaned_2022_2025.xlsx
├── 🧾 sql/
│   └── data_sampling.sql
└── README.md
```
---
## Data Sampling Query

```
WITH base_data AS (
  SELECT 
      o.order_id,
      o.user_id,
      o.status AS order_status,
      o.created_at AS order_date,
      o.shipped_at AS shipped_date,
      o.delivered_at AS delivered_date,
      o.returned_at AS returned_date,

      oi.product_id,
      p.category AS product_category,
      p.brand AS product_brand,
      p.cost AS product_cost,
      p.retail_price AS product_retail_price,
      oi.sale_price,

      u.gender AS user_gender,
      u.age AS user_age,
      u.state AS user_state,
      u.country AS user_country,
      u.traffic_source,

      dc.name AS distribution_center,
      ii.created_at AS inventory_created_date,
      ii.sold_at AS inventory_sold_date,

      e.event_type,
      EXTRACT(YEAR FROM o.created_at) AS order_year,
      RAND() AS randomizer
  FROM 
      `bigquery-public-data.thelook_ecommerce.orders` o
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.order_items` oi ON o.order_id = oi.order_id
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.products` p ON oi.product_id = p.id
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.users` u ON o.user_id = u.id
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.inventory_items` ii ON oi.inventory_item_id = ii.id
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.distribution_centers` dc ON ii.id = dc.id
  LEFT JOIN 
      `bigquery-public-data.thelook_ecommerce.events` e ON o.user_id = e.user_id
  WHERE
      o.created_at BETWEEN '2022-01-01' AND '2025-12-31'
)

SELECT *
FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY order_year ORDER BY randomizer) AS row_num
  FROM base_data
)
WHERE row_num <= 7500
ORDER BY order_year, row_num;
