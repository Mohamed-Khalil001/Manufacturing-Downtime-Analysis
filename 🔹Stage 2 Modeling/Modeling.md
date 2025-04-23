## ![image](https://github.com/user-attachments/assets/2b9803ac-6e30-4eb4-8ad2-d906cf3f4f78) Stage 2: Modeling

### 🎯 Objective

Define relationships between tables in Power BI to build an integrated data model for accurate analysis.

---

### 🔗 2.1 Establishing Relationships

Relationships were defined in **Model View** using shared keys with **single-direction** (one-way) filtering.

| Relationship                                    | Type        | Description                                        |
| ----------------------------------------------- | ----------- | -------------------------------------------------- |
| `Line Productivity` ← `Product` → `Products`    | Many-to-One | Links production batches to product details        |
| `Line Downtime` ← `Batch` → `Line Productivity` | Many-to-One | Connects downtime events to production batches     |
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
