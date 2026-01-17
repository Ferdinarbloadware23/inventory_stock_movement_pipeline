# inventory_stock_movement_pipeline

# Shopee Sales & Inventory Analytics Pipeline

This repository contains a SQL-based data pipeline built from Shopee transactional data.  
The project demonstrates how raw sales data and stock movements can be transformed into a structured, analysis-ready database for business reporting.

The pipeline covers two domains:

- Sales analytics using **1 week of real Shopee data**
- Inventory tracking using **controlled dummy stock-in data**

The goal is to showcase a realistic junior data analyst workflow using SQL only.

---

## Project Objectives

- Design a clean relational schema for sales and inventory
- Load raw data into staging tables
- Validate and clean incoming data
- Transform data into analytical tables
- Track product stock movements
- Generate business-ready analytical outputs

This project is intentionally built as a **manual SQL pipeline** to reflect how junior analysts often work in small teams or SMEs.

---

## Scope & Assumptions

- Sales data covers 1 week of completed orders
- Stock-in data is dummy but structured realistically
- Data is imported manually from CSV files
- Processing is executed using SQL only
- No scheduler or orchestration tools are used
- Focus is on clarity, correctness, and structure

---

## Database Overview

### Core Tables

- `sales_header`  
  Stores order-level transaction data (date, platform, gross/net amount, status)

- `sales_detail`  
  Stores item-level transaction data (product, quantity, pricing)

- `master_product`  
  Product reference and pricing information

- `stock_in`  
  Records inbound inventory from suppliers

- `stock_movement`  
  Unified log of all product movements (IN / OUT)

- `current_stock`  
  Snapshot of current stock level per product

- `stock_balance`  
  Aggregated stock metrics (total in, total out, current stock)

---

### Staging Tables

Used as temporary landing zones for raw CSV data:

- `staging_sales_header`
- `staging_sales_detail`
- `staging_stock_in`

---

## Data Pipeline Flow

1. Create schema and core tables  
2. Load raw CSV into staging tables  
3. Run validation queries:
   - Missing product mapping
   - Invalid quantity or pricing
   - Duplicate orders
4. Insert clean data into final tables  
5. Generate stock movements:
   - Stock-in → `IN`
   - Sales → `OUT`
6. Build stock balance & current stock tables  
7. Run analytical queries

---

## Key Analytical Outputs

### Daily Sales Performance

```sql
SELECT
    DATE(tanggal) AS sales_date,
    COUNT(DISTINCT order_id) AS total_orders,
    SUM(gross_amount) AS total_revenue
FROM sales_header
GROUP BY sales_date
ORDER BY sales_date;
```

### Top Product by Revenue
```sql
SELECT
    p.product_name,
    SUM(d.qty) AS total_qty_sold,
    SUM(d.price_total) AS total_revenue
FROM sales_detail d
JOIN master_product p
    ON d.id_product = p.id_product
GROUP BY p.product_name
ORDER BY total_revenue DESC
LIMIT 10;
```

### Average Order Value
```sql
SELECT
    ROUND(SUM(gross_amount) / COUNT(DISTINCT order_id), 2) AS avg_order_value
FROM sales_header;
