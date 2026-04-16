
# 🛍️ Retail Sales Analysis – End-to-End Project

---

## 📌 Project Overview

This project demonstrates a complete data analytics workflow/[Retail Sales Performance & Customer Insights Systems.xlsx]
Full data analytics pipeline: synthetic data creation in Excel, transformation in SQL Server, and interactive dashboard development in Power BI using a star schema and DAX delivering insights on revenue, profitability, customer behavior, and branch performance.


```text
Excel → SQL Server → Power BI
```

**Objective:** Build a realistic retail dataset from scratch, enrich it, perform SQL-based analysis, and develop a Power BI dashboard for actionable business insights.

---

## 🛠️ Tools Used

![Excel](https://img.shields.io/badge/Excel-Data%20Generation-green)
![SQL](https://img.shields.io/badge/SQL-Data%20Transformation-blue)
![PowerBI](https://img.shields.io/badge/PowerBI-Visualization-yellow)

---

## 🧮 1. Data Creation & Enrichment (Excel)

### 📊 Tables Created (>800 rows)

* **Sales**: `sale_id`, `date`, `branch_id`, `customer_id`, `product_id`, `quantity`, `price`, `discount`, `total_amount`
* **Customers**: `customer_id`, `name`, `gender`, `region`, `signup_date`, `loyalty_status`, `store_branch`
* **Products**: `product_id`, `category`, `product_name`, `cost_price`, `selling_price`
(https://github.com/user-attachments/files/26796398/Retail.Sales.Performance.Customer.Insights.Systems.xlsx)
---

### 🔧 Key Excel Techniques

* Random data generation:

```excel
=RANDBETWEEN(1,10)
```

```excel
=DATE(2024, RANDBETWEEN(1,12), RANDBETWEEN(1,28))
```

* Discount logic:

```excel
=IF(price<100,0,
 IF(price>=200,0.15,
 IF(price>=150,0.10,
 IF(price>=100,0.05,0))))
```

* Total amount calculation:

```excel
=price * quantity * (1 - discount)
```

---

### 🔗 Data Enrichment

* **VLOOKUP** used to populate:

  * `gender`
  * `loyalty_status`

* Additional techniques:

  * `IFS` for categorization
  * `SUMIFS / COUNTIFS` for aggregations
  * Margin % calculation

* **Power Query**:

  * Table merging
  * Date transformation

---

## 🗄️ 2. SQL Server Operations (SSMS)

### 📦 Tables

* `Sales`
* `Product_Table`
* `Customer_Table`

---

### 🔄 Key SQL Operations

#### 🔹 Populate Product Table (Sample Insert)

```sql
INSERT INTO Product_Table (Store_branch, Store_ID)
VALUES
(100, 1),
(100, 2),
(100, 3),
-- continues pattern up to branch 10
(100, 10);
```

---

#### 🔹 Generate Branch Labels

```sql
UPDATE Product_Table
SET Store_branch = 'Branch ' + CAST((Product_ID % 10) + 1 AS VARCHAR);
```

---

#### 🔹 Apply Discount Rules

```sql
UPDATE Sales
SET discount_rate = 
    CASE 
        WHEN price < 100 THEN 0
        WHEN price >= 200 THEN 0.15
        WHEN price >= 150 THEN 0.10
        WHEN price >= 100 THEN 0.05
    END;
```

---

#### 🔹 Recalculate Total Amount

```sql
UPDATE Sales
SET total_amount = price * quantity * (1 - discount_rate);
```

---

#### 🔹 Remove Duplicate Records

```sql
DELETE FROM Product_Table
WHERE Product_ID IN (
    SELECT Product_ID
    FROM (
        SELECT Product_ID,
               ROW_NUMBER() OVER (
                   PARTITION BY Product_ID, Product_name 
                   ORDER BY Product_ID
               ) AS rn
        FROM Product_Table
    ) d
    WHERE rn > 1
);
```

---

#### 🔹 Assign Most Frequent Branch per Customer

```sql
UPDATE c
SET c.branch_id = s.branch_id
FROM Customer_Table c
JOIN (
    SELECT customer_id, branch_id
    FROM (
        SELECT customer_id, branch_id,
               ROW_NUMBER() OVER (
                   PARTITION BY customer_id 
                   ORDER BY COUNT(*) DESC
               ) AS rn
        FROM Sales
        GROUP BY customer_id, branch_id
    ) x
    WHERE rn = 1
) s ON c.customer_id = s.customer_id;
```

> Assigning the most frequently visited branch ensures consistent customer segmentation and more accurate branch-level analysis.

---

#### 🔹 Revenue by Category

```sql
SELECT 
    p.category,
    SUM(s.selling_price * s.quantity) AS total_revenue
FROM Sales s
JOIN Product_Table p 
    ON s.product_id = p.product_id
GROUP BY p.category;
```

---

#### 🔹 Top 5 Customers

```sql
SELECT TOP 5
    customer_id,
    SUM(total_amount) AS total_revenue
FROM Sales
GROUP BY customer_id
ORDER BY total_revenue DESC;
```

---

#### 🔹 Profit Analysis

```sql
SELECT 
    s.store_branch,
    SUM(s.total_amount) AS total_revenue,
    SUM(p.cost_price * s.quantity) AS total_cost,
    SUM(s.discount) AS total_discount,
    SUM(s.total_amount - (p.cost_price * s.quantity) - s.discount) AS total_profit
FROM Sales s
JOIN Product_Table p 
    ON s.product_id = p.product_id
GROUP BY s.store_branch
ORDER BY total_profit DESC;
```

---

## 📊 3. Power BI Dashboard

### 📸 Dashboard Preview

<img width="1441" height="805" alt="Sales analysis dashboard project" src="https://github.com/user-attachments/assets/c1dad4b7-0a2c-4ab2-9aea-ec6a264ca8ab" />


---

### 🎯 Key Metrics

* Total Revenue: **$137,645.91**
* Net Profit: **$27,186**
* Profit Margin: **19.75%**
* Total Discount: **$6,245**

---

### 📈 Visualizations

* Revenue by Region
* Product Sales by Category
* Monthly Sales Trend
* Branch Performance
* Discount Impact on Profit

---

### 🎛️ Filters

* Year
* Loyalty Status

---

### ⭐ Data Model

```text
          Product_Table
               ↑
Customer_Table ← Sales → StoreBranch
```

✔ Star schema implemented
✔ Ambiguous relationships removed
✔ Optimized for DAX calculations

---

## 📈 4. DAX Measures

```dax
Total Revenue = SUM(Sales[total_amount])
```

```dax
Total Cost =
SUMX(
    Sales,
    Sales[quantity] * RELATED(Product_Table[cost_price])
)
```

```dax
Total Discount =
SUMX(
    Sales,
    Sales[total_amount] * Sales[discount]
)
```

```dax
Total Profit =
[Total Revenue] - [Total Cost] - [Total Discount]
```

```dax
Profit Margin % =
DIVIDE([Total Profit], [Total Revenue])
```

---

## 📊 5. Key Insights

* North region generates the highest revenue
* Electronics and Clothing lead product sales
* Sales peak mid-year with noticeable fluctuations
* Higher discounts reduce overall profit margins
* Loyal customers exhibit stronger purchasing behavior
* Branch performance varies significantly, with clear leaders and underperformers

---

## 🧠 6. Challenges & Solutions

| Challenge                      | Solution                                             |
| ------------------------------ | ---------------------------------------------------- |
| Missing columns during INSERT  | Verified schema using `INFORMATION_SCHEMA`           |
| Multiple branches per customer | Assigned most frequent branch using window functions |
| Incorrect discount aggregation | Fixed using `SUMX` in DAX                            |
| Incorrect profit margin        | Used `DIVIDE()` and proper formatting                |
| Excel #SPILL! errors           | Applied structured references                        |
| Month sorting issue            | Created Month Number column                          |
| Ambiguous relationships        | Implemented star schema                              |

---

## 🏆 7. Skills Demonstrated

* **Excel**: Data generation, IFS, VLOOKUP, Power Query
* **SQL**: CTEs, JOINs, UPDATE, DELETE, Window Functions
* **Power BI**: DAX, Data Modeling, Dashboard Design
* **Analytics**: Profitability analysis, segmentation, trend analysis

---

## 🚀 Conclusion

This project demonstrates a full analytics pipeline—from synthetic data creation to insight generation—using industry-standard tools. It reflects practical experience in data transformation, modeling, and business-driven analysis.

---
Author
Adigun Oladimeji 
Analystdims@gmail.com
