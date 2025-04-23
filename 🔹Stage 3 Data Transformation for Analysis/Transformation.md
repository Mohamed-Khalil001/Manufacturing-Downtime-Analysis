## ![image](https://github.com/user-attachments/assets/fddde30c-d35b-4636-b67b-fc4f8485994a) Stage 3: Data Transformation for Analysis

### ▪️ 3.1 Adding Financial Metrics

#### 🌍 Product Prices

| Size   | Price/Unit (USD) |
| ------ | ---------------- |
| 600 ml | $0.381           |
| 2 L    | $1.70            |

Added using **Conditional Column** in `products.csv`.

#### 💸 Downtime Factors Costs

| Factor            | Maintenance | Training |
| ----------------- | ----------- | -------- |
| Machine failure   | 300         | 0        |
| Emergency stop    | 300         | 0        |
| Conveyor belt jam | 300         | 0        |
| Labeling error    | 0           | 50       |
| Calibration error | 0           | 50       |
| Label switch      | 0           | 50       |
| Others            | 0           | 0        |

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

| Item               | Value         |
| ------------------ | ------------- |
| 600 ml Price       | $0.381        |
| 2 L Price          | $1.70         |
| 600 ml Rate        | 2000 units/hr |
| 2 L Rate           | 1000 units/hr |
| Downtime Cost Rate | $100/hr       |
| Maintenance Cost   | $300/event    |
| Training Cost      | $50/event     |

These values simulate realistic PepsiCo manufacturing assumptions based on publicly available data.

---

## 📆 Summary

**Stage 3:** Transformed dataset for financial and operational insight

- Added pricing, cost, lost units, and revenue
- Calculated total cost and handled missing values
- Restructured data with unpivot and indexing
