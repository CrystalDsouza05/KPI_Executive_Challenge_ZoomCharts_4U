# 📊 Nexoria Group — Executive KPI Dashboard
### ZoomCharts 4U Data Challenge · April Edition

A board-ready Executive KPI Report built in Power BI for a fictional global enterprise, Nexoria Group — covering 24 months of Financial, People, Customer and Operational KPIs across 3 divisions and 34 countries.


## 🖼️ Dashboard Preview

| Page | Preview |
|---|---|
| Executive Summary | Page 1 |
| Divisional & Operational Deep-Dive | Page 2 |
| Regional & Customer Intelligence | Page 3 |
| Division Drill-Through | Drill-through A |
| Country Drill-Through | Drill-through B |


Click Here to View [Dashboard](https://app.powerbi.com/view?r=eyJrIjoiYzY0NTdlNzAtMjc5NS00ZTUzLWJhYmItMGMxNmNmYWQ1NjUxIiwidCI6IjQ2NTRiNmYxLTBlNDctNDU3OS1hOGExLTAyZmU5ZDk0M2M3YiIsImMiOjl9)

<img width="853" height="480" alt="Crystal_Executive_KPI_Dashboard" src="https://github.com/user-attachments/assets/2fac259e-7038-4273-b62f-a5762e50f04d" />


## 🏆 Challenge Brief

Built for the **ZoomCharts 4U Data Challenge — April Edition**. The objective was to design a clean, executive-focused Power BI report for Nexoria Group that communicates high-level performance while allowing stakeholders to explore drivers through hierarchy-based drill-down paths.


## 📁 Dataset Overview

The dataset is a **star schema** with one fact table and four dimension tables spanning **April 2024 – March 2026** (24 months).

| Table | Rows | Description |
|---|---|---|
| `FactKPI_Monthly` | 440,640 | Unpivoted KPI values by month, org, region and scenario |
| `DimKPI` | 10 | KPI metadata — name, category, unit, polarity |
| `DimDate` | 24 | Month-grain date table with fiscal calendar |
| `DimOrg` | 18 | Org hierarchy — Division → BU → Product Line |
| `DimRegion` | 34 | Region hierarchy — Region → Subregion → Country |

**10 KPIs across 4 categories:**

| KPIKey | KPI Name | Category | Aggregation | Polarity |
|---|---|---|---|---|
| 10 | Revenue | Financial | SUM | Higher = Better |
| 20 | EBITDA | Financial | SUM | Higher = Better |
| 30 | Headcount | People | SUM + ROUND | Higher = Better |
| 40 | Net Customer Adds | Customer | SUM + ROUND | Higher = Better |
| 50 | Active Customers | Customer | SUM + ROUND | Higher = Better |
| 60 | On-time Delivery % | Operations | AVERAGE | Higher = Better |
| 70 | System Uptime % | Operations | AVERAGE | Higher = Better |
| 80 | Support Tickets | Operations | SUM + ROUND | **Lower = Better** |
| 90 | Cycle Time (days) | Operations | AVERAGE | **Lower = Better** |
| 100 | Gross Margin % | Financial | AVERAGE | Higher = Better |

> **Note:** % KPIs (Gross Margin, On-time Delivery, System Uptime) are stored as 0–1 ratios in the fact table. All measures use `AVERAGE` not `SUM` for these KPIs.


## 🗂️ Data Model

Star schema with 4 one-to-many relationships from dimension tables into `FactKPI_Monthly`:

```
DimDate      ──(1:*)──┐
DimOrg       ──(1:*)──┤
                       FactKPI_Monthly
DimRegion    ──(1:*)──┤
DimKPI       ──(1:*)──┘
```

**Relationship settings:** Cardinality = Many-to-one · Cross-filter = Single

**Hierarchies created:**
- Org Hierarchy: `Division → BusinessUnit → ProductLine`
- Region Hierarchy: `Region → Subregion → Country`
- Date Hierarchy: `Year → Quarter → MonthName`


## 📄 Report Pages

### 📊 Page 1 — Executive Summary
The board-ready overview — Revenue, EBITDA, Margins, Headcount and Customer KPIs all in one place, with a ZoomCharts drill-down line chart tracking Revenue Actual vs Budget across 24 months and a regional snapshot matrix that tells the full story at a glance.

### ⚙️ Page 2 — Divisional & Operational Deep-Dive
Where things get interesting — Headcount and EBITDA broken down by Division using ZoomCharts drill-down bars, paired with Operational KPIs like On-time Delivery, System Uptime, Support Tickets and Cycle Time, plus a full Division × KPI scorecard matrix.

### 🌍 Page 3 — Regional & Customer Intelligence
A worldwide view of customer growth, net adds and revenue performance by region — complete with a map visual, a drill-down bar by region and a full regional scorecard matrix with country-level detail and flag images.

### 🔍 Drill-through A — Division Detail
Right-click any division and land here — a month-by-month EBITDA breakdown by Business Unit, Gross Margin and System Uptime gauges, and a Revenue YTD trend with YoY comparison.

### 📍 Drill-through B — Country Detail
My personal favourite — right-click any country and get a full country-level performance report complete with the country flag, Revenue YTD trend, Gross Margin, System Uptime, On-time Delivery gauges and a granular scorecard by Executive Owner and Product Line.


## 🧮 DAX Measures Overview

All calculations are driven using DAX measures, organised into logical display folders for maintainability (click to expand).

---

<details>
<summary><strong>💰 Revenue KPIs (11 measures)</strong></summary>

```dax
Revenue Actual =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 10
)

Revenue Budget =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 10
)

Revenue Forecast =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Forecast",
    DimKPI[KPIKey] = 10
)

Revenue vs Budget % =
DIVIDE([Revenue Actual] - [Revenue Budget], [Revenue Budget])

Revenue vs Budget $ =
[Revenue Actual] - [Revenue Budget]

Revenue YTD Actual =
CALCULATE(
    [Revenue Actual],
    DATESYTD(DimDate[Fiscal Month Start Date])
)

Revenue YTD Budget =
CALCULATE(
    [Revenue Budget],
    DATESYTD(DimDate[Fiscal Month Start Date])
)

Revenue YTD vs Budget % =
DIVIDE(
    [Revenue YTD Actual] - [Revenue YTD Budget],
    [Revenue YTD Budget]
)

Revenue PY =
CALCULATE(
    [Revenue Actual],
    SAMEPERIODLASTYEAR(DimDate[Fiscal Month Start Date])
)

Revenue YoY % =
DIVIDE(
    [Revenue Actual] - [Revenue PY],
    [Revenue PY]
)

Revenue Board Season =
CALCULATE(
    [Revenue Actual],
    DimDate[IsBoardSeason] = 1
)
```

</details>

---

<details>
<summary><strong>📈 EBITDA KPIs (10 measures)</strong></summary>

```dax
EBITDA Actual =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 20
)

EBITDA Budget =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 20
)

EBITDA Forecast =
CALCULATE(
    SUM(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Forecast",
    DimKPI[KPIKey] = 20
)

EBITDA vs Budget $ =
[EBITDA Actual] - [EBITDA Budget]

EBITDA vs Budget % =
DIVIDE([EBITDA Actual] - [EBITDA Budget], [EBITDA Budget])

EBITDA Margin Actual =
DIVIDE([EBITDA Actual], [Revenue Actual])

EBITDA Margin Budget =
DIVIDE([EBITDA Budget], [Revenue Budget])

EBITDA Margin vs Budget (pp) =
([EBITDA Margin Actual] - [EBITDA Margin Budget]) * 100

EBITDA PY =
CALCULATE(
    [EBITDA Actual],
    SAMEPERIODLASTYEAR(DimDate[MonthStartDate])
)

EBITDA YoY % =
DIVIDE(
    [EBITDA Actual] - [EBITDA PY],
    [EBITDA PY]
)
```

</details>

---

<details>
<summary><strong>📉 Gross Margin % (5 measures)</strong></summary>

```dax
-- Stored as 0-1 ratio in fact table — use AVERAGE not SUM

Gross Margin % Actual =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 100
)

Gross Margin % Budget =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 100
)

Gross Margin vs Budget (pp) =
([Gross Margin % Actual] - [Gross Margin % Budget]) * 100

-- Donut chart complement measure (remaining %)
Actual Gm =
1 - [Gross Margin % Actual]

-- Donut chart label
Donut GM =
"v/s Budget (pp) " & FORMAT([Gross Margin vs Budget (pp)], "+0.00;-0.00;0.00")
```

</details>

---

<details>
<summary><strong>👥 People KPIs (3 measures)</strong></summary>

```dax
-- Values are fractional in source — always wrap in ROUND()

Headcount Actual =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Actual",
        DimKPI[KPIKey] = 30
    ),
    0
)

Headcount Budget =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Budget",
        DimKPI[KPIKey] = 30
    ),
    0
)

Headcount vs Budget =
[Headcount Actual] - [Headcount Budget]
```

</details>

---

<details>
<summary><strong>🤝 Customer KPIs (10 measures)</strong></summary>

```dax
Active Customers Actual =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Actual",
        DimKPI[KPIKey] = 50
    ),
    0
)

Active Customers Budget =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Budget",
        DimKPI[KPIKey] = 50
    ),
    0
)

Net Customer Adds Actual =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Actual",
        DimKPI[KPIKey] = 40
    ),
    0
)

Net Customer Adds Budget =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Budget",
        DimKPI[KPIKey] = 40
    ),
    0
)

Net Customer Adds vs Budget =
[Net Customer Adds Actual] - [Net Customer Adds Budget]

Active Customers PY =
CALCULATE(
    [Active Customers Actual],
    SAMEPERIODLASTYEAR(DimDate[MonthStartDate])
)

Customer Growth YoY % =
DIVIDE(
    [Active Customers Actual] - [Active Customers PY],
    [Active Customers PY]
)

-- Uses ISINSCOPE to handle both year and month level correctly
Net Customer Adds PY =
IF(
    ISINSCOPE(DimDate[MonthStartDate]),
    CALCULATE(
        [Net Customer Adds Actual],
        FILTER(
            ALL(DimDate),
            DimDate[Year] = MAX(DimDate[Year]) - 1
                && DimDate[MonthNumber] = MAX(DimDate[MonthNumber])
        )
    ),
    CALCULATE(
        [Net Customer Adds Actual],
        FILTER(
            ALL(DimDate),
            DimDate[Year] = MAX(DimDate[Year]) - 1
        )
    )
)

Net Customer Adds YoY % =
DIVIDE(
    [Net Customer Adds Actual] - [Net Customer Adds PY],
    [Net Customer Adds PY]
)

-- Display measure combining Actual and Budget for matrix visuals
Cust Active =
FORMAT([Active Customers Actual], "#,##0")
& " | " &
FORMAT([Active Customers Budget], "#,##0")
```

</details>

---

<details>
<summary><strong>🚚 Delivery KPIs (5 measures)</strong></summary>

```dax
-- Stored as 0-1 ratio — use AVERAGE not SUM

On-time Delivery Actual =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 60
)

On-time Delivery Budget =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 60
)

On-time Delivery vs Budget (pp) =
([On-time Delivery Actual] - [On-time Delivery Budget]) * 100

-- Donut chart complement measure
Act Del =
1 - [On-time Delivery Actual]

-- Donut chart label
Donut Del =
"v/s Budget (pp) " & FORMAT([On-time Delivery vs Budget (pp)], "+0.00;-0.00;0.00")
```

</details>

---

<details>
<summary><strong>🖥️ System KPIs (5 measures)</strong></summary>

```dax
-- Stored as 0-1 ratio — use AVERAGE not SUM

System Uptime Actual =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 70
)

System Uptime Budget =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 70
)

-- Donut chart complement measure
Act System =
1 - [System Uptime Actual]

System Uptime v/s Bud (pp) =
[System Uptime Actual] - [System Uptime Budget]

-- Donut chart label
Donut System =
"v/s Budget (pp) " & FORMAT([System Uptime v/s Bud (pp)], "+0.00;-0.00;0.00")
```

</details>

---

<details>
<summary><strong>🎫 Support Tickets (3 measures)</strong></summary>

```dax
Support Tickets Actual =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Actual",
        DimKPI[KPIKey] = 80
    ),
    0
)

Support Tickets Budget =
ROUND(
    CALCULATE(
        SUM(FactKPI_Monthly[Value]),
        FactKPI_Monthly[Scenario] = "Budget",
        DimKPI[KPIKey] = 80
    ),
    0
)

-- Negative value = GOOD (fewer tickets than budgeted)
Support Tickets vs Budget =
[Support Tickets Actual] - [Support Tickets Budget]
```

</details>

---

<details>
<summary><strong>⏱️ Cycle Time KPIs (3 measures)</strong></summary>

```dax
-- Use AVERAGE — cycle time is a rate, not a sum

Cycle Time Actual =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Actual",
    DimKPI[KPIKey] = 90
)

Cycle Time Budget =
CALCULATE(
    AVERAGE(FactKPI_Monthly[Value]),
    FactKPI_Monthly[Scenario] = "Budget",
    DimKPI[KPIKey] = 90
)

-- Negative value = GOOD (faster than budgeted)
Cycle Time vs Budget (days) =
[Cycle Time Actual] - [Cycle Time Budget]
```

</details>

---

<details>
<summary><strong>🏷️ Display & Formatting Measures (3 measures)</strong></summary>

```dax
-- Dynamic short format: auto-switches between Bn / M / K
Revenue Actual (Short) =
VAR val = [Revenue Actual]
RETURN
SWITCH(
    TRUE(),
    ABS(val) >= 1000000000, FORMAT(val / 1000000000, "0.0") & " Bn",
    ABS(val) >= 1000000,    FORMAT(val / 1000000, "0.0") & " M",
    ABS(val) >= 1000,       FORMAT(val / 1000, "0.0") & " K",
    FORMAT(val, "0")
)

EBITDA Actual (Short) =
VAR val = [EBITDA Actual]
RETURN
SWITCH(
    TRUE(),
    ABS(val) >= 1000000000, FORMAT(val / 1000000000, "0.0") & " Bn",
    ABS(val) >= 1000000,    FORMAT(val / 1000000, "0.0") & " M",
    ABS(val) >= 1000,       FORMAT(val / 1000, "0.0") & " K",
    FORMAT(val, "0")
)

EBITDA Budget (Short) =
VAR val = [EBITDA Budget]
RETURN
SWITCH(
    TRUE(),
    ABS(val) >= 1000000000, FORMAT(val / 1000000000, "0.0") & " Bn",
    ABS(val) >= 1000000,    FORMAT(val / 1000000, "0.0") & " M",
    ABS(val) >= 1000,       FORMAT(val / 1000, "0.0") & " K",
    FORMAT(val, "0")
)
```

</details>

---

<details>
<summary><strong>🔧 Utility Measures (3 measures)</strong></summary>

```dax
-- Returns 1 if data exists in context — used for conditional visibility
Dates Available =
IF(COUNTROWS(FactKPI_Monthly) > 0, 1, 0)

-- Static label for donut chart centre
_Donut KPI Label =
"Actual Headcount"

-- Blanks out current month to avoid partial-month distortion on trend lines
Revenue Actual (Normal) =
IF(
    SELECTEDVALUE('DimDate'[FYmonthName]) <> MAX('DimDate'[FYmonthName]),
    [Revenue Actual]
)
```

</details>

---

<details>
<summary><strong>✏️ Dynamic Titles & Subtitles (10 measures)</strong></summary>

```dax
-- Page 1 Title
_P1Title =
"Nexoria Group - " & SELECTEDVALUE(DimDate[Year],"All Years") & " Performance Dashboard"

-- Page 1 Subtitle — shows revenue variance direction and selected region
_P1Subtitle =
VAR _rvs = [Revenue vs Budget %]
VAR _dir = IF(_rvs >= 0, "ahead of", "behind")
VAR _pct = FORMAT(ABS(_rvs), "⮝0.0%;⮟0.0%;0.0%")
VAR _rgn = SELECTEDVALUE(DimRegion[Region], "All Regions")
RETURN "Revenue " & _pct & " " & _dir & " budget · " & _rgn
-- Preview → "Revenue ⮝1.2% ahead of budget · AMER"

-- Page 2 Title
_P2Title =
SELECTEDVALUE(DimOrg[Division], "All Divisions") & " - Operational Performance"

-- Page 2 Subtitle — shows OTD and headcount variance
_P2Subtitle =
VAR _hcv = [Headcount vs Budget]
VAR _otd = FORMAT([On-time Delivery Actual], "0.0%")
VAR _hctxt = IF(_hcv > 0, FORMAT(_hcv,"#,0") & " over headcount budget", FORMAT(ABS(_hcv),"#,0") & " under headcount budget")
RETURN "On-time delivery at " & _otd & " · " & _hctxt
-- Preview → "All Divisions - On-time delivery at 91.0% · 10,577 over headcount budget"

-- Page 3 Title
_P3Title =
SELECTEDVALUE(DimRegion[Region], "Global") & " - Customer & Revenue Intelligence"

-- Page 3 Subtitle — shows YoY customer growth and net adds vs budget
_P3Subtitle =
VAR _growth = FORMAT([Customer Growth YoY %], "+0.0%;-0.0%")
VAR _ncav = [Net Customer Adds vs Budget]
VAR _nctxt = IF(_ncav >= 0, FORMAT(_ncav,"#,0") & " net adds above budget", FORMAT(ABS(_ncav),"#,0") & " net adds below budget")
RETURN "Customer growth YoY " & _growth & " · " & _nctxt
-- Preview → "Global - Customer growth YoY +103.1% · 2,289 net adds above budget"

-- Drill-through A Title
_D1Title =
SELECTEDVALUE(DimOrg[Division]) & " Division - Full KPI Breakdown | FY: " & SELECTEDVALUE(DimDate[FY])

-- Drill-through A Subtitle
_D1Subtitle =
VAR _rev = FORMAT([Revenue Actual]/1000000, "$#,0.0M")
VAR _margin = FORMAT([EBITDA Margin Actual], "0.0%")
RETURN "Revenue " & _rev & " · EBITDA margin " & _margin
-- Preview → "Technology Division — Revenue $3,984.0M · EBITDA margin 33.2%"

-- Drill-through B Title
_D2Title =
SELECTEDVALUE(DimRegion[Country]) & " - Country Performance Report | FY: " & SELECTEDVALUE(DimDate[FY])

-- Drill-through B Subtitle
_D2Subtitle =
VAR _tier = SELECTEDVALUE(DimRegion[MarketTier])
VAR _growth = FORMAT([Customer Growth YoY %], "+0.0%;-0.0%")
RETURN _tier & " market · Customer growth " & _growth
-- Preview → "Sweden — Core market · Customer growth +3.79%"
```

</details>


## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Power BI Desktop | Report development |
| DAX | All calculations and dynamic titles |
| ZoomCharts Drill Down PRO | Hierarchy drill-down bar and line charts |
| Power Query (M) | Data transformation and type corrections |
| Figma and PowerPoint | Dashboard Design |
| flagcdn.com | Country flag images via URL |

---

## 📌 Key DAX Notes

- **% KPIs stored as 0–1 ratios** — `Gross Margin %`, `On-time Delivery %` and `System Uptime %` use `AVERAGE` not `SUM`
- **Headcount is fractional** — all headcount measures wrapped in `ROUND(..., 0)`
- **`ISINSCOPE` pattern** — used for `Net Customer Adds PY` to handle both year-level and month-level context correctly without a full calendar table
- **All 3 scenarios span all 24 months** — always filter by both `Scenario` AND `KPIKey` in every measure
- **Dynamic format strings** — variance measures use `"⮝0.00%;⮟0.00%;0.00%"` with UNICHAR(11165) and UNICHAR(11167) for directional arrows
- **All values in USD equivalent** — the `Currency` column in `DimRegion` is metadata only; all fact values are pre-normalised to a single base currency

---

*Built with 💙 for the ZoomCharts 4U Data Challenge — April Edition*
