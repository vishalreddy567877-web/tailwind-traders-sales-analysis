
# 📊 Tailwind Traders: End-to-End Sales Performance & Profitability Analytics Suite

## 🔗 Project Deliverables
* **Production Report File:** `dashboard/Tailwind_Traders_Report.pbix`
* **Core Source Dataset:** `data/Tailwind Traders Sales.xlsx`
* **Dynamic Currency Integration:** Embedded Pandas Python ETL Script

---

## 🎯 Strategic Business Case
Executive leadership at **Tailwind Traders** required a centralized, high-performance business intelligence architecture to unify global operational data. The primary objective was to eliminate visibility gaps regarding top-line revenue velocity and protect bottom-line profit margins across international markets. 

This project delivers an end-to-end data engineering and analytics solution, migrating raw Excel transactional data into an optimized star schema. It features separate diagnostic reporting environments for the Sales and Finance teams, a consolidated cloud-hosted executive dashboard, and automated data-governance guardrails to drive proactive enterprise decision-making.

---

## 🛠️ Technical Architecture & Ecosystem
* **Data Layer:** Microsoft Excel (Sales & Purchase records) and an Ingested Python Script (Currency Exchange Data).
* **ETL Pipeline:** Power Query & Excel Formulas for schema formatting, field validation, and strict data type profiling.
* **Data Modeling:** Multi-fact star schema optimization utilizing bi-directional cross-filtering and explicit relationship cardinality mapping.
* **Analytics Engine:** DAX (Data Analysis Expressions) for time-intelligence calculations, custom dimensions, and profitability metrics.
* **Optimization & Deployment:** Performance Analyzer validation (<200ms target latency), Mobile View responsive design, and automated cloud triggers/alerts.

---

## 🚀 End-to-End Development Lifecycle

### 🗂️ Phase 1: Ingestion & Data Engineering (ETL)
<details>
<summary><b>Click to expand Data Prep & Extraction details</b></summary>

* **Fact Table Processing:** Extracted and validated transactional records from `Tailwind Traders Sales.xlsx`. Verified 100% data integrity and validity across system identity keys (`OrderID`) using the Column Quality and Profile indicators.
* **Calculated Fields (Excel):** Meticulously engineered three downstream core revenue attributes prior to ingestion:
  * `Gross Revenue`: Calculated via `=Gross Product Price * Quantity Purchased` to establish baseline top-line metrics.
  * `Total Tax`: Mapped via `=Tax Per Product * Quantity Purchased` to isolate regional tax liabilities.
  * `Net Revenue`: Evaluated via `=Gross Revenue - Total Tax` to extract post-tax operational earnings per item.
* **Schema Constraints & Ingestion:** Loaded the calculated records into Power BI. Strictly cast data types to minimize memory footprints (`Gross Product Price` & `Tax Per Product` set to *Fixed Decimal Number*; `Quantity Purchased` cast to *Whole Number*; `Product Category` cast to *Text*).
* **Secondary Dimension Ingestion:** Loaded the regional dimensional attributes via a separate `Countries` source file, ensuring schema data types (`Country ID` & `Exchange ID` as *Whole Number*, `Country` as *Text*) matched exactly.
* **Dynamic Python Ingestion:** Engineered a lightweight Python script utilizing `pandas` and `io.StringIO` to read, parse, and dynamically transform raw currency exchange strings into a tabular format within Power BI to mitigate data silo risks.
  ```python
  import pandas as pd
  from io import StringIO
  
  data = """Exchange ID; ExchangeRate; Exchange Currency
  1;1;USD
  2;0,75;GBP
  3;0,85;EUR
  4;3,67;AED
  5;1,3;AUD"""
  
  df = pd.read_csv(StringIO(data), sep=';')
  # Return the transformed dataframe for relationship mapping

```

* **Data Auditing & Quality Controls:** Loaded the independent `Purchases` dataset. Mapped operational variables (`PurchaseID`, `OrderID`, `Return Policy (Days)`, `Warranty (Months)`) to whole numbers. Verified data health via Column Quality, and filtered out active product exceptions down to strict `Not Returned` records to protect baseline statistics from return variance distortion.

### 📐 Phase 2: Relational Data Modeling & Schema Design

Built a high-performance cross-functional model using explicit schema configurations in the Model View tab to ensure seamless filter propagation:

* **`Countries` ↔ `Exchange Data`:** Established a **One-to-One (1:1)** active, bi-directional relationship mapped across the `Exchange ID` field.
* **`Sales` ↔ `Countries`:** Mapped a **Many-to-One (*:1)** active, bi-directional relationship across the foundational `Country ID` field.
* **`Purchases` ↔ `Sales`:** Interlocked transaction layers via a **One-to-One (1:1)** active, bi-directional relationship mapped on the primary identity column `OrderID`.

#### Specialized Dimension Modeling (DAX Calendar)

Constructed an independent calendar table to provide a continuous date spine, enabling the deployment of downstream time-series charts and financial tracking:

```dax
Calendar Table = 
ADDCOLUMNS(
    CALENDAR(DATE(2020, 1, 1), DATE(2023, 12, 31)),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "Quarter", QUARTER([Date]),
    "Weekday", WEEKDAY([Date]),
    "Day", DAY([Date])
)

```

* **`Calendar Table` ↔ `Purchases`:** Connected this newly engineered custom date dimension to the model via a **Many-to-One (*:1)** active relationship linking `Date` directly to the operational `Purchase Date` attribute.

### 🧮 Phase 3: Analytics Engineering & Performance Auditing (DAX)

Calculated critical operational margins and time-aggregations using specific calculated expressions:

* **Normalized Cross-Currency Matrix (`Sales in USD`):** Standardized disparate local currency inputs into a unified reporting base (USD) by building an advanced calculated table utilizing context-aware row lookups via `RELATED()`:
```dax
Sales in USD = 
ADDCOLUMNS(
    Sales,
    "Country Name", RELATED(Countries[Country]),
    "Exchange Rate", RELATED('Exchange Data'[Exchange Rate]),
    "Exchange Currency", RELATED('Exchange Data'[Exchange Currency]),
    "Gross Revenue USD", [Gross Revenue] * RELATED('Exchange Data'[Exchange Rate]),
    "Net Revenue USD", [Net Revenue] * RELATED('Exchange Data'[Exchange Rate]),
    "Total Tax USD", [Total Tax] * RELATED('Exchange Data'[Exchange Rate])
)

```


* **`Sales in USD` ↔ `Sales`:** Formed a secure **Many-to-One (*:1)** active, bi-directional connection back to the core transactional table using the primary key `OrderID`.
* **Yearly Profit Margin Measure:** Developed explicitly using a DAX ratio to analyze the proportion of total profit against net revenue within the converted dataset:
```dax
Yearly Profit Margin = DIVIDE(SUM('Sales in USD'[Total Profit]), SUM('Sales in USD'[Net Revenue USD]))

```


* **Quarterly Rolling Profit Metric:** Leveraged explicit time-intelligence functions to accumulate profitability variables up to the close of each active fiscal quarter referencing the calendar boundary:
```dax
Quarterly Profit = DATESQTD('Calendar Table'[Date])

```


* **Year-to-Date Running Profit:** Mapped running cumulative performance totals from the start of the standard operational year up to the active date constraint:
```dax
YTD Profit = TOTALYTD(SUM('Sales in USD'[Profit USD]), 'Calendar Table'[Date])

```


* **Median Sales Volatility Indicator:** Selected to establish a stable central tendency baseline unaffected by transaction anomalies or outliers:
```dax
Median Sales = MEDIAN('Sales in USD'[Gross Revenue USD])

```


* **Performance Analyzer Optimization Audit:** Deployed the **Performance Analyzer** tool inside a controlled card visual canvas to test engine scalability. Monitored backend processing metrics during simulated report refreshes to guarantee that engine query execution latency consistently registered under the **<200ms enterprise threshold**, ensuring quick direct-query visualization loads for users.

### 📊 Phase 4: Executive UI/UX Design & Dashboard Curation

Designed a dual-perspective report framework structured to meet clear corporate leadership objectives:

#### 1. Sales Overview Report

* **Loyalty Points Analysis:** Clustered bar chart measuring customer loyalty point distribution across regional operational boundaries (`Country`), sorted via axis parameters to quickly isolate top-performing territories.
* **Volume Velocity:** Clustered column chart visualizing individual quantities ordered grouped down to specific product lines to evaluate product demand.
* **Central Tendency Mapping:** Pie charts tracking the share of `Median Sales Distribution by Country` paired with a time-series line chart featuring an active **Trend Line** to evaluate temporal sales direction.
* **KPI Metrics Block:** High-level summary cards providing quick visibility into `Stock`, `Quantity Purchased`, and `Median Sales`, accompanied by a structural geographic dimension slicer (`Country Name`).

#### 2. Profit Overview Report

* **Bottom-Line Drivers:** Detailed clustered bar charts tracking specific net margins against product lines (`Net Revenue USD` by `Product Name`), sorted descending to identify immediate high-margin product groups.
* **Margin Splits:** High-density donut charts tracking `Yearly Profit Margin by Country`, outputting detailed categorical segment calculations.
* **Trended Trajectories:** Area charts monitoring profit margin changes over time to evaluate operational business health.
* **KPI Blocks & Trend Axis:** Positioned simple cards for `YTD Profit` and `Net Revenue USD`, paired with a complex KPI card mapping `Gross Revenue USD` across a specified date trend axis.

### 📱 Phase 5: Mobile Matrixing & Automated Cloud Governance

* **Mobile Responsive Engineering:** Re-architected visual placements inside Power BI's mobile workspace layout tool. Optimized screen viewports by prioritizing micro-card positions stacked symmetrically above cross-functional charts (e.g., placing Net Revenue/Quantities side-by-side above full-width Gross Revenue KPI blocks to enable mobile accessibility).
* **Automated Exception Alerting:** Deployed proactive threshold monitors on the hosted cloud service environment. Created a data alert rule on the `Sum of Gross Revenue USD` KPI tile, configuring automated system triggers to alert infrastructure administrators immediately via daily alerts if gross metrics fall below the target critical threshold ($400).
* **Automated Stakeholder Distribution:** Programmed scheduled background server subscriptions to stream dynamic snapshots of report scorecards directly to leadership teams on predetermined corporate rhythms to keep stakeholders informed:
* **Sales Weekly Summary:** Configured to push the complete Sales report page to executives every Monday morning at 5:00 AM.
* **Profit Weekly Summary:** Programmed to distribute the operational profit overview layout to financial stakeholders every Monday, Wednesday, and Friday morning at 6:00 AM.



---

## 📈 Key Operational Insights & Business Impact

* **Revenue/Profit Disconnect:** Identified specific product categories producing notable gross sales top-line volume while demonstrating high variance in net post-tax return margins due to regional adjustments.
* **International Footprint Stability:** Identified structural currency risk factors through comparative baseline trend reviews, confirming the strategic necessity of utilizing active currency conversion algorithms (`Sales in USD`) for standard international reporting metrics.

```

```
