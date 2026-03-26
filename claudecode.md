Here's the complete chart-by-chart data source summary:

---

## Data Sources: RPF Country Dashboard

### Architecture Overview

```
External Sources → mega-indicators → Databricks (prd_mega.indicator.*)
                                          ↓
External Sources → mega-boost ──────→ Databricks (prd_mega.boost.*)
                                          ↓
                                   rpf-country-dash (Dash/Plotly)
```

**mega-boost** harmonizes BOOST government expenditure microdata (Excel files from government authorities → bronze → silver → gold pipeline in Databricks).

**mega-indicators** fetches socioeconomic indicators from external APIs (World Bank, WHO, OECD, Global Data Lab, PEFA).

---

## Home Page — Over Time Tab

| # | Chart | Databricks Table | Source Repo | Original Source | Key Transformations |
|---|-------|-----------------|-------------|-----------------|---------------------|
| 1 | **Total Expenditure** | `boost.pov_expenditure_by_country_year` | mega-boost | BOOST Excel microdata from government authorities | Country ETL (Excel→CSV→bronze→silver→gold), cross-country aggregation, CPI adjustment for real values, central/decentralized split |
| 2 | **Per Capita Expenditure** | Same + poverty_rate | mega-boost + mega-indicators | BOOST + World Bank Poverty & Inequality Platform API | Same as above + poverty rates joined (WB API, thresholds vary by income level: $3.65/$6.85/$24.35) |
| 3 | **Functional Breakdown** | `boost.expenditure_by_country_func_econ_year` | mega-boost | BOOST Excel | Country-specific budget codes mapped to 10 COFOG functional categories, grouped by country/year/func |
| 4 | **Functional Category Growth** | Same table | mega-boost | BOOST Excel | Same as #3; YoY % change computed client-side in dashboard |
| 5 | **Economic Breakdown** | Same table | mega-boost | BOOST Excel | Country-specific codes mapped to 8 economic categories (Wage Bill, Capital, Goods/Services, Subsidies, Social Benefits, Interest, Grants, Other) |
| 6 | **PEFA Overall** | `indicator.pefa_by_pillar` | mega-indicators | PEFA Secretariat (manual batch download) | Letter grades (A+ to D) → numeric scores (4.5 to 1.0), averaged across 7 pillars, harmonized across 2011 & 2016 frameworks |
| 7 | **PEFA by Pillar** | Same table | mega-indicators | PEFA Secretariat | Same processing; displayed as heatmap of 7 individual pillar scores over time |

## Home Page — Across Space Tab

| # | Chart | Databricks Table(s) | Source Repo | Original Source | Key Transformations |
|---|-------|---------------------|-------------|-----------------|---------------------|
| 8 | **Subnational Spending Map** | `boost.expenditure_by_country_geo1_year` + `indicator.admin1_boundaries_gold` | mega-boost + mega-indicators | BOOST Excel + WB Official Boundaries GeoJSON | Expenditure aggregated by admin1 region; boundaries parsed from GeoJSON with 34+ country-specific name corrections |
| 9 | **Subnational Poverty Map** | `indicator.subnational_poverty_rate` + boundaries | mega-indicators | World Bank Poverty & Inequality Platform | Poverty rates by region joined with admin1 boundary geometries |

---

## Education Page — Over Time Tab

| # | Chart | Databricks Table | Source Repo | Original Source | Key Transformations |
|---|-------|-----------------|-------------|-----------------|---------------------|
| 10 | **Public vs Private Expenditure** | `boost.edu_private_expenditure_by_country_year` | mega-boost (joins mega-indicators) | ICP (International Comparison Program) via WB API (databases for 2005, 2011, 2017, 2021) | Private education spending from 3 ICP databases, converted from billion LCU to units, expressed as % of GDP |
| 11 | **Total Education Expenditure** | `boost.expenditure_by_country_func_econ_year` (func=Education) | mega-boost | BOOST Excel | Same pipeline as Total Expenditure, filtered to Education functional category |
| 12 | **Education Outcomes** | `indicator.learning_poverty_rate` + `indicator.global_data_lab_hd_index` | mega-indicators | World Bank API (learning poverty) + Global Data Lab API (school attendance ages 6-17) | Learning poverty via `wbgapi`; school attendance via Global Data Lab R API with auth token |
| 13 | **Economic Breakdown (Education)** | Same as #11 | mega-boost | BOOST Excel | Operational vs capital spending split for education |

## Education Page — Across Space Tab

| # | Chart | Databricks Table(s) | Source Repo | Original Source | Key Transformations |
|---|-------|---------------------|-------------|-----------------|---------------------|
| 14 | **Central vs Regional** | `boost.expenditure_by_country_geo0_func_sub_year` | mega-boost | BOOST Excel | Expenditure split by geo0 (Central/Regional) and education sub-functions |
| 15 | **Education Sub-Functions** | Same table | mega-boost | BOOST Excel | Treemap hierarchy of education sub-function spending |
| 16 | **Education Expenditure Map** | `boost.expenditure_by_country_geo1_year` + boundaries | mega-boost + mega-indicators | BOOST Excel + WB Boundaries | Filtered to education; per-capita or total by admin1 region |
| 17 | **Education Outcome Map** | `indicator.global_data_lab_hd_index` + boundaries | mega-indicators | Global Data Lab API | School attendance (ages 6-17) by subnational region |
| 18 | **Education Subnational Ranking** | `boost.expenditure_and_outcome_by_country_geo1_func_year` | mega-boost (joins mega-indicators) | BOOST + Global Data Lab | Sankey diagram linking per-capita spending rank to education outcome rank by region |

---

## Health Page — Over Time Tab

| # | Chart | Databricks Table | Source Repo | Original Source | Key Transformations |
|---|-------|-----------------|-------------|-----------------|---------------------|
| 19 | **Public vs Private Expenditure** | `boost.health_private_expenditure_by_country_year` | mega-boost (joins mega-indicators) | WHO Global Health Expenditure Database (via GHO API) | Out-of-pocket health spending fetched from WHO API, merged with GDP to calculate total current health expenditure |
| 20 | **Total Health Expenditure** | `boost.expenditure_by_country_func_econ_year` (func=Health) | mega-boost | BOOST Excel | Same pipeline as Total Expenditure, filtered to Health functional category |
| 21 | **Health Outcomes** | `indicator.universal_health_coverage_index_gho` | mega-indicators | WHO GHO API | UHC service coverage index fetched from WHO API endpoint |
| 22 | **Economic Breakdown (Health)** | Same as #20 | mega-boost | BOOST Excel | Operational vs capital spending split for health |

## Health Page — Across Space Tab

| # | Chart | Databricks Table(s) | Source Repo | Original Source | Key Transformations |
|---|-------|---------------------|-------------|-----------------|---------------------|
| 23 | **Central vs Regional** | `boost.expenditure_by_country_geo0_func_sub_year` | mega-boost | BOOST Excel | Same as #14 but filtered to health |
| 24 | **Health Sub-Functions** | Same table | mega-boost | BOOST Excel | Treemap of health sub-function spending |
| 25 | **Health Expenditure Map** | `boost.expenditure_by_country_geo1_year` + boundaries | mega-boost + mega-indicators | BOOST Excel + WB Boundaries | Filtered to health; per-capita or total by admin1 region |
| 26 | **Health Outcome Map** | `indicator.universal_health_coverage_index_gho` + boundaries | mega-indicators | WHO GHO API | UHC index mapped to subnational regions |
| 27 | **Health Subnational Ranking** | `boost.expenditure_and_outcome_by_country_geo1_func_year` | mega-boost (joins mega-indicators) | BOOST + WHO GHO | Sankey diagram linking per-capita spending rank to health outcome rank |

---

## Supporting Data Tables

| Table | Source Repo | Original Source | Purpose |
|-------|-------------|-----------------|---------|
| `indicator.country` | mega-indicators | World Bank API (`wbgapi`) | Country metadata: codes, names, regions, income levels, currencies, map coordinates |
| `indicator.admin1_boundaries_gold` | mega-indicators | WB Official Boundaries GeoJSON | Admin1 boundary polygons for all choropleth maps |
| `indicator.admin0_disputed_boundaries_gold` | mega-indicators | WB Official Boundaries GeoJSON | Disputed territory boundaries |
| `boost.cpi_factor` | mega-boost (uses mega-indicators CPI) | World Bank API (CPI indicator via `consumer_price_index.py`) | Inflation adjustment factors for converting nominal to real expenditure |
| `boost.data_availability` | mega-boost | Generated metadata | Tracks BOOST data coverage by country/year, links to WB Data Catalog |
| `indicator.indicator_data_availability` | mega-indicators | Generated metadata | Tracks indicator availability with source URLs per country |

---

## External Source Summary

| Source | Indicators Used | Access Method |
|--------|----------------|---------------|
| **BOOST Microdata** (government authorities) | All expenditure data (functional, economic, subnational) | Excel files, country-specific ETL |
| **World Bank API** | GDP, population, poverty rates, CPI, learning poverty, education spending | Python `wbgapi` library |
| **WHO GHO API** | Health expenditure (CHE, OOP), UHC index | REST API (`ghoapi.azureedge.net`) |
| **Global Data Lab API** | HDI components, school attendance (ages 6-17), subnational population | R package with auth token |
| **OECD SDMX API** | Private education spending | REST API |
| **ICP (WB)** | Education spending in PPP terms | WB API (databases 71, 62, 90) |
| **PEFA Secretariat** | PFM assessment scores (7 pillars, 2011 & 2016 frameworks) | Manual batch download |
| **WB Official Boundaries** | Admin0 and Admin1 geographic boundaries | GeoJSON files |

The background planning agent also completed with additional detail. The summary I posted above covers all 27 charts — let me know if you'd like me to dive deeper into any specific chart's pipeline or if you'd like this written into a `DATA_SOURCES.md` file in the repo.