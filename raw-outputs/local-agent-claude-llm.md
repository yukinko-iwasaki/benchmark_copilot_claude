---
layout: page
title: "Local Agent with Claude LLM Output — Supplementary (Incomplete)"
permalink: /raw-outputs/local-agent-claude-llm/
description: "Raw output from a local agent framework using Claude as the LLM. This run produced incomplete results — included as a supplementary observation."
---

*Note: This output is incomplete. The local agent did not spawn sub-agents and stopped partway through the task, leaving the health section partially documented. It is included as a supplementary data point, not as a primary comparison.*

---

# Dashboard Data Lineage: Chart-by-Chart Summary

This document traces every chart in the **rpf-country-dash** dashboard back to its underlying data tables, processing pipelines in **mega-boost** and **mega-indicators**, and original external sources.

---

## Overview Page — "Over Time" Tab

### 1. Total Expenditure (`overview-total`)
- **What it shows:** Total public expenditure over time (central vs. regional stacked bar), with inflation-adjusted line overlay.
- **Dashboard query:** `prd_mega.boost.pov_expenditure_by_country_year`
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> `expenditure_by_country_year` aggregates `boost_gold` by country+year, sums `executed` for total expenditure, sums regional `executed` for decentralized. Joined with `cpi_factor` (for inflation adjustment) and `population`. Then `pov_expenditure_by_country_year` joins this with poverty rate.
- **Upstream sources:**
  - **BOOST microdata** (per-country Excel from World Bank BOOST team, extracted via `{COUNTRY}_extract_microdata_excel_to_csv.py`, harmonized via `{COUNTRY}_transform_load_dlt.py`) -> `boost_gold`
  - **CPI** (`prd_mega.indicator.consumer_price_index`): `mega-indicators/consumer_price_index.py` -- World Bank API indicator `FP.CPI.TOTL`
  - **Population** (`prd_mega.indicator.population`): `mega-indicators/population/national_population.py` -- World Bank API indicators `SP.POP.TOTL`, `SP.POP.TOTL.FE.IN` (source: UN, WB, Eurostat)

### 2. Per Capita Expenditure with Poverty Rate (`overview-per-capita`)
- **What it shows:** Per capita expenditure (nominal + inflation-adjusted) with poverty rate overlay on secondary axis.
- **Dashboard query:** `prd_mega.boost.pov_expenditure_by_country_year` (same as chart 1)
- **Processing:** Same as chart 1. Poverty rate joined from `prd_mega.indicator.poverty_rate`.
- **Additional upstream source:**
  - **Poverty rate** (`prd_mega.indicator.poverty_rate`): `mega-indicators/poverty/poverty.py` -- World Bank Poverty and Inequality Platform API indicators `SI.POV.DDAY`, `SI.POV.LMIC`, `SI.POV.UMIC`. The applicable rate depends on country income level (LIC->$3.00, LMIC->$4.20, UMIC/HIC->$8.30).

### 3. Functional Breakdown (`functional-breakdown`)
- **What it shows:** Stacked bar of expenditure % by COFOG functional categories over time.
- **Dashboard query:** `prd_mega.boost.expenditure_by_country_func_econ_year` -> aggregated client-side in `app.py` to `expenditure_by_country_func_year`.
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` chains: `boost_gold` -> `expenditure_by_country_admin_func_sub_econ_sub_year` -> `expenditure_by_country_func_econ_year` (joined with `population`) -> grouped by func.
- **Source:** BOOST microdata (functional classification follows COFOG), CPI, Population (same as above).

### 4. Budget Increment Analysis (`func-growth`)
- **What it shows:** Year-on-year budget growth rate by functional category (last 5 years).
- **Dashboard query:** Same `expenditure_by_country_func_econ_year`, processed in `rpf-country-dash/components/budget_increment_analysis.py`.
- **Source:** BOOST microdata (approved budget, domestic vs. foreign), CPI for inflation adjustment.

### 5. Economic Breakdown (`economic-breakdown`)
- **What it shows:** Expenditure % by economic categories (Wage bill, Capital expenditures, Goods & services, etc.) over time.
- **Dashboard query:** `expenditure_by_country_func_econ_year` -> aggregated client-side to `expenditure_by_country_econ_year`.
- **Source:** BOOST microdata (economic classification).

### 6. PEFA Overall & By Pillar (`pefa-overall`, `pefa-by-pillar`)
- **What it shows:** Quality of budget institutions (overall PEFA score + 7 pillars) over time, with poverty rate overlay.
- **Dashboard query:** `prd_mega.indicator.pefa_by_pillar`
- **Processing (mega-indicators):** `pefa/pefa_transform_load.py` -- PEFA assessment scores for 2011 and 2016 frameworks downloaded from https://www.pefa.org/assessments/batch-downloads, manually imported to bronze tables. Letter grades (A-D+) converted to numeric (4-1.5), averaged across PIs within each of 7 pillars.
- **Source:** PEFA Secretariat (batch download).

---

## Overview Page — "Across Space" Tab

### 7. Subnational Spending Map (`subnational-spending`)
- **What it shows:** Choropleth map of per capita or total expenditure by admin1 region.
- **Dashboard query:** `prd_mega.boost.expenditure_by_country_geo1_year`
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> `expenditure_by_country_geo1_func_year` (joins `boost_gold` by geo1 with subnational population and CPI) -> aggregated to `expenditure_by_country_geo1_year`.
- **Upstream sources:**
  - BOOST microdata, CPI
  - **Subnational population**: per-country scripts in `mega-indicators/population/` (national statistical offices, census)
  - **Admin boundaries** (`admin1_boundaries_gold`): `mega-indicators/geo/admin_boundaries_dlt.py` (geoBoundaries / national sources)

### 8. Subnational Poverty Map (`subnational-poverty`)
- **What it shows:** Choropleth map of subnational poverty rates.
- **Dashboard query:** `prd_mega.indicator.subnational_poverty_rate`
- **Processing (mega-indicators):** `poverty/subnational_poverty/subnational_poverty_index_extract_transform.py` extracts from SPID & GSAP via World Bank DDH OpenAPI; `subnational_poverty_index_transform_load_dlt.py` harmonizes region names.
- **Source:** World Bank SPID (Subnational Poverty & Inequality Database) & GSAP (Global Subnational Atlas of Poverty).

---

## Education Page — "Over Time" Tab

### 9. Public vs. Private Education Spending (`education-public-private`)
- **What it shows:** Public education expenditure (BOOST) alongside estimated private education expenditure.
- **Dashboard queries:** Public from `expenditure_by_country_func_econ_year` filtered to func=Education; Private from `prd_mega.boost.edu_private_expenditure_by_country_year`.
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> `edu_private_expenditure_by_country_year` = total ICP education spending - BOOST public education spending, inflation-adjusted via CPI.
- **Upstream sources:**
  - BOOST microdata, CPI
  - **ICP Education spending** (`prd_mega.indicator.edu_spending`): `mega-indicators/education/education_spending_icp.py` -- International Comparison Program (ICP) databases (2005, 2011, 2017, 2021 rounds), indicator code `9120000`, in billions of current LCU.

### 10. Total Education Expenditure (`education-total`)
- **What it shows:** Education spending over time (central vs. regional stacked bar), with inflation-adjusted line.
- **Dashboard query:** `expenditure_by_country_func_econ_year` filtered to func=Education.
- **Source:** BOOST microdata, CPI.

### 11. Education Spending vs. Learning Poverty (`education-outcome`)
- **What it shows:** Dual-axis chart of per capita real education spending vs. learning poverty rate.
- **Dashboard query:** `prd_mega.indicator.learning_poverty_rate`
- **Processing (mega-indicators):** `education/learning_poverty.py` -- World Bank Education Statistics database (db=12), indicator `SE.LPV.PRIM`.
- **Source:** World Bank & UNESCO Institute for Statistics (UIS).

### 12. Operational vs. Capital Spending -- Education (`econ-breakdown-func-edu`)
- **What it shows:** Stacked area chart of Wage bill vs. Capital expenditures vs. Non-wage recurrent as % of total education spending.
- **Dashboard query:** `expenditure_by_country_func_econ_year` filtered to func=Education, pivoted by econ categories in `rpf-country-dash/components/func_operational_vs_capital_spending.py`.
- **Source:** BOOST microdata (economic + functional classification), CPI.

---

## Education Page — "Across Space" Tab

### 13. Centrally vs. Geographically Allocated Education Spending (`education-central-vs-regional`, `education-sub-func`)
- **What it shows:** Pie chart of central vs. regional allocation; treemap of sub-functional categories (Primary, Secondary, Tertiary, etc.).
- **Dashboard query:** `prd_mega.boost.expenditure_by_country_geo0_func_sub_year`
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> aggregates `boost_gold` by country, year, geo0, func, func_sub with CPI adjustment.
- **Source:** BOOST microdata (geo + sub-functional classification), CPI.

### 14. Education Expenditure Map & Education Outcome Map (`education-expenditure-map`, `education-outcome-map`)
- **What it shows:** Side-by-side choropleths -- per capita education spending by region vs. school attendance (6-17yo) by region.
- **Dashboard query:** `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year`
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> joins `expenditure_by_country_geo1_func_year` with `global_data_lab_hd_index` (outcome). For Education: outcome = `attendance_6to17yo`.
- **Upstream sources:**
  - BOOST, CPI, Subnational population, Admin boundaries
  - **Global Data Lab HD Index** (`prd_mega.indicator.global_data_lab_hd_index`): `mega-indicators/human_development/global_data_lab_hdi_extract.r` extracts SHDI + education attendance from Global Data Lab API -> `global_data_lab_hid_transform_load_dlt.py` harmonizes region names and selects `attendance_6to17yo`, `health_index`, `education_index`, `income_index`.

### 15. Education Subnational Scatter/Rank (`education-subnational`)
- **What it shows:** Scatter plot ranking regions by per capita education spending vs. school attendance outcome.
- **Dashboard query:** Same `expenditure_and_outcome_by_country_geo1_func_year`.
- **Source:** Same as chart 14.

---

## Health Page — "Over Time" Tab

### 16. Public vs. Private Health Spending (`health-public-private`)
- **What it shows:** Public health expenditure (BOOST) alongside out-of-pocket health expenditure.
- **Dashboard queries:** Public from `expenditure_by_country_func_econ_year` filtered to func=Health; Private from `prd_mega.boost.health_private_expenditure_by_country_year`.
- **Processing (mega-boost):** `cross_country_aggregate_dlt.py` -> `health_private_expenditure_by_country_year` = CHE x OOP% / 100 (out-of-pocket in local currency), inflation-adjusted via CPI.
- **Upstream sources:**
  - BOOST, CPI
  - **Health expenditure** (`prd_mega.indicator.health_expenditure`): `mega-indicators/health/health_expenditure.py` -- WHO Global Health Expenditure Database (GHED) via GHO API, indicators: `GHED_OOPSCHE_SHA2011` (OOP % of CHE), `GHED_CHEGDP_SHA2011` (CHE % of GDP). CHE in LCU = `che_percent_gdp` x `gdp_current_lcu` (from GDP table).
  - **GDP** (`prd_mega.indicator.gdp`): `mega-indicators/gdp.py` -- World Bank & OECD National Accounts API, indicator `NY.GDP.MKTP.CN` (GDP current LCU).

### 17. Total Health Expenditure (`health-total`)
- **What it shows:** Health spending over time (central vs. regional), inflation-adjusted.
- **Source:** BOOST microdata (func=Health), CPI.

### 18. Health Spending vs. UHC Index (`health-outcome`)
- **What it shows:** Dual-axis chart of per capita real health spending vs. Universal Health Coverage index.
- **Dashboard query:** `prd_mega.indicator.universal_health_coverage_index_gho`
- **Processing (mega-indicators):** `health/sdg_health.py` -- World Bank Health Nutrition & Population Statistics (db=16), indicator `SH.UHC.SRVS.CV.XD` (SDG 3.8.1).
- **Source:** WHO Global Health Observatory.

### 19. Operational vs. Capital Spending -- Health (`econ-breakdown-func-health`)
- **What it shows:** Same as chart 12 but for health sector.
- **Source:** BOOST microdata (economic + functional classification), CPI.

---

## Health Page — "Across Space" Tab

### 20. Centrally vs. Geographically Allocated Health Spending (`health-central-vs-regional`, `health-sub-func`)
- **Same structure as chart 13** but filtered to func=Health.
- **Source:** BOOST microdata, CPI.

### 21. Health Expenditure Map & Health Outcome Map (`health-expenditure-map`, `health-outcome-map`)
- **What it shows:** Side-by-side choropleths -- per capita health spending vs. health index by region.
- **Processing:** Same as chart 14 but for Health: outcome = `health_index` from Global Data Lab.
- **Source:** BOOST, Global Data Lab HD Index, admin boundaries, subnational population.

### 22. Health Subnational Scatter/Rank (`health-subnational`)
- **Same structure as chart 15** but for Health.
- **Source:** Same as chart 21.

---

## Supporting Data Reference Table

| Table | Repo | Processing Script | Original Source |
|---|---|---|---|
| `boost_gold` | mega-boost | Per-country `*_transform_load_dlt.py` + `cross_country_aggregate_dlt.py` | World Bank BOOST database (country budget offices) |
| `consumer_price_index` | mega-indicators | `consumer_price_index.py` | World Bank API (`FP.CPI.TOTL`) |
| `population` | mega-indicators | `population/national_population.py` | UN, World Bank, Eurostat |
| `subnational_population` | mega-indicators | Per-country scripts in `population/` | National statistical offices, census |
| `country` | mega-indicators | `country.py` | World Bank API (`wbgapi`) |
| `admin1_boundaries_gold` | mega-indicators | `geo/admin_boundaries_dlt.py` | geoBoundaries / national sources |
| `poverty_rate` | mega-indicators | `poverty/poverty.py` | World Bank Poverty & Inequality Platform |
| `subnational_poverty_rate` | mega-indicators | `poverty/subnational_poverty/` | SPID & GSAP (World Bank DDH) |
| `pefa_by_pillar` | mega-indicators | `pefa/pefa_transform_load.py` | PEFA Secretariat |
| `global_data_lab_hd_index` | mega-indicators | `human_development/global_data_lab_hdi_extract.r` + `global_data_lab_hid_transform_load_dlt.py` | Global Data Lab (SHDI + Education datasets) |
| `learning_poverty_rate` | mega-indicators | `education/learning_poverty.py` | World Bank & UNESCO UIS |
| `edu_spending` | mega-indicators | `education/education_spending_icp.py` | International Comparison Program (ICP) |
| `health_expenditure` | mega-indicators | `health/health_expenditure.py` | WHO GHED via GHO API + GDP |
| `universal_health_coverage_index_gho` | mega-indicators | `health/sdg_health.py` | WHO Global Health Observatory |
| `gdp` | mega-indicators | `gdp.py` | World Bank & OECD National Accounts |
