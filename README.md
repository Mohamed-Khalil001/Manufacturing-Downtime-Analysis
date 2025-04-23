![image](https://github.com/user-attachments/assets/7f49b600-697c-42ef-a231-44c4f9e11d5d)
![image](https://github.com/user-attachments/assets/c0e2561a-85b0-4036-a40e-968bff5e461e)
![image](https://github.com/user-attachments/assets/b36c127e-6f8f-4ec3-88b5-eaa32c47f2e4)

# 🏭 Manufacturing Downtime Insights

This project analyzes the **weekly operational and financial performance** of a medium-sized beverage manufacturing plant.  
Using a dataset from [Kaggle](https://www.kaggle.com/datasets/agungpambudi/predict-manufacturing-downtime-performance-dataset), we explore **production batches**, **downtime events**, and their **financial impact**.

The goal is to **identify high-cost downtime causes** and deliver actionable insights—such as **optimizing maintenance schedules** or **enhancing operator training**—to improve overall efficiency.

---

## ![image](https://github.com/user-attachments/assets/d55f9689-9fe7-4a1e-ba23-c284b0d992e5) Project Structure

The analysis is structured into **five key stages**, each building on the previous:

1. **📥 Stage 1: Importing & Cleaning**  
   Load raw data, clean and standardize formats, and resolve inconsistencies.

2. **🧠 Stage 2: Modeling**  
   Define relationships between tables to create a unified, queryable data model.

3. **🔧 Stage 3: Data Transformation**  
   Calculate financial metrics such as lost revenue and downtime cost using DAX.

4. **📊 Stage 4: Visualization**  
   Design intuitive dashboards and charts to highlight key performance indicators.

5. **🔍 Stage 5: Insights & Recommendations**  
   Identify root causes, detect patterns, and propose efficiency improvements.
---
---
## ![image](https://github.com/user-attachments/assets/e266c9c7-bc69-4535-b1d7-71d44eb093cb) Stage 1: Importing, Cleaning, and Preparing the Data

### ▪️ 1.1 Data Import from External Source
**Source:** Kaggle - [Predict Manufacturing Downtime Performance Dataset](https://www.kaggle.com/datasets/agungpambudi/predict-manufacturing-downtime-performance-dataset)

**Main Tables:**

| Table                  | Description |
|------------------------|-------------|
| `line-productivity.csv` | Daily production details (Batch, Product, Start, End) |
| `products.csv`          | Product information (Size, Flavor, Min Batch Time) |
| `line-downtime.csv`     | Downtime causes and durations for each Batch |
| `downtime-factors.csv`  | Factor descriptions and operator error flags |

All tables were loaded into **Power BI** for cleaning and transformation.

_______________________________


### ▪️ 1.2 Data Cleaning in Power Query

**Key Cleaning Steps:**
- **Remove Empty Rows:** Applied to all tables.
- **Fix Headers:** Used "Use First Row as Headers" where needed.
- **Set Data Types:**
  - `Start Time` / `End Time` → Date/Time
  - `Downtime Minutes` → Decimal Number
  - Text fields → Text type
- **Remove Rows with Missing Values:** Especially missing `Batch` or `Product`
- **Filter Outliers:** Removed records where `End Time < Start Time` without justification

#### ⚡ Resolving Cross-Midnight Issues
Some batches crossed midnight, resulting in negative durations. Handled using:
```powerquery
if [End Time] < [Start Time] then [End Time] + #duration(1,0,0,0) else [End Time]
```

#### ⏱ Actual Production Time (Hours)
```powerquery
Duration.TotalHours(Duration.From([Adjusted End Time] - [Start Time]))
```
---
## 📆 Summary
**Stage 1:** Imported, cleaned, and prepared the dataset from Kaggle using Power Query
- Removed empty rows, adjusted headers and types
- Fixed midnight issues
- Added adjusted time and actual production time

---
## ![image](https://github.com/user-attachments/assets/2b9803ac-6e30-4eb4-8ad2-d906cf3f4f78) Stage 2: Modeling

### 🎯 Objective
Define relationships between tables in Power BI to build an integrated data model for accurate analysis.

---

### 🔗 2.1 Establishing Relationships

Relationships were defined in **Model View** using shared keys with **single-direction** (one-way) filtering.

| Relationship | Type         | Description |
|--------------|--------------|-------------|
| `Line Productivity` ← `Product` → `Products` | Many-to-One | Links production batches to product details |
| `Line Downtime` ← `Batch` → `Line Productivity` | Many-to-One | Connects downtime events to production batches |
| `Line Downtime` ← `Factor` → `Downtime Factors` | Many-to-One | Associates downtime causes with their descriptions |

---

### 🧭 2.2 Relationship Schema

```text
Products
   ↑
Line Productivity
   ↑
Line Downtime
   ↓
Downtime Factors
```

All relationships are **Many-to-One** and ensure clear data flow from raw events to descriptive dimensions.

---

### ✅ 2.3 Validation

- Tested relationships by building **simple reports** to confirm correct data linkage.
- Verified that DAX functions like `RELATED()` pulled accurate values across all joins.

> **Note:** 7 `Batch IDs` in `line-downtime.csv` were not found in `line-productivity.csv`. These were **addressed and removed** in Stage 1 (Handling Missing Batches).

---

### 💡 2.4 Benefits

- Enabled powerful DAX calculations using `RELATED()` and other functions.
- Ensured accurate connection between downtime causes, durations, production lines, and product metadata.
- Prepared a robust foundation for financial and operational analysis in Power BI.


---

## ![image](https://github.com/user-attachments/assets/fddde30c-d35b-4636-b67b-fc4f8485994a) Stage 3: Data Transformation for Analysis

### ▪️ 3.1 Adding Financial Metrics

#### 🌍 Product Prices
| Size     | Price/Unit (USD) |
|----------|------------------|
| 600 ml   | $0.381           |
| 2 L      | $1.70            |

Added using **Conditional Column** in `products.csv`.

#### 💸 Downtime Factors Costs

| Factor               | Maintenance | Training |
|----------------------|-------------|----------|
| Machine failure      | 300         | 0        |
| Emergency stop       | 300         | 0        |
| Conveyor belt jam    | 300         | 0        |
| Labeling error       | 0           | 50       |
| Calibration error    | 0           | 50       |
| Label switch         | 0           | 50       |
| Others               | 0           | 0        |

Created as **two columns** in Power Query based on factor descriptions.

---

### ▪️ 3.2 Calculated Metrics Using DAX

#### ⚖️ Units Lost
```dax
Units Lost =
IF(
    RELATED('Products'[Size]) = "600 ml",
    ('Line Downtime'[Downtime Minutes] / 60) * 2000,
    ('Line Downtime'[Downtime Minutes] / 60) * 1000
)
```

#### 💰 Revenue Lost
```dax
Revenue Lost =
'Line Downtime'[Units Lost] * RELATED('Products'[Price per Unit])
```

#### 📉 Additional Costs
```dax
Total Maintenance Cost =
IF('Line Downtime'[Downtime Minutes] > 0,
   RELATED('Downtime Factors'[Maintenance Cost]),
   0)

Total Training Cost =
IF('Line Downtime'[Downtime Minutes] > 0,
   RELATED('Downtime Factors'[Training Cost]),
   0)
```

#### 💡 Total Cost
```dax
Total Cost =
'Line Downtime'[Downtime Cost] +
'Line Downtime'[Total Maintenance Cost] +
'Line Downtime'[Total Training Cost]
```

---

### ▪️ 3.3 Handling Missing Batches

**Issue:** 7 Batch IDs in `line-downtime.csv` were not found in `line-productivity.csv`, causing blank values in `RELATED()`-based calculations.

**Solution:**
- Identified missing Batches
- Removed them using Power Query filters

**Code Example:**
```powerquery
if Table.HasAnyRows(Table.SelectRows(#"Line Productivity", each [Batch] = [Batch])) then "Yes" else "No"
```

**Documentation Note:**
> Seven Batch IDs were removed from the Line Downtime table as they were not found in the Line Productivity table, indicating they were not part of the production line. Adding hypothetical data was avoided to maintain data integrity.

---

### ▪️ 3.4 Additional Transformations

- **Unpivoting Downtime Table:**
  - Converted Factor1, Factor2... columns into rows
- **Index Columns:**
  - Added to support tracking
- **Filtered Zero Downtime Records** to improve performance

---

## 🧾 Assumptions (PepsiCo Case Logic)
| Item                  | Value |
|-----------------------|-------|
| 600 ml Price          | $0.381 |
| 2 L Price             | $1.70  |
| 600 ml Rate           | 2000 units/hr |
| 2 L Rate              | 1000 units/hr |
| Downtime Cost Rate    | $100/hr |
| Maintenance Cost      | $300/event |
| Training Cost         | $50/event |

These values simulate realistic PepsiCo manufacturing assumptions based on publicly available data.

---

## 📆 Summary
**Stage 3:** Transformed dataset for financial and operational insight
- Added pricing, cost, lost units, and revenue
- Calculated total cost and handled missing values
- Restructured data with unpivot and indexing

---
## ![image](https://github.com/user-attachments/assets/5c7b4c91-76c3-4bb9-9e4c-07e625bd49e0) Stage 4: Data Visualization
---
## ![image](https://github.com/user-attachments/assets/bab2824d-2f3b-44ad-a230-a1dc56423364) Stage 5: Data Insights and Story
