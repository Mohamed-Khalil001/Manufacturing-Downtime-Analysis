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
│   ├── Modeling.md
│
├── 🔹Stage 3: Data Transformation for Analysis/
│   ├── Transformation.md
│
├── 🔹Stage 4: Data Visualization/
│   ├── 1-overview .png
│   ├── 2-insights.png
│   ├── Manufacturing-Downtime-Analysis.pbix
|
├── 🔹Stage 5: Data Insights and Story/
│   ├── story.pdf
│   ├── insights_and_recommendations.pdf
|
├── 🔹FINAL PRESENTATION /
│   ├── Manufacturing Downtime.pptx
│ 

```
---
---
## ![image](https://github.com/user-attachments/assets/e266c9c7-bc69-4535-b1d7-71d44eb093cb) Stage 1: Importing, Cleaning, and Preparing the Data

### ▪️ 1.1 Data Import from External Source

**Source:** Kaggle - [Predict Manufacturing Downtime Performance Dataset](https://www.kaggle.com/datasets/agungpambudi/predict-manufacturing-downtime-performance-dataset)

**Main Tables:**

| Table                   | Description                                           |
| ----------------------- | ----------------------------------------------------- |
| `line-productivity.csv` | Daily production details (Batch, Product, Start, End) |
| `products.csv`          | Product information (Size, Flavor, Min Batch Time)    |
| `line-downtime.csv`     | Downtime causes and durations for each Batch          |
| `downtime-factors.csv`  | Factor descriptions and operator error flags          |

All tables were loaded into **Power BI** for cleaning and transformation.

---

### ▪️ 1.2 Data Cleaning in Power Query

---

### 📝 Detailed Steps

The cleaning process involved standardizing data types, fixing headers, removing inconsistencies, and enriching the dataset with financial data to enable operational and financial analysis.

---

### **1. Products Table**

* **Objective**: Standardize the "Size" column and add financial data for analysis.

* **Steps**:

  * **Standardize Size Units**: The "Size" column had mixed units (ml, L) and non-numeric values. We unified all units to liters and converted the column to a numeric (decimal) format for calculations.

    * Replaced "ml" and "L" with empty strings to remove units.
    * Replaced "600 ml" with "0.6" to convert milliliters to liters.
    * Converted the "Size" column to a decimal type.
      ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/2-products.png)

  * **Add Financial Data**: Added a "Price" column using conditional logic to assign prices based on average beverage prices for mid-sized companies:

    * 2 L bottles: \$1.70/unit.
    * 600 ml bottles: \$0.381/unit.

    This enabled financial impact analysis of operational performance.
      ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/4-products.png)

---

### **2. Downtime-Factors Table**

* **Objective**: Enrich the table with financial data to quantify downtime costs.

* **Steps**:

  * The table was relatively clean but lacked financial metrics.
  * Added two columns for financial data:

    * **Maintenance Cost**: Used conditional logic based on the downtime factor:

      * Operator Error: \$50 (average training cost).
      * Mechanical Error: \$300 (average maintenance cost).
    * **Total Cost**: Calculated based on the downtime factor and associated costs.
     ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/7-Downtime_factors.png)

  This enriched the table for financial analysis of downtime events.

---

### **3. Line-Productivity Table**

* **Objective**: Fix timestamp issues and extract day-of-week for daily analysis.

* **Steps**:

  * **Fix Timestamp Errors**: The "Date" column contained full timestamps, which were irrelevant for manufacturing dates. Some production hours spanned across two days (e.g., starting on one day and ending on the next), causing incorrect day attribution.

    * Replaced full timestamps with date-only values.
    * Adjusted production hours spanning two days using an M query:

    ```m
    let
        startDateTime = #datetime(Date.Year([Date]), Date.Month([Date]), Date.Day([Date]), Time.Hour([Start Time]), Time.Minute([Start Time]), Time.Second([Start Time])),
        endDateTimeRaw = #datetime(Date.Year([Date]), Date.Month([Date]), Date.Day([Date]), Time.Hour([End Time]), Time.Minute([End Time]), Time.Second([End Time])),
        endDateTime = if [End Time] < [Start Time] then endDateTimeRaw + #duration(1, 0, 0, 0) else endDateTimeRaw
    in
        Duration.TotalHours(endDateTime - startDateTime)
    ```

    This ensured accurate duration calculations by adding 24 hours to the end time when it was less than the start time, then calculating the difference.
     ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/9-line_productivity.png)

  * **Extract Day of Week**: The dataset lacked a "Day of Week" column, necessary for daily performance analysis. Added a custom column using an M query:

    ```m
    Date.ToText([Date], "dddd")
    ```

    This extracted the day name (e.g., Monday) for each date.
     ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/10-line_productivity.png)

---

### **4. Line-Downtime Table**

* **Objective**: Clean inconsistencies, reduce complexity, and add financial data.

* **Steps**:

  * **Handle Null Values**: Replaced "nil" values with 0 to ensure data consistency.

  * **Simplify Structure**: Unpivoted columns to reduce the table from 13 to 3 columns for better usability.

  * **Clean Headers**: Replaced "Factor" values containing "nil" with clean descriptions to improve readability.
     ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/12-line_downtime.png)

  * **Add Financial Data**: Added a custom column to calculate downtime cost based on an average downtime cost of \$100/hour for a mid-sized beverage plant:

    * Formula: `Downtime Cost = Downtime Hours * 100`.

    This enabled financial impact analysis of downtime events.
     ![See](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/8940504a1205ced6093068481d6a831653c345bd/%F0%9F%94%B9Stage%201%20Importing%20Cleaning%20and%20Preparing%20the%20Data/applied_steps/13-line_downtime.png)

---

### 📄 Summary

**Stage 1:** Successfully cleaned and enriched four tables (**Products**, **Downtime-Factors**, **Line-Productivity**, **Line-Downtime**) by standardizing data types, fixing inconsistencies, and adding financial metrics. The dataset is now ready for analysis in Power BI.

---

---
## ![image](https://github.com/user-attachments/assets/2b9803ac-6e30-4eb4-8ad2-d906cf3f4f78) Stage 2: Modeling

### 🎯 Objective
Before completing the second part of data transformation inside Power BI, we need to build a solid data model to ensure proper relationships between tables. A well-structured model is essential for enabling accurate analysis, meaningful measures, and smooth transformation logic in Power BI.

### 📝 Context
Data transformation in this project is divided into two main parts:

#### 1-✅ Part 1 (Power Query): Initial cleaning and standardization of data, which has already been completed.
#### 2-🔄 Part 2 (Power BI interface): Additional transformations, calculations, and business logic to be applied after the data model is built. 
 
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
![](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/60ccac6a79cdf5fb520d1e20c9c527622854a116/%F0%9F%94%B9Stage%202%20Modeling/Modeling.png)

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

- **Solution**:  
  - Identified the missing Batch IDs in Power Query.  
  - Filtered out these batches (instead of removing them) to exclude them from analysis while preserving the original data


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
![Page 1 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/846f7cff7e88482826f4a1228639d45e97a256da/%F0%9F%94%B9Stage%204%20Data%20Visualization/2-Overview.PNG)

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
    Number of Downtime Events = CALCULATE(
    COUNTROWS('Line-Downtime'),
    'line-downtime'[Downtime(min)] > 0)
    ```
  - **Top Costly Factor**:
    ```dax
    Top Costly Factor = 
    CALCULATE(
    SELECTEDVALUE('downtime-factors'[Description]),
    FILTER(
        'line-downtime',
        'Line-Downtime'[Total Cost] = MAX('line-downtime'[Total Cost])
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
    IF('line-downtime'[Downtime(min)] > 0, 
    RELATED('downtime-factors'[Maintenance Cost $]), 
    0)
    ```
  - **Total Training Cost**:
    ```dax
    Total Training Cost = 
    IF('line-downtime'[Downtime(min)]> 0, 
    RELATED('downtime-factors'[Training Cost $]), 
     0)
    ```

###### 3. Donut Chart - Revenue Lost by Flavor
- **Description**: Shows revenue losses across product flavors (Orange, Lemon Lime, Cola, Diet Cola).
- **Purpose**: Highlight flavors most affected by downtime.
- **Data and DAX**:
  - Values: `Revenue Lost`:
    ```dax
    Revenue Lost = 
     'line-downtime'[Units Lost] * RELATED('Products'[Price per Unit])
    ```

###### 4. Horizontal Bar Chart - Operator Errors by Operator
- **Description**: Shows the number of errors per operator.
- **Purpose**: Identify operators needing additional training.
- **Data and DAX**:
  - Values: `Operator Errors Count`:
    ```dax
    Operator Errors Count = 
     CALCULATE(
       COUNTROWS('line-downtime'),
        FILTER(
        'downtime-factors',
        'downtime-factors'[Operator Error] = "Yes"
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
![Page 2 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/846f7cff7e88482826f4a1228639d45e97a256da/%F0%9F%94%B9Stage%204%20Data%20Visualization/3-insights.PNG)

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
     'line-downtime'[Downtime Cost $] + 
     'line-downtime'[Total Maintenance Cost] + 
     'line-downtime'[Total Training Cost]
    ```

###### 2. Treemap - Daily Downtime Events
- **Description**: Shows downtime events across days of the week.
- **Purpose**: Identify days with the most downtime events.
- **Data and DAX**:
  - Values: `Daily Downtime Events`:
    ```dax
    Daily Downtime Events = 
    CALCULATE(
        [Number of Downtime Events],
         ALLEXCEPT('Line-Productivity', 'line-productivity'[Day of week])
     )
    ```


###### 3. Line Chart - Downtime Cost by Size (USD)
- **Description**: Tracks daily downtime costs for 600 ml and 2 L products.
- **Purpose**: Compare financial impact by product size.
- **Data and DAX**:
  - **Cost by Size 600ml**:
    ```dax
    Cost by Size 600 ML = 
     CALCULATE(
      SUM('Line-Downtime'[Total Cost]),
     'products'[Size(L)]=0.6) 
    ```
  - **Cost by Size 2L**:
    ```dax
    Cost by Size 2L = 
     CALCULATE(
      SUM('Line-Downtime'[Total Cost]),
      'products'[Size(L)]=2)
      )
    ```

###### 4. Card - Total Units Lost
- **Description**: Shows total units lost (33K).
- **Purpose**: Quantify production losses due to downtime.
- **Data and DAX**:
  - **Total Units Lost**:
    ```dax
    Total Units Lost = 
      SUM('Line-Downtime'[Units Lost])
    ```
  - **Units Lost**:
    ```dax
     Units Lost = 
      IF(
        RELATED('products'[Size(L)]) = 0.6,
        ('line-downtime'[Downtime(min)] / 60) * 2000,
        ('line-downtime'[Downtime(min)] / 60) * 1000
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
## 1- 📌 Insights and Recommendations

### ✅ Operational Insights

**Highlights the root causes of downtimes and operational performance inside the plant:**

| Operational Indicator          | Value / Description                                            |
| ------------------------------ | -------------------------------------------------------------- |
| Total Number of Downtimes      | 50 Downtimes                                                   |
| Total Downtime Hours           | 18.83 hours                                                    |
| Main Causes of Downtime        | 1) Machine Failure<br>2) Inventory Shortage<br>3) Batch Change |
| Day with the Most Downtimes    | Friday                                                         |
| Day with the Fewest Downtimes  | Tuesday                                                        |
| Operators with the Most Errors | Charlie > Mac > Dee > Dennis                                   |
| Recurring Operational Issues   | Batch Coding Error, Machine Adjustment                         |

---

### ✅ Financial Impact

| Financial Indicator   | Value                  |
| --------------------- | ---------------------- |
| Total Downtime Cost   | 6.43K USD              |
| Total Lost Revenue    | 18.68K USD             |
| Total Lost Units      | 33K Units              |
| Most Affected Product | Cola (75.6% of losses) |

---

## 🔁 Operational and Financial Link - With Actual Figures

| Operational Cause  | Downtime Duration (minutes) | Frequency (Occurrences) | Cost (USD) | Lost Revenue (USD) | Lost Units (Estimate) |
| ------------------ | --------------------------- | ----------------------- | ---------- | ------------------ | --------------------- |
| Machine Failure    | 270                         | 9                       | 3,166.65   | 15,750+            | \~27,800 Units        |
| Inventory Shortage | 30                          | 1                       | 350        | 1,000+             | \~1,700 Units         |
| Batch Change       | 40                          | 2                       | 166.66     | 500+               | \~900 Units           |
| Batch Coding Error | 40                          | 2                       | 166.66     | 500+               | \~900 Units           |
| Machine Adjustment | 20                          | 1                       | 83.33      | 200+               | \~400 Units           |

> ⚠️ **Note:** The revenue loss rate is estimated at \$570 per 1K units.

---

## 📈 Summary of Direct Financial Impact:

- **Largest Financial Loss Cause:** Machine Failure (75% of losses).
- **Top Operator Causing Failures:** Charlie (more than 55 errors).
- **Most Affected Product:** Cola – Accounted for 75.6% of lost revenue.

---

## ✅ Recommendations

### 🔧 Operational:

- Implement proactive equipment maintenance to reduce failures.
- Conduct a root cause analysis of Charlie's performance and improve operator training.
- Reduce batch changeover time and procedures.
- Strengthen supply chain management to avoid downtimes.

### 💰 Financial:

- Reevaluate losses for high-margin products (such as Cola).
- Develop financial KPIs linked to operations to measure performance.
- Compare margin against downtimes to identify the most loss-making lines.

---
## 2- 📌📖 [the Story ](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/254fc1b20dc1699c789f1f70ae2a23decba05307/%F0%9F%94%B9Stage%205%20Data%20Insights%20and%20Story/The_Story.pdf)
