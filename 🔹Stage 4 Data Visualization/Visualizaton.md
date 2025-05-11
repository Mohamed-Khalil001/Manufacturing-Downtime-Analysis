## ![image](https://github.com/user-attachments/assets/5c7b4c91-76c3-4bb9-9e4c-07e625bd49e0) Stage 4: Data Visualization

### 🎯 Objective

Create interactive Power BI dashboard to analyze manufacturing downtime, focusing on financial and operational performance (Page 1) and detailed daily insights (Page 2). The goal is to identify high-cost downtime causes and support data-driven decisions.

---

### 📊 Dashboards Created

#### 🟪 Page 1: Weekly Manufacturing Downtime

This page provides a high-level overview of weekly performance, focusing on financial impacts, key downtime factors, and operator performance.

##### 📸 Screenshot of Page 1

![Page 1 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/143c0f952d0ca58ee19886d92e43c5a6da6b573c/%F0%9F%94%B9Stage%204%20Data%20Visualization/2-Overview.PNG)

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

![Page 2 Screenshot](https://github.com/Mohamed-Khalil001/Manufacturing-Downtime-Analysis/blob/143c0f952d0ca58ee19886d92e43c5a6da6b573c/%F0%9F%94%B9Stage%204%20Data%20Visualization/3-insights.PNG)

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
