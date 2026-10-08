# Workforce Analytics Dashboard | Power BI

A Power BI workforce analytics project that turns employee-level HR data into an interactive report for monitoring **headcount, employee turnover, absence, and workforce composition**. The dashboard is designed to help People & Culture teams identify patterns, prioritise further investigation, and support workforce planning.

> **Data note:** This is a portfolio/practice project using **240 fictional employee records** for a UK housing-association scenario. It does not contain real employee information from Thirteen Group or any other employer.

## Project objectives

- Monitor workforce size and active headcount at a defined reporting date.
- Measure employee leavers and turnover between **1 January and 30 September 2026**.
- Compare workforce metrics across departments.
- Explore absence totals and workforce gender composition.
- Communicate practical, evidence-based HR recommendations.

## Tools and skills

**Power BI Desktop** · **Power Query (M)** · **DAX** · Data validation · Data modelling · Interactive reporting · KPI analysis · HR analytics

## Dataset

The employee dataset includes:

| Field | Description |
|---|---|
| `EmployeeID` | Unique employee identifier |
| `Department` | Employee's department |
| `ContractType` | Employment contract category |
| `Gender` | Recorded gender category |
| `WorkPattern` | Full-time or part-time |
| `StartDate` | Employment start date |
| `LeavingDate` | Actual leaving date; blank for employees with no recorded departure |
| `AbsenceDays2026` | Recorded absence days in 2026 |
| `AnnualSalaryGBP` | Annual salary (£) |

## 1. Data preparation with Power Query

1. **Imported the Excel dataset** using **Get Data → Excel Workbook → Transform Data**.
2. **Inspected column quality and distribution** to review missing values, errors and category consistency.
3. **Checked data types:** employee identifiers and categories as text, start/leaving dates as dates, and absence days as whole numbers.
4. **Checked for duplicate employee IDs**, using `EmployeeID` as the unique identifier rather than treating every field as a key.
5. **Investigated missing leaving dates.** Preserved the original `LeavingDate` values: a blank leaving date is meaningful and should not automatically be treated as a data error.
6. **Created an employee status classification** for active/leaver analysis.
7. **Created a reference end-date column** (e.g., `New Leaving Date`) for tenure calculations: active employees use the reporting date, **30 September 2026**, while leavers retain their actual leaving date.
8. **Calculated tenure in years** using a custom Power Query column and rounded it to one decimal place.
9. **Applied transformations** using **Close & Apply**.

### Power Query example: tenure in years

```powerquery
Number.Round(
    Duration.Days(
        (if [LeavingDate] = null
         then #date(2026, 9, 30)
         else [LeavingDate])
        - [StartDate]
    ) / 365.25,
    1
)
```

**Important data-quality lesson:** The reference end date is suitable for tenure calculations but **not** for counting leavers. Using it for turnover would incorrectly count active employees as departures.

## 2. DAX measures

The following examples assume the Power BI table is named `'Employee Data'` and use a fixed reporting date of **30 September 2026**.

### Total employee records

```dax
Total Employees =
DISTINCTCOUNT('Employee Data'[EmployeeID])
```

### Active headcount

```dax
Active Headcount =
CALCULATE(
    DISTINCTCOUNT('Employee Data'[EmployeeID]),
    'Employee Data'[StartDate] <= DATE(2026,9,30),
    ISBLANK('Employee Data'[LeavingDate])
        || 'Employee Data'[LeavingDate] > DATE(2026,9,30)
)
```

### Leavers during the reporting period

```dax
Leavers2026 =
CALCULATE(
    DISTINCTCOUNT('Employee Data'[EmployeeID]),
    'Employee Data'[LeavingDate] >= DATE(2026,1,1),
    'Employee Data'[LeavingDate] <= DATE(2026,9,30)
)
```

### Starting headcount

```dax
Starting Head Count =
CALCULATE(
    DISTINCTCOUNT('Employee Data'[EmployeeID]),
    'Employee Data'[StartDate] <= DATE(2026,1,1),
    ISBLANK('Employee Data'[LeavingDate])
        || 'Employee Data'[LeavingDate] > DATE(2026,1,1)
)
```

### Turnover rate

```dax
Turnover Rate =
DIVIDE(
    [Leavers2026],
    ([Starting Head Count] + [Active Headcount]) / 2,
    0
)
```

Format **Turnover Rate** as **Percentage** with one decimal place. Do not also multiply by 100 in the DAX expression.

### Total absence days

```dax
Absence Days =
SUM('Employee Data'[AbsenceDays2026])
```

> The turnover formula uses the average of starting and ending headcount. The start/end boundary convention should be agreed with HR before use in production reporting.

## 3. Dashboard design

The report contains:

| Visual | Purpose |
|---|---|
| KPI cards | Total employee records, active headcount, leavers, turnover rate |
| Horizontal bar chart | Active headcount by department |
| Horizontal bar chart | Turnover rate by department |
| Bar chart | Absence days by department |
| Donut chart | Workforce distribution by gender |
| Department slicer | Interactive department-level filtering |

The dashboard uses a dark theme, clear visual titles, data labels and a defined reporting period.

### Dashboard screenshot

Add an exported image of your completed dashboard at `images/workforce-dashboard.png`, then uncomment this line:

<!-- ![Workforce Analytics Dashboard](images/workforce-dashboard.png) -->

## 4. Key results and insights

| Metric | Result |
|---|---:|
| Employee records | **240** |
| Active headcount at 30 September 2026 | **211** |
| Leavers during January–September 2026 | **29** |
| Overall turnover rate | **~12.9%** |
| Total recorded absence days | **906** |

**Findings:**

- **Repairs & Maintenance** showed the highest departmental turnover rate (approximately **22%**), followed by **Customer Experience** (approximately **20%**). These warrant further retention analysis.
- **Housing Services** had the largest active headcount (**54**) and the highest total recorded absence days (**208**). This does **not** establish that it has the highest absence *rate*, because department sizes differ.
- **IT & Digital** showed comparatively low turnover (approximately **3%**). This is worth exploring but does not, by itself, prove high employee satisfaction.

## 5. Business recommendations

1. **Investigate higher-turnover departments:** Review exit interviews, employee feedback, workload, job roles and historical trends before proposing targeted retention initiatives.
2. **Normalise absence comparisons:** Compare absence days against workforce size or, preferably, scheduled working days. Review patterns over time and consider wellbeing support where appropriate.
3. **Support workforce planning:** Use departmental headcount and turnover trends to anticipate recruitment and capacity needs, particularly in operational and customer-facing services.
4. **Explore potential retention strengths:** Investigate what may be contributing to lower turnover in IT & Digital and whether useful practices are transferable.

## 6. Limitations and next steps

- The data is **synthetic**, so findings are illustrative, not real-world organisational conclusions.
- The report uses a **fixed reporting date**. A date table and dynamic measures would enable month-by-month reporting.
- The dataset does not provide scheduled working days, so it cannot support a robust absence-rate calculation without additional information.
- Department-level differences indicate where to investigate; they do not prove causes.
- Potential extensions include **new-starter trends, vacancy rates, FTE, retention, and recruitment metrics** if the necessary data becomes available.

## Repository structure

```text
workforce-analytics-powerbi/
├── README.md
├── data/
│   └── thirteen_workforce_practice.xlsx
├── images/
│   └── workforce-dashboard.png       # Add your dashboard screenshot
└── Workforce_Analytics.pbix           # Add your saved Power BI report
```

## How to reproduce

1. Open the Excel practice dataset in Power BI Desktop.
2. Use **Transform Data** to inspect types, nulls and employee identifiers, and add tenure/status fields.
3. Select **Close & Apply**.
4. Create the DAX measures above.
5. Add KPI cards, departmental charts, gender distribution and a department slicer.
6. Validate totals and format turnover as a percentage.

---

**Project focus:** Translating workforce data into clear, actionable insights through data preparation, DAX calculations and interactive Power BI reporting.
