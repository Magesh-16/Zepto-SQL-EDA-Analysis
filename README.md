# 🛒 Zepto Inventory & Sales Analysis (SQL/MySQL)

End-to-end SQL project on a real-world e-commerce inventory dataset from **Zepto** — covering database setup, exploratory data analysis (EDA), data cleaning, and business-focused analysis to uncover pricing, revenue, and inventory insights.

---

## 📌 Project Overview

This project simulates how a Data Analyst would approach a raw e-commerce dataset:
1. Set up the database and explore its structure
2. Identify data quality issues (nulls, duplicates, invalid values, wrong units)
3. Clean the data
4. Answer real business questions using SQL

The dataset contains product-level details such as category, MRP, discount percentage, selling price, weight, available quantity, and stock status.

---

## 🛠️ Tech Stack

- **MySQL** (MySQL Workbench)
- SQL concepts used: `JOIN`-free aggregation, `GROUP BY`, `HAVING`, `CASE WHEN`, window-free ranking via `ORDER BY` + `LIMIT`, `DISTINCT`, subqueries-free filtering, data type conversion via arithmetic updates

---

## 🗂️ Dataset

| Column | Description |
|---|---|
| `name` | Product name |
| `category` | Product category |
| `mrp` | Maximum Retail Price (originally in paise) |
| `discountPercent` | Discount percentage offered |
| `discountedSellingPrice` | Final selling price after discount |
| `weightInGms` | Product weight in grams |
| `availableQuantity` | Units available in inventory |
| `outOfStock` | Stock availability flag |
| `quantity` | Order quantity unit |

---

## 🔍 Step 1: Database & Table Setup

```sql
USE zepto_project_main;

CREATE TABLE zepto LIKE zepto_v2;

INSERT INTO zepto
SELECT * FROM zepto_v2;

ALTER TABLE zepto
ADD COLUMN p_id INT PRIMARY KEY AUTO_INCREMENT;
```
A working copy of the raw table was created and a primary key (`p_id`) was added to support reliable row-level operations (deletion, deduplication checks) throughout the project.

---

## 🔎 Step 2: Data Exploration (EDA)

**Row count and sample check**
```sql
SELECT COUNT(*) FROM zepto;
SELECT * FROM zepto LIMIT 10;
```

**Null value check across all key columns**
```sql
SELECT * FROM zepto
WHERE name IS NULL OR category IS NULL OR mrp IS NULL
   OR discountPercent IS NULL OR discountedSellingPrice IS NULL
   OR weightInGms IS NULL OR availableQuantity IS NULL
   OR outOfStock IS NULL OR quantity IS NULL;
```

**Category exploration**
```sql
SELECT DISTINCT category FROM zepto ORDER BY category;
```

**Stock status distribution**
```sql
SELECT outOfStock, COUNT(p_id) FROM zepto GROUP BY outOfStock;
```

**Duplicate product check**
```sql
SELECT name, COUNT(p_id) AS "Number of id"
FROM zepto
GROUP BY name
HAVING COUNT(p_id) > 1
ORDER BY COUNT(p_id) DESC;
```

**Key findings from exploration:**
- No critical nulls found in core pricing/category columns
- Several products appeared multiple times under the same name (different package sizes/listings)
- A handful of products had `mrp` or `discountedSellingPrice` = 0 — invalid entries needing removal
- Prices were stored in **paise**, not rupees (e.g., ₹199 stored as 19900)

---

## 🧹 Step 3: Data Cleaning

**Remove invalid zero-price rows**
```sql
SET SQL_SAFE_UPDATES = 0;

DELETE FROM zepto
WHERE mrp = 0;
```

**Convert currency from paise to rupees**
```sql
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;

UPDATE zepto
SET mrp = ROUND(mrp),
    discountedSellingPrice = ROUND(discountedSellingPrice);
```

This step was essential — without it, every downstream revenue and pricing calculation would have been inflated by 100x.

---

## 📊 Step 4: Business Analysis

### Q1. Top 10 best-value products by discount percentage
```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```

### Q2. Estimated revenue per category
```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue;
```

### Q3. Premium products with low discounts (MRP > ₹500, discount < 10%)
```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;
```

### Q4. Top 5 categories with the highest average discount
```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

### Q5. Best value-for-money products by price per gram
```sql
SELECT DISTINCT name, weightInGms, discountedSellingPrice,
       ROUND(discountedSellingPrice / weightInGms, 2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram;
```

### Q6. Segmenting products by weight (Low / Medium / Bulk)
```sql
SELECT DISTINCT name, weightInGms,
       CASE
           WHEN weightInGms < 1000 THEN 'Low'
           WHEN weightInGms < 5000 THEN 'Medium'
           ELSE 'Bulk'
       END AS weight_category
FROM zepto;
```

### Q7. Total inventory weight per category
```sql
SELECT category,
       SUM(weightInGms * availableQuantity) AS total_weight
FROM zepto
GROUP BY category
ORDER BY total_weight;
```

---

## 💡 Key Insights

- Identified the highest-discount products, useful for flash-sale/promotional targeting
- Estimated category-wise revenue contribution to spot top-performing categories
- Flagged premium, low-discount products as potential candidates for pricing strategy review
- Calculated price-per-gram to surface genuinely best-value products beyond just discount %
- Segmented inventory by weight class to support warehousing and logistics planning
- Quantified total inventory weight by category, relevant for storage and shipping cost estimation

---

## 📁 Repository Structure

```
├── zepto_eda_analysis.sql     # Full SQL script (setup, EDA, cleaning, analysis)
└── README.md                  # Project documentation
```

---

## 🚀 Key Takeaway

This project reflects a realistic analyst workflow — starting from a raw, messy dataset and working through structured exploration and cleaning before drawing business conclusions, rather than jumping straight to analysis on unverified data.

---

**Author:** Magesh
**Tools:** MySQL

