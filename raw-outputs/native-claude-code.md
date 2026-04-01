---
layout: page
title: "Native Claude Code Output — Full Data Lineage"
permalink: /raw-outputs/native-claude-code/
description: "Raw output from native Claude Code (browser) documenting the RPF Country Dashboard data lineage."
---

# Data Sources Documentation

This document provides a chart-by-chart summary of the data sources used in the RPF Country Dashboard, how the data is processed, and where it originates.

## Architecture Overview

The dashboard relies on two upstream data pipelines that feed into a Databricks data warehouse:

| Repository | Role | Output Schema |
|---|---|---|
| **mega-boost** | Harmonizes BOOST public finance microdata from 18 countries into standardized expenditure tables | `prd_mega.boost.*` |
| **mega-indicators** | Collects and transforms global development indicators from external APIs and datasets | `prd_mega.indicator.*` |
| **rpf-country-dash** | Plotly Dash application that queries the above tables and renders interactive visualizations | N/A (consumer) |

### Data Flow

```
Country Finance Ministries --> mega-boost (Excel -> Bronze -> Silver -> Gold) --> prd_mega.boost.*
                                                                                      |
World Bank API, WHO, GDL,  --> mega-indicators (API -> Transform -> Gold)  --> prd_mega.indicator.*
UNESCO, Ember, PEFA                                                                   |
                                                                                      v
                                                                         rpf-country-dash (Dash app)
                                                                         queries via Databricks SQL
```

---

## Home Page

### "Over Time" Tab

#### 1. Total Expenditure Over Time
- **Chart type:** Stacked bar (domestic/foreign funded) + line (real expenditure)
- **Databricks table:** `prd_mega.boost.pov_expenditure_by_country_year`
- **Key columns:** `year`, `expenditure`, `real_expenditure`, `foreign_funded_expenditure`, `domestic_funded_expenditure`
- **Source (mega-boost):** Aggregated from `boost_gold` by country and year. `real_expenditure` is inflation-adjusted using CPI from `prd_mega.indicator.consumer_price_index`. Foreign/domestic split comes from the `is_foreign` flag in the harmonized microdata.
- **Original data:** BOOST CCI Excel files from country finance ministries.

#### 2. Per Capita Expenditure Over Time
- **Chart type:** Bar (per capita expenditure) + line (poverty rate) on secondary axis
- **Databricks table:** `prd_mega.boost.pov_expenditure_by_country_year`
- **Key columns:** `year`, `per_capita_expenditure`, `poverty_rate`
- **Source (mega-boost):** Per capita calculated by dividing total expenditure by subnational population from `prd_mega.indicator.subnational_population`.
- **Source (mega-indicators):** Poverty rate from World Bank Poverty & Inequality Platform (PIP), stored in `prd_mega.indicator.poverty_rate` and joined during the BOOST aggregation step.
- **Original data:** BOOST microdata (expenditure), World Bank PIP (poverty at $2.15/day threshold).

#### 3. Functional Breakdown Over Time
- **Chart type:** Stacked bar chart
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year`
- **Key columns:** `year`, `func`, `expenditure`, `real_expenditure`
- **Aggregation:** Dashboard groups by (country, year, func) and sums expenditure.
- **Source (mega-boost):** COFOG-mapped functional classifications from country-specific microdata. Each country's raw functional codes are mapped to standard COFOG categories (General public services, Defence, Education, Health, Social protection, etc.) during the Silver transformation stage.
- **Original data:** BOOST CCI Excel files; functional classification codes mapped via `tag_code_mapping.csv` (700+ codes).

#### 4. Budget Growth by Functional Category
- **Chart type:** Multi-line chart
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year`
- **Key columns:** `year`, `func`, `real_domestic_funded_budget`
- **Dashboard processing:** Computes `real_domestic_funded_budget = (real_expenditure / expenditure) * domestic_funded_budget`. Calculates year-on-year growth rates and CAGR over 5-year windows. Filters to domestic-funded budget only.
- **Source (mega-boost):** Same as Functional Breakdown. The `domestic_funded_budget` column comes from the BOOST aggregation, while `real_expenditure` is CPI-adjusted.
- **Original data:** BOOST CCI files (approved/revised budget amounts).

#### 5. Economic Breakdown Over Time
- **Chart type:** Stacked area/line chart
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year`
- **Key columns:** `year`, `econ`, `expenditure`, `real_expenditure`
- **Aggregation:** Dashboard groups by (country, year, econ) and sums expenditure.
- **Source (mega-boost):** Economic classification mapped from country-specific codes to standard categories (Wage bill, Capital expenditure, Goods & services, Subsidies, Social benefits, Interest on debt, Other expenses) during Silver transformation.
- **Original data:** BOOST CCI Excel files.

#### 6. PEFA Overall Score Over Time
- **Chart type:** Line (overall PEFA score) + scatter (poverty rate) on secondary axis
- **Databricks table:** `prd_mega.indicator.pefa_by_pillar`
- **Key columns:** `year`, `pillar1_budget_reliability` through `pillar7_external_audit`
- **Dashboard processing:** Letter grades (A, B+, B, C+, C, D+, D) are mapped to numeric scores (A=4, D=1). Overall score is the mean across all 7 pillars. Poverty rate overlay comes from the main expenditure dataset.
- **Source (mega-indicators):** PEFA assessment data, manually uploaded to an intermediate table, then transformed via DLT. Covers both 2011 and 2016 PEFA frameworks.
- **Original data:** PEFA Secretariat assessments (https://www.pefa.org).

#### 7. PEFA by Pillar Over Time
- **Chart type:** Heatmap
- **Databricks table:** `prd_mega.indicator.pefa_by_pillar`
- **Key columns:** `year`, 7 pillar columns (budget reliability, transparency, asset/liability management, policy-based strategy, predictability/control, accounting/reporting, external audit)
- **Dashboard processing:** Same letter-to-numeric mapping as above. Displayed as a color-coded heatmap across pillars and assessment years.
- **Source:** Same as PEFA Overall Score.

### "Across Space" Tab

#### 8. Subnational Spending Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_by_country_geo1_year` (spending by region)
  - `prd_mega.indicator.admin1_boundaries_gold` (GeoJSON boundaries)
  - `prd_mega.indicator.admin0_disputed_boundaries_gold` (disputed area overlays)
- **Key columns:** `adm1_name`, `year`, `expenditure`, `per_capita_expenditure`
- **Source (mega-boost):** Subnational expenditure aggregated by `geo1` (admin-1 region). Region names are harmonized during Silver transformation to match boundary data.
- **Source (mega-indicators):** Admin-1 boundaries from World Bank official GeoJSON files, with region name corrections for 14+ countries and polygon unions for harmonized regions.
- **Original data:** BOOST microdata (expenditure by region), World Bank Geospatial Operations (boundaries).

#### 9. Subnational Poverty Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.indicator.subnational_poverty_rate` (poverty by region)
  - `prd_mega.indicator.admin1_boundaries_gold` (GeoJSON boundaries)
  - `prd_mega.indicator.admin0_disputed_boundaries_gold` (disputed area overlays)
- **Key columns:** `adm1_name`, `year`, `poverty_rate`
- **Source (mega-indicators):** Subnational poverty rates from SPID-GSAP (World Bank), with extensive region name harmonization for 18 countries.
- **Original data:** World Bank SPID/GSAP surveys.

---

## Education Page

### "Over Time" Tab

#### 10. Public vs. Private Education Spending
- **Chart type:** Bar/mixed comparison
- **Databricks tables:**
  - `prd_mega.boost.expenditure_by_country_func_econ_year` (public spending, filtered to func="Education")
  - `prd_mega.boost.edu_private_expenditure_by_country_year` (private spending)
- **Key columns:** `year`, `real_expenditure` (public), `real_expenditure` (private/ICP)
- **Source (mega-boost):** Public education spending is the BOOST expenditure filtered to the "Education" COFOG function.
- **Source (mega-indicators/mega-boost):** Private education expenditure sourced from the International Comparison Program (ICP) via World Bank API, stored in the BOOST schema.
- **Original data:** BOOST CCI (public), World Bank ICP (private).

#### 11. Total Education Expenditure Over Time
- **Chart type:** Stacked bar (domestic/foreign) + line (real expenditure)
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year` (filtered to func="Education")
- **Key columns:** `year`, `expenditure`, `real_expenditure`, `foreign_funded_expenditure`, `domestic_funded_expenditure`
- **Source:** Same as Total Expenditure (chart 1) but filtered to the Education functional category.
- **Original data:** BOOST CCI files.

#### 12. Education Spending and Outcomes
- **Chart type:** Scatter + line (spending vs. learning poverty or school attendance)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_by_country_func_econ_year` (education spending)
  - `prd_mega.indicator.learning_poverty_rate` (learning poverty)
- **Key columns:** `year`, `real_expenditure` (education), `learning_poverty_rate`
- **Source (mega-indicators):** Learning poverty rate from World Bank Education Statistics database. School attendance data from Global Data Lab (GDL).
- **Original data:** World Bank EdStats (learning poverty), GDL surveys (attendance).

#### 13. Operational vs. Capital Spending in Education
- **Chart type:** Stacked area chart
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year` (filtered to func="Education")
- **Key columns:** `year`, `econ`, `real_expenditure`
- **Dashboard processing:** Economic categories are remapped: "Wage bill" stays as-is, "Capital expenditures" stays as-is, all others become "Non-wage recurrent". Proportions are calculated as (category expenditure / total education expenditure) * 100.
- **Source (mega-boost):** BOOST economic classification within education functional category.
- **Original data:** BOOST CCI files.

### "Across Space" Tab

#### 14. Central vs. Regional Education Spending (Pie Chart)
- **Chart type:** Pie/Donut chart
- **Databricks table:** `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Education")
- **Key columns:** `geo0` (Central/Regional), `expenditure`, `real_expenditure`
- **Source (mega-boost):** The `geo0` field distinguishes centrally-allocated vs. regionally-allocated spending. Derived from the `admin0` dimension in BOOST microdata.
- **Original data:** BOOST CCI files.

#### 15. Education Sub-Functional Breakdown (Treemap)
- **Chart type:** Treemap
- **Databricks table:** `prd_mega.boost.expenditure_by_country_geo0_func_sub_year` (filtered to func="Education")
- **Key columns:** `func_sub`, `geo0`, `expenditure`, `real_expenditure`
- **Source (mega-boost):** Sub-functional categories (e.g., Primary education, Secondary education, Tertiary education) mapped from country-specific codes during Silver transformation.
- **Original data:** BOOST CCI files.

#### 16. Education Expenditure Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Education")
  - `prd_mega.indicator.admin1_boundaries_gold`
  - `prd_mega.indicator.admin0_disputed_boundaries_gold`
- **Key columns:** `adm1_name`, `year`, `per_capita_expenditure`
- **Source:** Same as Subnational Spending Map (chart 8), but filtered to Education.

#### 17. Education Outcomes Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (outcome index)
  - `prd_mega.indicator.admin1_boundaries_gold`
- **Key columns:** `adm1_name`, `year`, `outcome_index`
- **Source (mega-indicators):** The `outcome_index` is derived from Global Data Lab's subnational Human Development Index (education component). Stored in `prd_mega.indicator.global_data_lab_hd_index` and joined to BOOST data during cross-country aggregation.
- **Original data:** Global Data Lab HDI surveys.

#### 18. Education Spending vs. Outcomes by Region
- **Chart type:** Treemap + scatter comparison
- **Databricks table:** `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Education")
- **Key columns:** `adm1_name`, `per_capita_expenditure`, `outcome_index`
- **Source:** Combines BOOST subnational education expenditure with GDL outcome indices.

---

## Health Page

### "Over Time" Tab

#### 19. Public vs. Private Health Spending
- **Chart type:** Bar/mixed comparison
- **Databricks tables:**
  - `prd_mega.boost.expenditure_by_country_func_econ_year` (public spending, filtered to func="Health")
  - `prd_mega.boost.health_private_expenditure_by_country_year` (out-of-pocket spending)
- **Key columns:** `year`, `real_expenditure` (public), `real_expenditure` (OOP)
- **Source (mega-boost):** Public health spending is BOOST expenditure filtered to "Health" COFOG function.
- **Source (mega-indicators/mega-boost):** Out-of-pocket health expenditure from WHO Global Health Expenditure Database, stored in the BOOST schema.
- **Original data:** BOOST CCI (public), WHO GHED (private/OOP).

#### 20. Total Health Expenditure Over Time
- **Chart type:** Stacked bar (domestic/foreign) + line (real expenditure)
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year` (filtered to func="Health")
- **Key columns:** `year`, `expenditure`, `real_expenditure`, `foreign_funded_expenditure`, `domestic_funded_expenditure`
- **Source:** Same as Total Expenditure (chart 1) but filtered to the Health functional category.
- **Original data:** BOOST CCI files.

#### 21. Health Spending and Outcomes
- **Chart type:** Scatter + line (spending vs. UHC index)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_by_country_func_econ_year` (health spending)
  - `prd_mega.indicator.universal_health_coverage_index_gho` (UHC index)
- **Key columns:** `year`, `real_expenditure` (health), `uhc_index`
- **Source (mega-indicators):** Universal Health Coverage Service Coverage Index from WHO Global Health Observatory (GHO) API.
- **Original data:** WHO GHO database.

#### 22. Operational vs. Capital Spending in Health
- **Chart type:** Stacked area chart
- **Databricks table:** `prd_mega.boost.expenditure_by_country_func_econ_year` (filtered to func="Health")
- **Key columns:** `year`, `econ`, `real_expenditure`
- **Dashboard processing:** Same remapping as Education (chart 13): Wage bill, Capital expenditures, Non-wage recurrent.
- **Source:** Same as Economic Breakdown (chart 5) but filtered to Health.
- **Original data:** BOOST CCI files.

### "Across Space" Tab

#### 23. Central vs. Regional Health Spending (Pie Chart)
- **Chart type:** Pie/Donut chart
- **Databricks table:** `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Health")
- **Key columns:** `geo0`, `expenditure`, `real_expenditure`
- **Source:** Same as Education Central vs. Regional (chart 14) but for Health.

#### 24. Health Sub-Functional Breakdown (Treemap)
- **Chart type:** Treemap
- **Databricks table:** `prd_mega.boost.expenditure_by_country_geo0_func_sub_year` (filtered to func="Health")
- **Key columns:** `func_sub`, `geo0`, `expenditure`, `real_expenditure`
- **Source:** Same as Education Sub-Functional (chart 15) but for Health.

#### 25. Health Expenditure Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Health")
  - `prd_mega.indicator.admin1_boundaries_gold`
  - `prd_mega.indicator.admin0_disputed_boundaries_gold`
- **Source:** Same as Education Expenditure Map (chart 16) but for Health.

#### 26. Health Outcomes Map
- **Chart type:** Choropleth map (Mapbox)
- **Databricks tables:**
  - `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (outcome index)
  - `prd_mega.indicator.admin1_boundaries_gold`
- **Key columns:** `adm1_name`, `year`, `outcome_index`
- **Source (mega-indicators):** The health `outcome_index` is derived from WHO Universal Health Coverage Index (GHO). Stored in `prd_mega.indicator.universal_health_coverage_index_gho` and joined during cross-country aggregation.
- **Original data:** WHO GHO database.

#### 27. Health Spending vs. Outcomes by Region
- **Chart type:** Treemap + scatter comparison
- **Databricks table:** `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` (filtered to func="Health")
- **Key columns:** `adm1_name`, `per_capita_expenditure`, `outcome_index`
- **Source:** Combines BOOST subnational health expenditure with WHO UHC outcome indices.

---

## Reference: All Databricks Tables Used

### BOOST Schema (`prd_mega.boost`)

| Table | Created By | Original Source | Description |
|---|---|---|---|
| `pov_expenditure_by_country_year` | mega-boost | BOOST CCI + PIP | Total expenditure with poverty rate, per capita, CPI-adjusted |
| `expenditure_by_country_func_econ_year` | mega-boost | BOOST CCI | Expenditure by COFOG function and economic classification |
| `expenditure_by_country_geo0_func_sub_year` | mega-boost | BOOST CCI | Expenditure by central/regional allocation and sub-function |
| `expenditure_by_country_geo1_year` | mega-boost | BOOST CCI | Subnational expenditure by admin-1 region |
| `expenditure_and_outcome_by_country_geo1_func_year` | mega-boost | BOOST CCI + GDL/WHO | Subnational expenditure with outcome indices |
| `edu_private_expenditure_by_country_year` | mega-boost | World Bank ICP | Private education expenditure |
| `health_private_expenditure_by_country_year` | mega-boost | WHO GHED | Out-of-pocket health expenditure |
| `data_availability` | mega-boost | Metadata | BOOST source URLs and year coverage |

### Indicator Schema (`prd_mega.indicator`)

| Table | Created By | Original Source | Description |
|---|---|---|---|
| `country` | mega-indicators | World Bank | Country metadata (currency, coordinates, income level) |
| `consumer_price_index` | mega-indicators | World Bank API | CPI for inflation adjustment |
| `subnational_poverty_rate` | mega-indicators | World Bank SPID/GSAP | Poverty rates by subnational region |
| `subnational_population` | mega-indicators | National census offices | Population by subnational region |
| `learning_poverty_rate` | mega-indicators | World Bank EdStats | Learning poverty by country-year |
| `universal_health_coverage_index_gho` | mega-indicators | WHO GHO API | UHC service coverage index |
| `global_data_lab_hd_index` | mega-indicators | Global Data Lab API | Subnational Human Development Index |
| `pefa_by_pillar` | mega-indicators | PEFA Secretariat | PFM assessment scores by 7 pillars |
| `admin1_boundaries_gold` | mega-indicators | World Bank Geospatial | Admin-1 GeoJSON boundaries (harmonized) |
| `admin0_disputed_boundaries_gold` | mega-indicators | World Bank Geospatial | Disputed territory boundaries |
| `indicator_data_availability` | mega-indicators | Metadata | Indicator source URLs and year coverage |

---

## Reference: Processing Pipeline Summary

### mega-boost Pipeline

```
Country Excel Files (BOOST CCI)
    |
    +-- Extract: Excel -> CSV (per country, e.g., ken_extract_microdata_excel_to_csv.py)
    |
    +-- Bronze: Raw CSV loaded with minimal cleaning
    |
    +-- Silver: Country-specific transformations
    |   +-- Map functional codes -> COFOG standard categories
    |   +-- Map economic codes -> standard categories (Wage bill, Capital, etc.)
    |   +-- Extract admin levels (central/regional, geo1 region names)
    |   +-- Flag foreign vs. domestic funding
    |   +-- Harmonize region names to match boundary data
    |
    +-- Gold: Standardized schema per country ({code}_boost_gold)
    |
    +-- Union: All 18 countries -> boost_gold
    |
    +-- Aggregation: 8+ derived tables with CPI adjustment, per-capita, ratios
        +-- Quality validation against CCI reference data (<=5% discrepancy)
```

### mega-indicators Pipeline

```
External APIs (World Bank, WHO, GDL, UNESCO, Ember, PEFA)
    |
    +-- Extract: API calls via wbgapi, requests, or manual upload
    |
    +-- Transform:
    |   +-- Reshape year columns to long format
    |   +-- Standardize column names (snake_case)
    |   +-- Harmonize subnational region names (14+ countries)
    |   +-- Union polygons for boundary corrections
    |   +-- Join with country master table for enrichment
    |
    +-- Gold: Final tables in prd_mega.indicator schema
```
