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
