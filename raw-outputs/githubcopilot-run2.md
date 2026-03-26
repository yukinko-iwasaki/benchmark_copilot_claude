---
layout: page
title: "GitHub Copilot Output — Run 2"
permalink: /raw-outputs/githubcopilot-run2/
description: "Raw output from GitHub Copilot (Run 2) documenting the RPF Country Dashboard data lineage."
---

# RPF Country Dashboard — Data Lineage

Chart-by-chart summary of all 27 visualizations, their data sources, and how the underlying data is processed across three repos:
- **rpf-country-dash** — Dash/Plotly visualization app
- **mega-boost** — Government budget data ETL pipelines
- **mega-indicators** — External indicator data pipelines

---

## Overview Page

### Over Time Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 1 | **Total Expenditure** | Stacked Bar + Line | `pov_expenditure_by_country_year` | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget spreadsheets (Ministry of Finance) from 18 countries | Per-country Excel → CSV extract → country-specific DLT (bronze→silver→gold) → union into `boost_gold` → aggregate by [country, year] → join CPI (WB) + population (WB/UN) for real & per-capita values → join poverty rate |
| 2 | **Per Capita Expenditure + Poverty** | Dual-axis Bars + Line | `pov_expenditure_by_country_year` | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data + **World Bank** poverty rate | Same pipeline as #1; poverty_rate is right-joined on [year, country] |
| 3 | **Spending by Functional Categories** | Stacked Bar | `expenditure_by_country_func_econ_year` | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data (COFOG functional classification: Education, Health, etc.) | `boost_gold` → aggregate by [country, year, func, econ] → join CPI (**World Bank** `FP.CPI.TOTL`) + population (**World Bank** `SP.POP.TOTL`) for deflation & per-capita |
| 4 | **Spending by Economic Categories** | Stacked Bar | `expenditure_by_country_func_econ_year` | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data (economic classification: wages, capital, goods & services, etc.) | Same table as #3, grouped by `econ` instead of `func` |
| 5 | **Budget Growth Rate** | Multi-line (YoY % + CAGR) | `expenditure_by_country_func_econ_year` | **mega-boost** + **dashboard** transformation | Government budget data | Same source as #3; dashboard computes YoY growth: `(current - prior) / prior × 100` per functional category, plus compound annual growth rate |

### Across Space Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 6 | **Regional Expenditure Map** | Choropleth | `expenditure_by_country_geo1_year` + `admin1_boundaries_gold` | **mega-boost** (expenditure) + **mega-indicators** `geo/admin_boundaries_dlt.py` (boundaries) | Government budget data (subnational allocation) + **World Bank Official Boundaries** GeoJSON | `boost_gold` → aggregate by [country, adm1, year] → join subnational population; boundaries: GeoJSON → harmonize region names (60+ corrections) → union polygons for harmonized regions → gold |
| 7 | **Regional Poverty Map** | Choropleth | `subnational_poverty_rate` + `admin1_boundaries_gold` | **mega-indicators** `poverty/subnational_poverty/` | **World Bank SPID** + **GSAP** APIs | Download SPID (older years) + GSAP (lineup years) Excel → validate overlap → harmonize region names → select poverty line by income classification ($3.00/LIC, $4.20/LMC, $8.30/UMC) |

### Quality of Budget Institutions

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 8 | **PEFA Overall Score** | Line + Poverty overlay | `pefa_by_pillar` + poverty from stored data | **mega-indicators** `pefa/pefa_transform_load.py` | **PEFA** (pefa.org) — batch-downloaded assessment data (2011 & 2016 frameworks) | Manual Excel upload → letter grades (A–D+) mapped to numeric (1–4.5) → grouped into 7 pillars → averaged; dashboard calculates overall = mean of 7 pillars |
| 9 | **PEFA by Pillar (Heatmap)** | Heatmap | `pefa_by_pillar` | **mega-indicators** `pefa/pefa_transform_load.py` | **PEFA** (pefa.org) | Same as #8; pivoted by pillar × year, color-coded by score (1–4) |

---

## Education Page

### Over Time Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 10 | **Public vs Private Education Spending** | Horizontal Stacked Bar (% share) | `expenditure_by_country_func_econ_year` (func=Education) + `edu_private_spending` | **mega-boost** (public) + **mega-indicators** `education/education_private_spending.py` (private) | Government budgets (public) + **OECD SDMX API** indicator PRY_TRY/EARLYCHILDEDU (private) | Public: boost pipeline filtered to Education func; Private: OECD CSV → sum by country/year → convert % to decimal → multiply by GDP for absolute LCU value |
| 11 | **Total Education Spending** | Stacked Bar + Line | `expenditure_by_country_func_econ_year` (func=Education) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | Same as #3, filtered to func="Education"; shows central vs regional split + inflation-adjusted line |
| 12 | **Education Spending vs Learning Outcomes** | Dual-axis Scatter | `expenditure_by_country_func_econ_year` + `learning_poverty_rate` + `global_data_lab_hd_index` | **mega-boost** (spending) + **mega-indicators** `education/learning_poverty.py` + `human_development/global_data_lab_hid_transform_load_dlt.py` | Government budgets + **World Bank/UNESCO UIS** indicator `SE.LPV.PRIM` + **Global Data Lab** (attendance) | Plots both 6–17yo attendance rate (GDL) and learning poverty rate (WB/UNESCO) against per-capita spending |
| 13 | **Operational vs Capital Spending (Education)** | Stacked Area + Narrative | `expenditure_by_country_func_econ_year` (func=Education) | **mega-boost** + **dashboard** `components/func_operational_vs_capital_spending.py` | Government budget data (economic classification) | Econ categories mapped to 3 groups (Wage bill, Capital, Non-wage recurrent) → calculate proportions within each year → AI narrative generated |

### Across Space Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 14 | **Central vs Regional Spending (Donut)** | Pie/Donut | `expenditure_by_country_geo0_func_sub_year` (func=Education) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | `boost_gold` → aggregate by [country, year, geo0, func, func_sub] → CPI-deflate; geo0 = "Central" or "Regional" |
| 15 | **Education Sub-functions (Treemap)** | Treemap | `expenditure_by_country_geo0_func_sub_year` (func=Education) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | Same as #14; hierarchy: func_sub (Primary/Secondary/Tertiary) → geo0 (Central/Regional) |
| 16 | **Regional Education Spending Map** | Choropleth | `expenditure_and_outcome_by_country_geo1_func_year` (func=Education) + `admin1_boundaries_gold` | **mega-boost** (spending) + **mega-indicators** (boundaries + outcomes) | Government budgets + **Global Data Lab** HDI + **WB Official Boundaries** | `boost_gold` → aggregate by [country, adm1, func, year] → join subnational population → inner join with `global_data_lab_hd_index` (attendance) → rank by spending & outcomes |
| 17 | **Regional School Attendance Map** | Choropleth | `expenditure_and_outcome_by_country_geo1_func_year` (outcome_index = attendance_6to17yo) + `admin1_boundaries_gold` | **mega-indicators** `human_development/global_data_lab_hid_transform_load_dlt.py` | **Global Data Lab** Human Development Index (subnational education outcomes) | GDL data → harmonize subnational names → convert attendance % to decimal → joined with expenditure in mega-boost aggregation |
| 18 | **Spending vs Attendance (Sankey)** | Sankey | `expenditure_and_outcome_by_country_geo1_func_year` | **mega-boost** (spending ranks) + **mega-indicators** (outcome ranks) | Government budgets + **Global Data Lab** | Regions ranked by per_capita_real_expenditure (left) and by attendance_6to17yo (right); flows connect same region across both rankings |

---

## Health Page

### Over Time Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 19 | **Public vs Private Health Spending** | Horizontal Stacked Bar (% share) | `expenditure_by_country_func_econ_year` (func=Health) + `health_expenditure` | **mega-boost** (public) + **mega-indicators** `health/health_expenditure.py` (private) | Government budgets (public) + **WHO Global Health Observatory** API indicators GHED_OOPSCHE_SHA2011, GHED_CHEGDP_SHA2011, etc. (private/OOP) | Public: boost filtered to Health; Private: WHO GHO API → fetch 4 indicators → merge → multiply CHE% × GDP for LCU → join country table |
| 20 | **Total Health Spending** | Stacked Bar + Line | `expenditure_by_country_func_econ_year` (func=Health) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | Same as #3, filtered to func="Health" |
| 21 | **Health Spending vs UHC Index** | Scatter + Trend | `expenditure_by_country_func_econ_year` + `universal_health_coverage_index_GHO` | **mega-boost** (spending) + **mega-indicators** `health/sdg_health.py` (UHC) | Government budgets + **World Bank Health DB** indicator `SH.UHC.SRVS.CV.XD` (sourced from WHO GHO) | Spending: per-capita from boost; UHC: WB API → melt → join country table; scatter correlates spending × UHC index |
| 22 | **Operational vs Capital Spending (Health)** | Stacked Area + Narrative | `expenditure_by_country_func_econ_year` (func=Health) | **mega-boost** + **dashboard** components | Government budget data | Same logic as #13 but filtered to func="Health" |

### Across Space Tab

| # | Chart | Viz Type | Data Table(s) | Processed By | Original Source | How It's Created |
|---|-------|----------|---------------|-------------|----------------|-----------------|
| 23 | **Central vs Regional Spending (Donut)** | Pie/Donut | `expenditure_by_country_geo0_func_sub_year` (func=Health) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | Same as #14 but filtered to Health |
| 24 | **Health Sub-functions (Treemap)** | Treemap | `expenditure_by_country_geo0_func_sub_year` (func=Health) | **mega-boost** `cross_country_aggregate_dlt.py` | Government budget data | Same as #15 but filtered to Health |
| 25 | **Regional Health Spending Map** | Choropleth | `expenditure_and_outcome_by_country_geo1_func_year` (func=Health) + `admin1_boundaries_gold` | **mega-boost** + **mega-indicators** | Government budgets + **Global Data Lab** + **WB Official Boundaries** | Same as #16 but func=Health; outcome_index = health_index from GDL |
| 26 | **Regional Health Outcome Map** | Choropleth | `expenditure_and_outcome_by_country_geo1_func_year` (outcome_index = health_index) + `admin1_boundaries_gold` | **mega-indicators** `human_development/global_data_lab_hid_transform_load_dlt.py` | **Global Data Lab** HDI (subnational health outcomes) | Same as #17 but using health_index instead of attendance |
| 27 | **Spending vs Health Outcome (Sankey)** | Sankey | `expenditure_and_outcome_by_country_geo1_func_year` | **mega-boost** + **mega-indicators** | Government budgets + **Global Data Lab** | Same as #18 but ranking by health_index instead of attendance |

---

## Data Source Summary

| Original Source | Tables Created | Processing Repo |
|----------------|---------------|------|
| **Government budget data** (18 countries' Ministry of Finance Excel files) | `boost_gold`, all `expenditure_*` tables, `pov_expenditure_by_country_year` | mega-boost |
| **World Bank API** (`FP.CPI.TOTL`) | `consumer_price_index` → used as `cpi_factor` for inflation adjustment | mega-indicators |
| **World Bank API** (`SP.POP.TOTL`) | `population` → used for per-capita calculations | mega-indicators |
| **World Bank API** (poverty endpoints) | `poverty_rate` → joined with expenditure | mega-indicators |
| **World Bank SPID + GSAP** APIs | `subnational_poverty_rate` | mega-indicators |
| **PEFA** (pefa.org batch downloads) | `pefa_by_pillar` | mega-indicators |
| **Global Data Lab** HDI | `global_data_lab_hd_index` (attendance_6to17yo, health_index) | mega-indicators |
| **WHO Global Health Observatory** API | `health_expenditure`, `universal_health_coverage_index_GHO` | mega-indicators |
| **OECD SDMX API** | `edu_private_spending` | mega-indicators |
| **World Bank/UNESCO UIS** (`SE.LPV.PRIM`) | `learning_poverty_rate` | mega-indicators |
| **World Bank Official Boundaries** GeoJSON | `admin1_boundaries_gold`, `admin0_disputed_boundaries_gold` | mega-indicators |
| **World Bank API** (economy metadata) | `country` (coordinates, currency, income level) | mega-indicators |

---

## Key Data Processing Patterns

### mega-boost Pipeline (Government Budget Data)

```
Per-country Excel files (Ministry of Finance)
  ↓ Extract: {COUNTRY}_extract_microdata_excel_to_csv.py
Raw CSV files
  ↓ Transform & Load: {COUNTRY}_transform_load_dlt.py (bronze → silver → gold)
{COUNTRY}_boost_gold tables (18 countries)
  ↓ Union: cross_country_aggregate_dlt.py
boost_gold (master harmonized microdata)
  ↓ Aggregate + Join CPI + Population
Output tables: expenditure_by_country_*, pov_expenditure_*, expenditure_and_outcome_*
```

### mega-indicators Pipeline (External Data)

```
External APIs (World Bank, WHO, OECD, Global Data Lab)
  ↓ Fetch via Python (wbgapi, requests, SDMX)
Raw data
  ↓ Transform: melt wide→long, harmonize names, convert units
  ↓ Load to prd_mega.indicator schema
Output tables: population, consumer_price_index, learning_poverty_rate,
              universal_health_coverage_index_GHO, health_expenditure,
              edu_private_spending, pefa_by_pillar, subnational_poverty_rate,
              global_data_lab_hd_index, admin1_boundaries_gold, country
```

### Cross-Repo Dependencies

```
mega-indicators (CPI, population, poverty, outcomes, boundaries)
        ↓ consumed by
mega-boost (joins with boost_gold to create enriched aggregates)
        ↓ consumed by
rpf-country-dash (queries tables, creates 27 charts)
```
