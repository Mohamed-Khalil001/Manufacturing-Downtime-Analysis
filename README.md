![image](https://github.com/user-attachments/assets/b36c127e-6f8f-4ec3-88b5-eaa32c47f2e4) ![image](https://github.com/user-attachments/assets/c05601f8-051c-49c0-95a3-a62e950a21a4)

# Manufacturing Downtime

This project analyzes the **weekly operational and financial performance** of a medium-sized beverage manufacturing plant.  
Using a dataset from [Kaggle](https://www.kaggle.com/datasets/agungpambudi/predict-manufacturing-downtime-performance-dataset), we explore **production batches**, **downtime events**, and their **financial impact**.

The goal is to **identify high-cost downtime causes** and deliver actionable insights—such as **optimizing maintenance schedules** or **enhancing operator training**—to improve overall efficiency.

---

## 📂 Project Structure

The repository is organized as follows:
```
Investment-Dashboards/
│
├── 🔹Stage 1: Importing Cleaning and Preparing the Data/
│   ├── Cleaning.md
│   ├── Whole_Data.zip
│   ├── downtime-factors.csv
│   ├── line-downtime.csv
│   ├── line-productivity.csv
│   ├── products.csv
│   ├── metadata.csv
│
├── 🔹Stage 2: Modeling/
│   ├── investment_report.pbix
│
├── 🔹Stage 3: Data Transformation for Analysis/
│   ├── market_overview.png
│   ├── financial_performance.png
│   ├── sector_analysis.png
│   ├── final_story.png
│
├── 🔹Stage 4: Data Visualization/
│   ├── story.pdf
│   ├── insights_and_recommendations.pdf
|
├── 🔹Stage 5: Data Insights and Story/
│   ├── story.pdf
│   ├── insights_and_recommendations.pdf
|
├── 🔹FINAL PRESENTATION /
│   ├── story.pdf
│   ├── insights_and_recommendations.pdf

```
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

### 🎯 Objective
Create interactive Power BI dashboard to analyze manufacturing downtime, focusing on financial and operational performance (Page 1) and detailed daily insights (Page 2). The goal is to identify high-cost downtime causes and support data-driven decisions.

---

### 📊 Dashboards Created

#### 🟪 Page 1: Weekly Manufacturing Downtime 
This page provides a high-level overview of weekly performance, focusing on financial impacts, key downtime factors, and operator performance.

##### 📸 Screenshot of Page 1
![Page 1 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/ba87059bee7d37e3d054bbebdf8b19581a2a2506/%F0%9F%94%B9Stage%204%20Data%20Visualization/1-overview%20.png)

##### 📝 Annotations for Page 1 Screenshot
1. **KPIs**: Displays key financial and operational metrics.
2. **Stacked Area Chart**: Shows top 5 downtime factors by cost.
3. **Donut Chart**: Displays revenue losses by flavor.
4. **Horizontal Bar Chart**: Shows operator errors by operator.
5. **Slicers**: Filters by Day of Week, Operator, and Product.

##### 📈 Visuals Breakdown

###### 1. KPIs (Cards)
- **Description**: Five cards showing weekly performance metrics.
- **Purpose**: Quick snapshot of financial and operational impact.
- **Metrics and DAX**:
  - **Total Downtime Cost**:
    ```dax
    Total Downtime Cost = SUM('Line Downtime'[Total Cost])
    ```
  - **Total Revenue Lost**:
    ```dax
    Total Revenue Lost = SUM('Line Downtime'[Revenue Lost])
    ```
  - **Total Downtime Hours**:
    ```dax
    Total Downtime Hours = SUM('Line Downtime'[Downtime Minutes]) / 60
    ```
  - **Total Downtime Events**:
    ```dax
    Total Downtime Events = COUNTROWS('Line Downtime')
    ```
  - **Top Costly Factor**:
    ```dax
    Top Costly Factor = 
    CALCULATE(
        SELECTEDVALUE('Downtime Factors'[Description]),
        FILTER(
            'Line Downtime',
            'Line Downtime'[Total Cost] = MAX('Line Downtime'[Total Cost])
        )
    )
    ```

###### 2. Stacked Area Chart - Top 5 Downtime Factors by Cost
- **Description**: Shows the top 5 downtime factors by cost (Downtime Cost, Maintenance Cost, Training Cost).
- **Purpose**: Identify the most expensive downtime causes for corrective actions.
- **Data and DAX**:
  - **Downtime Cost**:
    ```dax
    Downtime Cost = ('Line Downtime'[Downtime Minutes] / 60) * 100
    ```
  - **Total Maintenance Cost**:
    ```dax
    Total Maintenance Cost = 
    IF('Line Downtime'[Downtime Minutes] > 0, 
       RELATED('Downtime Factors'[Maintenance Cost]), 
       0)
    ```
  - **Total Training Cost**:
    ```dax
    Total Training Cost = 
    IF('Line Downtime'[Downtime Minutes] > 0, 
       RELATED('Downtime Factors'[Training Cost]), 
       0)
    ```

###### 3. Donut Chart - Revenue Lost by Flavor
- **Description**: Shows revenue losses across product flavors (Orange, Lemon Lime, Cola, Diet Cola).
- **Purpose**: Highlight flavors most affected by downtime.
- **Data and DAX**:
  - Values: `Total Revenue Lost`:
    ```dax
    Total Revenue Lost = SUM('Line Downtime'[Revenue Lost])
    ```

###### 4. Horizontal Bar Chart - Operator Errors by Operator
- **Description**: Shows the number of errors per operator.
- **Purpose**: Identify operators needing additional training.
- **Data and DAX**:
  - Values: `Operator Errors Count`:
    ```dax
    Operator Errors Count = 
    CALCULATE(
        COUNTROWS('Line Downtime'),
        FILTER(
            'Downtime Factors',
            'Downtime Factors'[Operator Error] = "Yes"
        )
    )
    ```

###### 5. Slicers
- **Description**: Filters for Day of Week, Operator, and Product.
- **Purpose**: Enable customized analysis by filtering data.

---

#### 🟪 Page 2: Weekly Manufacturing Downtime 
This page provides detailed insights into daily downtime events, cost by product size, and production losses.

##### 📸 Screenshot of Page 2
![Page 2 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/ba87059bee7d37e3d054bbebdf8b19581a2a2506/%F0%9F%94%B9Stage%204%20Data%20Visualization/2-insights.png)

##### 📝 Annotations for Page 2 Screenshot
1. **Table**: Lists the top 5 most costly downtime events.
2. **Treemap**: Shows downtime events by day of the week.
3. **Line Chart**: Tracks daily downtime costs by product size.
4. **Card**: Shows total units lost.
5. **Slicers**: Filters by Day of Week, Operator, and Product.

##### 📈 Visuals Breakdown

###### 1. Table - Top 5 Costly Downtime Events
- **Description**: Lists the top 5 downtime events (Batch, Factor Description, Downtime Minutes, Total Cost).
- **Purpose**: Identify the most expensive events for corrective actions.
- **Data and DAX**:
  - **Total Cost**:
    ```dax
    Total Cost = 
    'Line Downtime'[Downtime Cost] +
    'Line Downtime'[Total Maintenance Cost] +
    'Line Downtime'[Total Training Cost]
    ```

###### 2. Treemap - Daily Downtime Events
- **Description**: Shows downtime events across days of the week.
- **Purpose**: Identify days with the most downtime events.
- **Data and DAX**:
  - Values: `Daily Downtime Events`:
    ```dax
    Daily Downtime Events = 
    CALCULATE(
        [Total Downtime Events],
        ALLEXCEPT('Line Productivity', 'Line Productivity'[Day Name])
    )
    ```
  - **Total Downtime Events**:
    ```dax
    Total Downtime Events = COUNTROWS('Line Downtime')
    ```

###### 3. Line Chart - Downtime Cost by Size (USD)
- **Description**: Tracks daily downtime costs for 600 ml and 2 L products.
- **Purpose**: Compare financial impact by product size.
- **Data and DAX**:
  - **Cost by Size 600ml**:
    ```dax
    Cost by Size 600ml = 
    CALCULATE(
        SUM('Line Downtime'[Total Cost]),
        'Products'[Size] = "600 ml"
    )
    ```
  - **Cost by Size 2L**:
    ```dax
    Cost by Size 2L = 
    CALCULATE(
        SUM('Line Downtime'[Total Cost]),
        'Products'[Size] = "2 L"
    )
    ```

###### 4. Card - Total Units Lost
- **Description**: Shows total units lost (33K).
- **Purpose**: Quantify production losses due to downtime.
- **Data and DAX**:
  - **Total Units Lost**:
    ```dax
    Total Units Lost = SUM('Line Downtime'[Units Lost])
    ```
  - **Units Lost**:
    ```dax
    Units Lost = 
    IF(
        RELATED('Products'[Size]) = "600 ml",
        ('Line Downtime'[Downtime Minutes] / 60) * 2000,
        ('Line Downtime'[Downtime Minutes] / 60) * 1000
    )
    ```

###### 5. Slicers
- **Description**: Filters for Day of Week, Operator, and Product.
- **Purpose**: Enable detailed analysis by filtering data.

---

### 🔄 Slicer Synchronization
Slicers are synchronized between both pages for consistent filtering.

---

### 💡 Benefits
- **Page 1**: High-level overview of financial and operational impacts.
- **Page 2**: Detailed insights into daily events, cost by size, and production losses.
- Interactive slicers enable customized analysis.

---
## ![image](https://github.com/user-attachments/assets/bab2824d-2f3b-44ad-a230-a1dc56423364) Stage 5: Data Insights and Story
