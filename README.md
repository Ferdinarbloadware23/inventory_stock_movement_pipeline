# Inventory & Stock Movement Pipeline

This repository extends the sales analytics project by introducing a structured inventory and stock movement pipeline.

The project demonstrates how inbound stock, product movements, and sales transactions can be unified into a single, analysis-ready database using SQL only.

It simulates a realistic workflow used by small businesses to track inventory without complex ERP systems.

---

## Project Objective

This project is designed to:

- Build an inventory-aware data model
- Track stock movements (IN / OUT) per product
- Integrate sales data as stock-out events
- Maintain current stock per SKU
- Demonstrate end-to-end operational analytics using SQL

The pipeline is intentionally manual to reflect how junior analysts often work in small teams or SMEs.

---

## Scope & Assumptions

- Sales data covers **1 week of real Shopee transactions**
- Stock-in data uses **controlled dummy data**
- Data is imported manually via CSV
- Processing is done using SQL only
- No automation or orchestration tools are used
- Focus is on logic clarity and data modeling

---

## Core Tables

- `sales_header` – order-level sales data  
- `sales_detail` – item-level sales data  
- `stock_in` – inbound inventory records  
- `stock_movement` – unified stock ledger (IN / OUT)  
- `current_stock` – latest stock position per SKU  
- `master_product` – product reference  
- `supplier` – supplier master data  

Staging layer:

- `staging_sales_header`
- `staging_sales_detail`
- `staging_stock_in`

---

## Data Pipeline Flow

1. Create schema and core tables  
2. Load raw CSV into staging tables  
3. Run validation checks  
4. Insert clean data into final tables  
5. Generate stock movements:
   - Stock-in → `IN`
   - Sales → `OUT`
6. Build stock balance & current stock tables  
7. Run analytical queries  

---

## Key Analytical Outputs

This project focuses on operational analytics such as:

- Current stock per product  
- Stock movement history (IN vs OUT)  
- Net stock change per day  
- Low-stock detection  
- Sales-driven stock depletion  

### current stock per SKU

```sql
SELECT
    id_product,
    SUM(CASE WHEN movement_type = 'IN' THEN qty ELSE 0 END)
    - SUM(CASE WHEN movement_type = 'OUT' THEN qty ELSE 0 END) AS current_stock
FROM stock_movement
GROUP BY id_product;
