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

**Stage 2:** Successfully cleaned and enriched four tables (**Products**, **Downtime-Factors**, **Line-Productivity**, **Line-Downtime**) by standardizing data types, fixing inconsistencies, and adding financial metrics. The dataset is now ready for analysis in Power BI.

---



