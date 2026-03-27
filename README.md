# Medicare Spending & Readmission Signals

**An end-to-end healthcare analytics project analyzing Medicare hospital spending across 2,871 U.S. facilities — from raw CMS data to an interactive Power BI dashboard built for finance and clinical operations stakeholders.**

---

## Dashboard Preview

### Page 1 — National Quality & Readmission Risk Analysis
![National Dashboard](visuals/Screenshot%202025-12-16%20033610.png)

> Filters: State, Hospital Owner, Rating Band, Risk Level
> KPI cards: Hospitals with Score, High MSPB Outlier %, Median MSPB Score, Avg Readmission Worse Share %, Average Gap
> Visuals: MSPB vs. Readmission scatter plot with risk-level color coding, Hospital Risk Concentration choropleth map, High Risk Hospitals by Ownership treemap

### Page 2 — Hospital-Level Drill Down
![Drill Down Dashboard](visuals/Screenshot%202025-12-16%20041050.png)

> Filters: State, Owner Type, MSPB Range slider, Readmission Worse Share slider, Risk Level
> Visuals: Hospital Performance table sorted by severity, MSPB vs. National Median gauge, Readmission vs. National Avg gauge

---

## Project Overview

Hospital leaders often lack a clear, unified view of where Medicare Spending per Beneficiary (MSPB) is elevated, which facilities are true outliers within their peer group, and whether poor quality signals — like elevated readmission rates — are driving that spending up.

This project builds that view from scratch: raw CMS public data in, actionable insight out.

> **Business Question:** How can hospital leaders identify facilities with unusually high Medicare Spending per Beneficiary and understand how spending patterns relate to hospital characteristics and quality outcomes — enabling targeted cost and quality improvement?

---

## Stakeholders

| Role | Need |
|---|---|
| CFO / Finance | Identify high-MSPB outliers and peer benchmarks to prioritize cost-reduction work |
| Quality / Clinical Ops | Pinpoint where high MSPB and poor readmission results co-exist to fix care pathways |

---

## Tools & Stack

![Excel](https://img.shields.io/badge/Excel-Advanced-217346?logo=microsoft-excel&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)
![CMS Data](https://img.shields.io/badge/Data-CMS%20Public%20Dataset-005EA2)

| Tool | Role in Project |
|---|---|
| **Excel** | Data cleaning, field mapping, EDA, IQR-based outlier detection, feature engineering, pivot analysis |
| **Power BI** | Interactive two-page dashboard: national risk overview + hospital-level drill down |
| **CMS Public Data** | Source: Medicare MSPB scores + hospital general information (2023) |

---

## Dataset

| File | Description | Rows |
|---|---|---|
| `medicare.csv` | CMS Medicare MSPB scores by facility | 4,591 |
| `hospitalGen.csv` | Hospital characteristics (type, ownership, rating, readmission measures) | 5,381 |
| `mspbProject.xlsx` | Combined working file: EDA, engineered features, pivot tables, analysis sheets | 2,871 matched facilities |

---

## Data Preparation & Field Mapping

Both source datasets were cleaned and standardized before analysis. All transformations are documented in the field mapping files included in this repo.

### Medicare Dataset (`medicare.csv`)

| Source Field | Cleaned Field | Transformation | Purpose |
|---|---|---|---|
| Facility ID | facilityID | Renamed; ensures joinability | Foreign key to hospitalGen |
| Facility Name | facilityName | Camel case | Hospital dimension attribute |
| Score | score | Decimal; removed errors | Core measure value |
| Footnote | *(dropped)* | Removed entire column | Unnecessary noise |
| Start Date / End Date | startDate / endDate | Date type; locale verified | Filtering and reporting windows |

### HospitalGen Dataset (`hospitalGen.csv`)

| Source Field | Cleaned Field | Transformation | Purpose |
|---|---|---|---|
| facilityId | facilityID | Standardized casing | Primary key |
| hospitalRating | hospitalRating | Whole number; removed errors | Quality indicator |
| zipCode | zipCode | Text format | Avoid numeric truncation |
| readmGroupMeasure* | readmGroupMeasure* | Whole number; removed errors | Readmission outcome counts |

![Medicare Field Mapping](visuals/Medicare%20Field%20Mapping.png)
![HospitalGen Field Mapping](visuals/HospitalGen%20Field%20Mapping.png)

---

## Feature Engineering

Derived fields built on top of cleaned data to power outlier detection and benchmarking:

| Feature | Description |
|---|---|
| `peer_group_key` | Groups hospitals by state + type + ownership for apples-to-apples comparison |
| `peerMedian` / `peerQ1` / `peerQ3` / `peerIQR` | Peer-level statistical benchmarks |
| `outlier_low` / `outlier_high` | IQR-based thresholds: Q1 − 1.5×IQR and Q3 + 1.5×IQR |
| `outlier_flag` | High Outlier / Low Outlier / Normal / Insufficient Data (N ≥ 8 floor enforced) |
| `gap` | Distance between facility MSPB score and its peer median |
| `readm_worse_share` | Worse readmission measures ÷ total measures (normalized for fair comparison) |
| `readm_total` | Sum of all readmission measure categories |
| `rating_band` | Low / Mid / High grouping derived from CMS star ratings |

---

## EDA Summary

| Metric | Value |
|---|---|
| Unique facilities | 2,871 |
| Unique states | 54 |
| Total columns (finalData) | 17 |
| Min MSPB score | 0.68 |
| Max MSPB score | 1.34 |
| Mean MSPB score | 0.995 |
| Median MSPB score | 0.990 |
| Std deviation | 0.074 |
| Score range | 0.66 |

The MSPB distribution is approximately normal with a slight right skew — most facilities cluster near the national median of 0.990, with a long tail of high-cost outliers.

![MSPB Distribution](visuals/Healthcare%20-%20Avg%20MSPB%20score%20by%20Num%20of%20facilities.png)
![EDA Summary](visuals/Healthcare%20-%20EDA.png)

---

## Key Findings

### Outlier Detection
- **36 high outliers flagged** (1.8% of total facilities)
- Top states by outlier count: **FL (7), CA (6), PA (4), TX (3), OH (3), NJ (3)**
- Top states by outlier share %: **NE (5.26%), NJ (5.00%), LA (4.55%), FL (4.49%)**

![High Outliers by State](visuals/Healthcare%20-%20High%20outliers%20by%20state.png)
![Outlier Share by State](visuals/Healthcare%20-%20Outlier%20%25%20share%20by%20state.png)

### Quality-Cost Relationship

| Rating Band | Median MSPB | High Outlier % | Mean Readm-Worse Share |
|---|---|---|---|
| High (4–5 stars) | 0.980 | 0.91% | 4.20% |
| Mid (3 stars) | 0.990 | 1.23% | 6.72% |
| Low (1–2 stars) | 1.010 | 2.25% | 13.31% |

- Pearson r between MSPB and readmission-worse share: **r = 0.22** — directional relationship confirmed

![Rating Band Analysis](visuals/Healthcare%20-%20Distribution%20of%20MSPB%20to%20Hospital%20rating.png)
![MSPB vs Outlier Flag](visuals/Healthcare%20-%20Median%20MSPB%2C%20High%20Outlier%25%20share%2C%20and%20Readmisson%20worse%20share%20%25%20by%20Outlier%20flag.png)

### Geographic Patterns
- **Highest state medians:** NJ (1.065), LA (1.060), NV (1.060), AR (1.040), TX (1.040)
- **Lowest state medians:** OR, AK, MT, MN — significant geographic variation confirmed

![Median MSPB by State Chart](visuals/Healthcare%20-%20Median%20MSPB%20by%20state%20chart.png)
![Median MSPB by State Table](visuals/Healthcare%20-%20Median%20MSPB%20by%20state.png)

### Ownership Type
- **Highest median MSPB:** Tribal (1.03), Proprietary (1.02), Government-State (1.02)
- **Lowest median MSPB:** Government-Federal (0.96), Voluntary non-profit Other (0.975)

![Median MSPB by Ownership](visuals/Healthcare%20-%20Median%20MSPB%20score%20by%20Ownership.png)
![Ownership Distribution](visuals/Healthcare%20-%20Distribution%20of%20ownership%20type.png)

### Risk Classification (Power BI)
The dashboard introduces a four-tier **Risk Level** classification combining MSPB and readmission signals:

| Risk Level | Definition |
|---|---|
| 🔴 Extreme Risk | High MSPB outlier + elevated readmission worse share |
| 🟠 High Cost Only | High MSPB outlier; readmissions within normal range |
| 🟡 High Readmission Only | Readmissions elevated; MSPB within normal range |
| 🟢 Normal | Both metrics within expected range |

- **Extreme Risk cohort (12 facilities):** Median MSPB 1.200, avg readmission-worse share 29.35%, average gap 0.199
- **Physician-owned hospitals:** 25% high-risk share — highest of any ownership type
- **Proprietary hospitals:** 22.25% high-risk share

![Extreme Risk Filter View](visuals/Screenshot%202025-12-16%20041437.png)
![Ownership Treemap Detail](visuals/Screenshot%202025-12-16%20041509.png)

---

## Recommendations

1. **Run targeted reviews** for Tribal, Proprietary, and Government-State ownership types — structural cost drivers likely differ from nonprofit peers.
2. **Deploy an alert rule** — flag any facility exceeding its peer MSPB threshold with a large gap and elevated readmission-worse share for automatic review.
3. **Prioritize by count and severity together** — FL, CA, PA have the most outliers; NE, NJ, LA have the highest outlier share rates. Intervention strategies should differ by state.
4. **Focus Extreme Risk hospitals first** — 12 facilities show all three warning signals simultaneously (high cost, high readmissions, high outlier flag).
5. **Respect the N ≥ 8 floor** — peer groups with fewer than 8 facilities are flagged as Insufficient Data; do not act on small-sample comparisons.

---

## Repository Structure

```
medicare-spending-readmission-signals/
│
├── data/
│   ├── medicare.csv                      # CMS Medicare MSPB scores (2023)
│   └── hospitalGen.csv                   # Hospital characteristics
│
├── analysis/
│   └── mspbProject.xlsx                  # Full working file: EDA, features, pivots, analysis
│
├── visuals/
│   ├── Healthcare_-_EDA.png
│   ├── Healthcare_-_Avg_MSPB_score_by_Num_of_facilities.png
│   ├── Healthcare_-_Distribution_of_MSPB_to_Hospital_rating.png
│   ├── Healthcare_-_Distribution_of_ownership_type.png
│   ├── Healthcare_-_High_outliers_by_state.png
│   ├── Healthcare_-_Median_MSPB_by_state.png
│   ├── Healthcare_-_Median_MSPB_by_state_chart.png
│   ├── Healthcare_-_Median_MSPB_score_by_Ownership.png
│   ├── Healthcare_-_Median_MSPB__High_Outlier__share__and_Readmisson_worse_share___by_Outlier_flag.png
│   ├── Healthcare_-_Outlier___share_by_state.png
│   ├── Medicare_Field_Mapping.png
│   ├── HospitalGen_Field_Mapping.png
│   ├── Screenshot_2025-12-16_033610.png  # Power BI Page 1 — National Overview
│   ├── Screenshot_2025-12-16_041050.png  # Power BI Page 2 — Hospital Drill Down
│   ├── Screenshot_2025-12-16_041437.png  # Extreme Risk filter view
│   └── Screenshot_2025-12-16_041509.png  # Ownership treemap detail
│
├── docs/
│   ├── KPI_Data_Definitions.docx         # Field-level definitions for all 21 KPIs
│   ├── Project_Scenario.docx             # Problem statement, stakeholders, sub-questions
│   └── Insights_Recommendations.docx    # Final findings and action items
│
└── README.md
```

---

## How to Explore This Project

1. **Start with the docs** — `Project_Scenario.docx` frames the business problem; `KPI_Data_Definitions.docx` defines every field
2. **Review the field mappings** — `visuals/Medicare_Field_Mapping.png` and `HospitalGen_Field_Mapping.png` show the full cleaning and transformation logic
3. **Open the Excel workbook** — `analysis/mspbProject.xlsx`; begin on the **EDA** sheet, then follow tabs in order through to `finalData`
4. **Explore the dashboard screenshots** — Pages 1 and 2 in the `visuals/` folder show the full interactive Power BI output
5. **Read the findings** — `docs/Insights_Recommendations.docx` for the final summary

---

## About

Built as a portfolio project demonstrating end-to-end healthcare analytics: data sourcing, cleaning, field mapping, peer benchmarking, IQR-based outlier detection, feature engineering, and stakeholder-ready Power BI dashboard design — using publicly available CMS data.

**Data source:** [CMS Care Compare](https://www.cms.gov/medicare/quality/initiatives/hospital-quality-initiative/hospital-compare) — Medicare Hospital Spending per Patient (MSPB-1), 2023.
