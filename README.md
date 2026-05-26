# Call Center ETL & Analytics Project

A complete data pipeline and business intelligence solution for call center performance analysis. The project transforms raw call records into a structured star-schema data warehouse, then visualizes KPIs through a Power BI dashboard.

---

## Project Files

| File | Description |
|---|---|
| `Call-Center-Dataset.xlsx` | Raw source data (5,000 call records) |
| `CallCenter_ETL_Code.ipynb` | Python ETL pipeline (extract, transform, load) |
| `CallCenter_Warehouse_Output.xlsx` | Transformed data warehouse output |
| `CallCenter_Dashboard.pbix` | Power BI dashboard for KPI visualization |

---

## Data Source

The raw dataset (`Call-Center-Dataset.xlsx`) contains **5,000 call records** with the following fields:

| Column | Description |
|---|---|
| `Call Id` | Unique call identifier (e.g. `ID0001`) |
| `Date` | Date of the call |
| `Agent` | Agent name (Becky, Dan, Diane, Greg, Jim, Joe, Martha, Stewart) |
| `Agent Group` | Agent team (Group A through Group E) |
| `Answered (Y/N)` | Whether the call was answered |
| `Issue Resolved?` | Whether the customer's issue was resolved |
| `Resolved within SLA?` | Whether resolution met the SLA target |
| `First Call Resolution` | Whether the issue was resolved on first contact |
| `Speed of Answer (Secs)` | Time from call arrival to agent pickup |
| `Talk Time (Secs)` | Duration of active conversation |
| `Hold Time (Secs)` | Total time the caller was on hold |
| `Satisfaction rating` | Customer satisfaction label (Extremely Dissatisfied → Extremely Satisfied) |
| `Main Call Reason` | Top-level call category |
| `Sub Call Reason` | Detailed call reason |

---

## ETL Pipeline

The notebook `CallCenter_ETL_Code.ipynb` runs a full **Extract → Transform → Load** pipeline using Python (`pandas`, `openpyxl`).

### How to run

```bash
# Install dependencies
pip install pandas openpyxl numpy

# Run the ETL
jupyter nbconvert --to notebook --execute CallCenter_ETL_Code.ipynb
# or simply open the notebook and run all cells
```

The script reads `Call-Center-Dataset.xlsx` from the working directory and writes `CallCenter_Warehouse_Output.xlsx`.

### Transform steps

- **Validation** — checks all required columns are present; logs row count and any null/unexpected values
- **Y/N encoding** — converts `Y`/`N` flags to `1`/`0` integers
- **AHT derivation** — calculates Average Handle Time as `Talk Time + Hold Time`
- **Satisfaction scoring** — maps text labels to a numeric 1–5 scale
- **Dimension building** — generates surrogate keys for all dimension tables
- **Fact table assembly** — joins all surrogate keys onto the call records
- **KPI summary** — computes headline metrics across all answered calls
- **Styled output** — applies color-coded headers and auto-sized columns per sheet

---

## Data Warehouse Schema

The output file `CallCenter_Warehouse_Output.xlsx` follows a **star schema** with one fact table and five dimension tables.

```
                    Dim_Date
                       │
Dim_Agent_Group        │
       │               │
   Dim_Agent ──── Fact_Table ──── Dim_Satisfaction
                       │
                   Dim_Reason
```

### Fact_Table

Central table with one row per call (5,000 rows).

| Column | Type | Description |
|---|---|---|
| `Call Id` | String | Natural key from source |
| `Date_Key` | Integer | FK → `Dim_Date.Date_Key` (YYYYMMDD) |
| `Agent_ID` | Integer | FK → `Dim_Agent.Agent_ID` |
| `Answered (Y/N)` | 0/1 | |
| `Issue Resolved?` | 0/1 | |
| `Resolved within SLA?` | 0/1 | |
| `First Call Resolution` | 0/1 | |
| `Speed of Answer (Secs)` | Float | Null for unanswered calls |
| `Talk Time (Secs)` | Float | Null for unanswered calls |
| `Hold Time (Secs)` | Float | Null for unanswered calls |
| `AHT` | Float | `Talk Time + Hold Time` |
| `Satisfaction_Score` | Integer | 1–5 numeric score |
| `Satisfaction_rating_ID` | Integer | FK → `Dim_Satisfaction` |
| `Reason_ID` | Integer | FK → `Dim_Reason` |

### Dim_Agent

| Column | Description |
|---|---|
| `Agent_ID` | Surrogate key |
| `Agent` | Agent name |
| `Agent_Group_ID` | FK → `Dim_Agent_Group` |

### Dim_Agent_Group

| Column | Description |
|---|---|
| `Agent_Group_ID` | Surrogate key |
| `Agent Group` | Group name (A–E) |

### Dim_Satisfaction

| Column | Description |
|---|---|
| `Satisfaction_rating_ID` | Surrogate key (0 = unanswered) |
| `Satisfaction rating` | Label |
| `Satisfaction_Score` | Numeric score (1–5) |

### Dim_Reason

| Column | Description |
|---|---|
| `Reason_ID` | Surrogate key (0 = unanswered) |
| `Main Call Reason` | Category (Billing, Calls & SMS, Content Services, Devices, Internet, Packages and Promos, Payment) |
| `Sub Call Reason` | Specific reason (e.g. High Bill Amount, Disconnection) |

### Dim_Date

Spine of all dates in the dataset, with pre-calculated attributes:

| Column | Description |
|---|---|
| `Date_Key` | Integer key (YYYYMMDD) |
| `Full_Date` | Date value |
| `Day`, `Day_Name` | Day of month and weekday name |
| `Month_Num`, `Month` | Month number and name |
| `Quarter` | Quarter number |
| `Year` | Year |
| `Is_Weekday` | 1 if Mon–Fri, 0 if weekend |

---

## KPI Summary

Headline metrics computed over all **4,054 answered calls** (81.08% answer rate):

| KPI | Value |
|---|---|
| Total Calls | 5,000 |
| Answered Calls | 4,054 |
| Unanswered Calls | 946 |
| Answer Rate | 81.08% |
| Avg Handle Time (AHT) | 301.03 secs |
| Avg Speed of Answer | 67.52 secs |
| Avg Talk Time | 224.92 secs |
| Avg Hold Time | 76.11 secs |
| Avg Satisfaction Score | 3.40 / 5 |
| First Call Resolution Rate | 65.47% |
| Issue Resolution Rate | 89.94% |
| SLA Compliance Rate | 63.86% |

---

## Dashboard

Open `CallCenter_Dashboard.pbix` in **Power BI Desktop** to explore the interactive dashboard. The `.pbix` connects to the warehouse output and visualizes performance by agent, team, call reason, time period, and satisfaction tier.

**Requirements:** Power BI Desktop (free) — [download here](https://powerbi.microsoft.com/desktop)

---

## Dependencies

```
Python  >= 3.8
pandas
openpyxl
numpy
```

Install with:

```bash
pip install pandas openpyxl numpy
```
