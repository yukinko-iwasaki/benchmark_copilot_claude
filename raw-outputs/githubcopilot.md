---
layout: page
title: "GitHub Copilot Output — Full Data Lineage"
permalink: /raw-outputs/githubcopilot/
description: "Raw output from GitHub Copilot (Run 1) documenting the RPF Country Dashboard data lineage."
---

# RPF Country Dashboard — Data Source Summary

This document maps every chart in the dashboard to its underlying data tables, processing pipelines, and original external sources.

---

## Overview Page — "Over Time" Tab

### 1. Total Expenditure
**Chart ID:** `overview-total` — bar + line chart
**Question:** *How has total expenditure changed over time?*

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.boost.pov_expenditure_by_country_year` | |
| mega-boost | `pov_expenditure_by_country_year` — joins `expenditure_by_country_year` (from `boost_gold`) with `poverty_rate` | |
| mega-boost | `boost_gold` — unions per-country gold tables. Each country has its own `*_transform_load_dlt.py` | **BOOST Database** (World Bank) |
| CPI adjustment | `cpi_factor` — base-year CPI from `consumer_price_index` | **World Bank API** (`FP.CPI.TOTL`) |
| Population | `prd_mega.indicator.population` | **World Bank API** (`SP.POP.TOTL`) — UN / Eurostat |
| Poverty rate | `prd_mega.indicator.poverty_rate` | **World Bank Poverty & Inequality Platform** |

### 2. Per Capita Expenditure with Poverty Rate
**Chart ID:** `overview-per-capita` — bar + dual-axis line
**Question:** *How has per capita expenditure changed over time?*

Same data as chart 1 (`pov_expenditure_by_country_year`). Adds a poverty rate overlay from `prd_mega.indicator.poverty_rate`.

| Source | Detail |
|---|---|
| **BOOST Database** | Central + regional expenditure |
| **World Bank CPI** | Inflation adjustment |
| **World Bank / UN / Eurostat** | Population for per-capita |
| **World Bank Poverty & Inequality Platform** | Poverty rate (income-level-specific threshold) |

### 3. Functional Breakdown (COFOG)
**Chart ID:** `functional-breakdown` — stacked bar
**Question:** *How has sector prioritization changed over time?*

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.boost.expenditure_by_country_func_econ_year` → client-side aggregation to `expenditure_by_country_func_year` | |
| mega-boost | `expenditure_by_country_func_econ_year` ← `expenditure_by_country_admin_func_sub_econ_sub_year` ← `boost_gold` | **BOOST Database** |
| Population | `prd_mega.indicator.population` | **World Bank / UN / Eurostat** |

### 4. Budget Increment Analysis (YoY Growth)
**Chart ID:** `func-growth` — line chart
**Question:** *How do budgets for functional categories fluctuate over time?*

Same data as chart 3 (`expenditure_by_country_func_year`), specifically `domestic_funded_budget` and `real_domestic_funded_budget`.

| Source | Detail |
|---|---|
| **BOOST Database** | Approved budget data (excluding foreign-funded) |
| **World Bank CPI** | Inflation adjustment for real budget |

### 5. Economic Breakdown
**Chart ID:** `economic-breakdown` — stacked area
**Question:** *How much was spent on each economic category?*

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.boost.expenditure_by_country_func_econ_year` → client-side aggregation to `expenditure_by_country_econ_year` | |
| mega-boost | Same pipeline as chart 3, grouped by economic classification (Wage bill, Capital expenditures, etc.) | **BOOST Database** |

### 6. PEFA — Quality of Budget Institutions
**Chart IDs:** `pefa-overall`, `pefa-by-pillar` — line charts
**Question:** *How did the quality of budget institutions change over time?*

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.indicator.pefa_by_pillar` | |
| mega-indicators | `pefa_transform_load.py` — reads bronze tables, maps letter grades (A–D+) to numeric scores, aggregates by 7 PEFA pillars | |
| Raw data | Manually downloaded batch exports | **PEFA Secretariat** (https://www.pefa.org) — 2011 & 2016 frameworks |
| Poverty overlay | `pov_expenditure_by_country_year` | **World Bank Poverty & Inequality Platform** |

---

## Overview Page — "Across Space" Tab

### 7. Subnational Spending Map
**Chart ID:** `subnational-spending` — choropleth (per capita or total)
**Question:** *How much was spent in each region?*

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.boost.expenditure_by_country_geo1_year` | |
| mega-boost | `expenditure_by_country_geo1_year` ← `expenditure_by_country_geo1_func_year` ← `boost_gold` | **BOOST Database** |
| Subnational population | `prd_mega.indicator.subnational_population` — consolidated from per-country scripts | **Country-specific census / statistics offices** |
| Admin boundaries | `prd_mega.indicator.admin1_boundaries_gold` | **World Bank Official Boundaries** (GeoJSON) |
| Disputed boundaries | `prd_mega.indicator.admin0_disputed_boundaries_gold` | **World Bank Official Boundaries** |

### 8. Subnational Poverty Map
**Chart ID:** `subnational-poverty` — choropleth

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.indicator.subnational_poverty_rate` | |
| mega-indicators | `subnational_poverty_index_transform_load_dlt.py` ← `poverty_rate_spid_gsap` | |
| Raw data | `subnational_poverty_index_extract_transform.py` — fetches and merges SPID + GSAP | **World Bank SPID** + **GSAP** |

---

## Education Page — "Over Time" Tab

### 9. Who Pays for Education? (Public vs. Private)
**Chart ID:** `education-public-private` — bar chart

| Layer | Table / Pipeline | Source |
|---|---|---|
| Public spending | Filtered from `expenditure_by_country_func_year` (`func == 'Education'`) | **BOOST Database** |
| Private spending | `prd_mega.boost.edu_private_expenditure_by_country_year` | |
| mega-boost | Private = ICP total education spending − BOOST public education spending | |
| ICP education data | `prd_mega.indicator.edu_spending` ← `education_spending_icp.py` | **International Comparison Program (ICP)** — World Bank |

### 10. Total Education Expenditure
**Chart ID:** `education-total` — bar + line
**Question:** *How has govt spending on education changed over time?*

Same as public education spending from chart 9.
- Source: **BOOST Database** + **World Bank CPI**

### 11. Public Spending & Education Outcome
**Chart ID:** `education-outcome` — dual-axis

| Layer | Table / Pipeline | Source |
|---|---|---|
| Public spending | Education spending from `expenditure_by_country_func_year` | **BOOST Database** |
| Learning poverty | `prd_mega.indicator.learning_poverty_rate` ← `learning_poverty.py` | **World Bank & UNESCO UIS** (`SE.LPV.PRIM`) |
| HD Index | `prd_mega.indicator.global_data_lab_hd_index` ← `global_data_lab_hdi_extract.r` + transform DLT | **Global Data Lab** (education index, school attendance) |

### 12. Operational vs. Capital Spending — Education
**Chart ID:** `econ-breakdown-func-edu` — stacked area

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard | `expenditure_by_country_func_econ_year` filtered for Education, grouped into Wage bill / Capital / Non-wage recurrent | |
| mega-boost | Same pipeline as chart 3 | **BOOST Database** |

---

## Education Page — "Across Space" Tab

### 13. Centrally vs. Geographically Allocated Education Spending
**Chart IDs:** `education-central-vs-regional` (pie), `education-sub-func` (treemap)

| Layer | Table / Pipeline | Source |
|---|---|---|
| Dashboard query | `prd_mega.boost.expenditure_by_country_geo0_func_sub_year` | |
| mega-boost | `boost_gold` grouped by `geo0` (Central/Regional) and `func_sub` | **BOOST Database** |

### 14. Education Spending vs. Outcomes Map
**Chart IDs:** `education-expenditure-map`, `education-outcome-map`, `education-subnational` — dual choropleth + bar rank

| Layer | Table / Pipeline | Source |
|---|---|---|
| Expenditure | `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` | **BOOST Database** + subnational population |
| Outcome | `attendance_6to17yo` from `global_data_lab_hd_index` | **Global Data Lab** (school attendance ages 6–17) |
| Boundaries | `admin1_boundaries_gold` | **World Bank Official Boundaries** |

---

## Health Page — "Over Time" Tab

### 15. Who Pays for Healthcare? (Public vs. Private)
**Chart ID:** `health-public-private` — bar chart

| Layer | Table / Pipeline | Source |
|---|---|---|
| Public spending | Filtered from `expenditure_by_country_func_year` (`func == 'Health'`) | **BOOST Database** |
| Private (OOP) spending | `prd_mega.boost.health_private_expenditure_by_country_year` | |
| mega-boost | OOP = CHE × OOP% / 100, CPI-adjusted | |
| Health expenditure | `prd_mega.indicator.health_expenditure` ← `health_expenditure.py` | **WHO Global Health Expenditure Database** (GHO API) + **World Bank GDP** |

### 16. Total Health Expenditure
**Chart ID:** `health-total` — bar + line
**Question:** *How has govt spending on health changed over time?*

Same as public health spending from chart 15.
- Source: **BOOST Database** + **World Bank CPI**

### 17. Public Spending & Health Outcome
**Chart ID:** `health-outcome` — dual-axis

| Layer | Table / Pipeline | Source |
|---|---|---|
| Public spending | Health spending from `expenditure_by_country_func_year` | **BOOST Database** |
| UHC Index | `prd_mega.indicator.universal_health_coverage_index_gho` ← `sdg_health.py` | **WHO Global Health Observatory** (SDG 3.8.1, `SH.UHC.SRVS.CV.XD`) |

### 18. Operational vs. Capital Spending — Health
**Chart ID:** `econ-breakdown-func-health` — stacked area

Same structure as chart 12 but filtered for Health.
- Source: **BOOST Database**

---

## Health Page — "Across Space" Tab

### 19. Centrally vs. Geographically Allocated Health Spending
**Chart IDs:** `health-central-vs-regional` (pie), `health-sub-func` (treemap)

Same structure as chart 13 but filtered for Health.
- Source: **BOOST Database**

### 20. Health Spending vs. Outcomes Map
**Chart IDs:** `health-expenditure-map`, `health-outcome-map`, `health-subnational` — dual choropleth + bar rank

| Layer | Table / Pipeline | Source |
|---|---|---|
| Expenditure | `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` | **BOOST Database** + subnational population |
| Outcome | `health_index` from `global_data_lab_hd_index` | **Global Data Lab** (SHDI — health index) |
| Boundaries | `admin1_boundaries_gold` | **World Bank Official Boundaries** |

---

## Cross-Cutting: Shared Supporting Tables

| Indicator Table | Processing Repo / Script | Original Source |
|---|---|---|
| `consumer_price_index` | mega-indicators / `consumer_price_index.py` | **World Bank API** (`FP.CPI.TOTL`) |
| `population` | mega-indicators / `national_population.py` | **World Bank API** (`SP.POP.TOTL`) — UN, Eurostat |
| `subnational_population` | mega-indicators / `subnational_population_official_dlt.py` — consolidates per-country scripts | **Country-specific census / statistics offices** |
| `country` | mega-indicators / `country.py` | **World Bank API** via wbgapi |
| `admin1_boundaries_gold` | mega-indicators / `admin_boundaries_extract.py` + `admin_boundaries_dlt.py` | **World Bank Official Boundaries** (GeoJSON) |
| `gdp` | mega-indicators / `gdp.py` | **World Bank / OECD National Accounts** |
| `boost_gold` | mega-boost / per-country ETL → `cross_country_aggregate_dlt.py` | **BOOST Database** — raw budget/expenditure data from 18 countries |

---

## Primary External Data Sources

| # | Source | Used For |
|---|---|---|
| 1 | **BOOST Database** (World Bank) | Core budget & expenditure microdata from national authorities |
| 2 | **World Bank Indicators API** | CPI, population, GDP, poverty rates, learning poverty |
| 3 | **Global Data Lab** | Subnational HDI, health index, education attendance |
| 4 | **WHO Global Health Observatory** | UHC index, health expenditure (CHE, OOP) |
| 5 | **International Comparison Program (ICP)** | Total education spending (public + private) |
| 6 | **PEFA Secretariat** | Public Expenditure and Financial Accountability assessments |
| 7 | **World Bank SPID + GSAP** | Subnational poverty rates |
| 8 | **World Bank Official Boundaries** | Admin-level GeoJSON boundaries |
| 9 | **Country-specific national statistics offices** | Subnational population data |
