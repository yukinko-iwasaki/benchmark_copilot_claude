---
layout: page
title: "Claude Agent in Copilot Output — Full Data Lineage"
permalink: /raw-outputs/claude-agent-copilot/
description: "Raw output from the Claude Agent in GitHub Copilot documenting the RPF Country Dashboard data lineage."
---

# Dashboard Data Lineage: Chart-by-Chart Summary

The dashboard (`rpf-country-dash`) reads from **Databricks** tables in two schemas:
- **`prd_mega.boost`** — produced by **mega-boost** (government budget/expenditure microdata)
- **`prd_mega.indicator`** — produced by **mega-indicators** (development outcome indicators)

---

## Overview Page — "Over Time" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 1 | **Total Expenditure Over Time** | Stacked Bar + Line | `boost.pov_expenditure_by_country_year` | **mega-boost** | BOOST Excel microdata (country government budget records) + CPI for inflation adjustment (`mega-indicators` -> World Bank API) |
| 2 | **Per Capita Expenditure Over Time** | Bar + dual-axis Line | `boost.pov_expenditure_by_country_year` | **mega-boost** | BOOST Excel microdata + population (`mega-indicators` -> World Bank API) + poverty rate (`mega-indicators` -> World Bank Poverty & Inequality Platform) |
| 3 | **Functional Breakdown (sector prioritization)** | 100% Stacked Bar | `boost.expenditure_by_country_func_econ_year` | **mega-boost** | BOOST Excel microdata, mapped to COFOG functional categories |
| 4 | **Budget Increment / Functional Growth** | Multi-line (YoY growth) | `boost.expenditure_by_country_func_econ_year` | **mega-boost** | BOOST Excel microdata + CPI (`mega-indicators` -> World Bank API) for real budget computation |
| 5 | **Economic Breakdown** | 100% Stacked Bar | `boost.expenditure_by_country_func_econ_year` | **mega-boost** | BOOST Excel microdata, mapped to economic categories (Wage bill, Capital, etc.) |
| 6 | **PEFA Overall Score** | Dual-axis Line | `indicator.pefa_by_pillar` + `boost.pov_expenditure_by_country_year` | **mega-indicators** (PEFA) + **mega-boost** (poverty overlay) | PEFA.org (manually downloaded assessments) + World Bank poverty data |
| 7 | **PEFA Pillar Heatmap** | Heatmap | `indicator.pefa_by_pillar` | **mega-indicators** | PEFA.org batch downloads; letter grades -> numeric scores averaged per pillar |

## Overview Page — "Across Space" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 8 | **Subnational Spending Choropleth** | Choropleth Map | `boost.expenditure_by_country_geo1_year` + `indicator.admin1_boundaries_gold` + `indicator.admin0_disputed_boundaries_gold` | **mega-boost** (spending) + **mega-indicators** (boundaries) | BOOST Excel microdata + World Bank official GeoJSON admin boundaries |
| 9 | **Subnational Poverty Choropleth** | Choropleth Map | `indicator.subnational_poverty_rate` + `indicator.admin1_boundaries_gold` | **mega-indicators** | World Bank DDH (SPID + GSAP subnational poverty databases) + World Bank admin boundaries |

---

## Education Page — "Over Time" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 10 | **Public vs Private Education Spending** | Horizontal Stacked Bar | `boost.edu_private_expenditure_by_country_year` + `boost.expenditure_by_country_func_econ_year` | **mega-boost** (computes private = total - public) | Public: BOOST Excel microdata. Private: derived using total education spending from `mega-indicators` (UNESCO UIS via World Bank API + OECD SDMX API for private component) |
| 11 | **Education Total Spending** | Stacked Bar + Line | `boost.expenditure_by_country_func_econ_year` (func=Education) | **mega-boost** | BOOST Excel microdata + CPI for inflation adjustment |
| 12 | **Education Outcome** | Dual-axis Lines | `indicator.global_data_lab_hd_index` + `indicator.learning_poverty_rate` + `boost.expenditure_by_country_func_econ_year` | **mega-indicators** (outcomes) + **mega-boost** (spending overlay) | Global Data Lab API (school attendance 6-17yo via R `gdldata` package) + World Bank Education Stats (learning poverty) |
| 13 | **Education Operational vs Capital** | Stacked Area | `boost.expenditure_by_country_func_econ_year` (func=Education) | **mega-boost** | BOOST Excel microdata; econ categories grouped into Wage bill / Capital / Non-wage recurrent |

## Education Page — "Across Space" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 14 | **Central vs Regional Edu Spending** | Donut/Pie | `boost.expenditure_by_country_geo0_func_sub_year` | **mega-boost** | BOOST Excel microdata; admin hierarchy mapped to Central vs Regional |
| 15 | **Education Sub-function Treemap** | Treemap | `boost.expenditure_by_country_geo0_func_sub_year` | **mega-boost** | BOOST Excel microdata; func_sub = Primary, Secondary, Tertiary education etc. |
| 16 | **Education Expenditure Map** | Choropleth Map | `boost.expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **mega-boost** + **mega-indicators** (boundaries) | BOOST Excel microdata + subnational population (`mega-indicators` -> Census Bureau / GDL / country sources) |
| 17 | **Education Outcome Map (Attendance)** | Choropleth Map | `boost.expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **mega-boost** (joins outcome) + **mega-indicators** (attendance data + boundaries) | Global Data Lab API (6-17yo school attendance) |
| 18 | **Education Subnational Rank Sankey** | Sankey Diagram | `boost.expenditure_and_outcome_by_country_geo1_func_year` | **mega-boost** (joins outcome from mega-indicators) | BOOST microdata (per-capita spending) + GDL attendance (outcome index) |

---

## Health Page — "Over Time" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 19 | **Public vs Private Health Spending** | Horizontal Stacked Bar | `boost.health_private_expenditure_by_country_year` + `boost.expenditure_by_country_func_econ_year` | **mega-boost** (computes private = total - public) | Public: BOOST Excel microdata. Private: derived using total health expenditure from `mega-indicators` (WHO GHO API) |
| 20 | **Health Total Spending** | Stacked Bar + Line | `boost.expenditure_by_country_func_econ_year` (func=Health) | **mega-boost** | BOOST Excel microdata + CPI for inflation adjustment |
| 21 | **Health Outcome (UHC Index)** | Dual-axis Lines | `indicator.universal_health_coverage_index_gho` + `boost.expenditure_by_country_func_econ_year` | **mega-indicators** (UHC) + **mega-boost** (spending overlay) | World Bank Health Stats (WHO UHC service coverage index) |
| 22 | **Health Operational vs Capital** | Stacked Area | `boost.expenditure_by_country_func_econ_year` (func=Health) | **mega-boost** | BOOST Excel microdata; econ grouped into Wage bill / Capital / Non-wage recurrent |

## Health Page — "Across Space" Tab

| # | Chart | Type | DB Table | Processing Repo | Ultimate Source |
|---|-------|------|----------|----------------|----------------|
| 23 | **Central vs Regional Health Spending** | Donut/Pie | `boost.expenditure_by_country_geo0_func_sub_year` | **mega-boost** | BOOST Excel microdata |
| 24 | **Health Sub-function Treemap** | Treemap | `boost.expenditure_by_country_geo0_func_sub_year` | **mega-boost** | BOOST Excel microdata |
| 25 | **Health Expenditure Map** | Choropleth Map | `boost.expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **mega-boost** + **mega-indicators** (boundaries) | BOOST microdata + subnational population |
| 26 | **Health Outcome Map (UHC)** | Choropleth Map | `boost.expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **mega-boost** (joins outcome) + **mega-indicators** (UHC data + boundaries) | WHO UHC service coverage index (via Global Data Lab HDI) |
| 27 | **Health Subnational Rank Sankey** | Sankey Diagram | `boost.expenditure_and_outcome_by_country_geo1_func_year` | **mega-boost** (joins outcome from mega-indicators) | BOOST microdata + GDL health index (outcome) |

---

## Summary of External Data Sources

| Source | Fetched By | Used In Charts |
|--------|-----------|----------------|
| **BOOST Excel microdata** (country government budgets) | mega-boost (per-country ETL) | 1-5, 8, 10-11, 13-20, 22-27 |
| **World Bank API** (CPI, GDP, population, poverty) | mega-indicators | 1-2, 4, 6, 8-9, 11, 16, 20, 25 |
| **PEFA.org** (budget institution quality assessments) | mega-indicators | 6-7 |
| **Global Data Lab API** (subnational HDI, school attendance) | mega-indicators (R script) | 12, 17-18, 26-27 |
| **World Bank DDH** (SPID + GSAP subnational poverty) | mega-indicators | 9 |
| **WHO GHO API** (health expenditure, UHC index) | mega-indicators | 19, 21, 26 |
| **World Bank Education Stats** (learning poverty, literacy) | mega-indicators | 12 |
| **UNESCO UIS / OECD SDMX** (education private spending) | mega-indicators | 10 |
| **World Bank Geospatial** (admin boundaries GeoJSON) | mega-indicators | 8-9, 16-17, 25-26 |
| **Census Bureau / country statistical offices** (subnational population) | mega-indicators (per-country scripts) | 8, 16, 25 |

---

## Data Flow Summary

```
External APIs & Files
        |
        v
+------------------+     +----------------------+
|  mega-indicators |     |     mega-boost        |
|  (outcome data)  |     |  (budget/expenditure) |
|                  |     |                       |
|  World Bank API  |     |  BOOST Excel files    |
|  WHO GHO API     |     |  (18 countries)       |
|  Global Data Lab |     |  bronze->silver->gold |
|  PEFA.org        |     |  per-country ETL      |
|  Census Bureau   |     |                       |
+--------+---------+     +----------+------------+
         |                          |
         v                          v
   prd_mega.indicator         prd_mega.boost
   (Databricks)               (Databricks)
         |                          |
         +----------+---------------+
                    v
            rpf-country-dash
            (Plotly Dash app)
            27 visualizations
```
