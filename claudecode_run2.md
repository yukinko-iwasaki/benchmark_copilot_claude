
## Summary: Data Sources & Processing by Chart

Here's a chart-by-chart breakdown of every visualization in the RPF Country Dashboard, tracing data from original source through processing to display.

---

### HOME PAGE — "Over Time" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 1 | **How has total expenditure changed over time?** (stacked bar + line) | `pov_expenditure_by_country_year` | **BOOST** microdata (Excel) + **World Bank Poverty & Inequality Platform** | `mega-boost`: Country Excel → bronze CSV → silver (COFOG/economic classification) → gold (`boost_gold`) → `expenditure_by_country_year` aggregation. Joined with poverty rate from `mega-indicators` (`poverty_rate` table via wbgapi `SI.POV.*`). CPI-adjusted to real values using `consumer_price_index`. |
| 2 | **How has per capita expenditure changed over time?** (stacked bar + line) | `pov_expenditure_by_country_year` | **BOOST** + **World Bank Poverty Platform** + **World Bank Population** | Same as above. Per capita = total expenditure ÷ population. Population from `mega-indicators` (`population` table via wbgapi `SP.POP.TOTL`). Poverty rate overlaid on secondary axis. |
| 3 | **How has sector prioritization changed over time?** (stacked bar) | `expenditure_by_country_func_econ_year` | **BOOST** microdata | `mega-boost`: Raw budget data classified into 10 COFOG functional categories (Education, Health, Defence, Social Protection, etc.). Aggregated by country × function × year. Dashboard computes proportional shares. |
| 4 | **How do budgets for functional categories fluctuate over time?** (multi-line) | `expenditure_by_country_func_econ_year` | **BOOST** microdata | Same source as above. Dashboard computes year-on-year growth rates and CAGR (Compound Annual Growth Rate) per functional category. Top 4 categories shown by default. |
| 5 | **How much was spent on each economic category?** (stacked bar) | `expenditure_by_country_func_econ_year` | **BOOST** microdata | `mega-boost`: Raw budget data classified into economic categories (Wage bill, Capital expenditures, Goods & services, Subsidies & transfers, Interest on debt). Aggregated by country × economic category × year. |
| 6 | **How did the overall quality of budget institutions change over time?** (line + secondary axis) | `pefa_by_pillar` + poverty rate | **PEFA Secretariat** + **World Bank Poverty Platform** | `mega-indicators`: PEFA assessment PDFs batch-downloaded from pefa.org. Letter grades (A–D) mapped to numeric scores (4.0–1.0). Averaged across 7 pillars for overall score. Poverty rate overlay from wbgapi. |
| 7 | **How did various pillars of the budget institutions change over time?** (heatmap) | `pefa_by_pillar` | **PEFA Secretariat** | `mega-indicators`: Same PEFA source. 7 pillars displayed: (1) Budget Reliability, (2) Transparency, (3) Asset/Liability Management, (4) Policy-Based Strategy, (5) Predictability & Control, (6) Accounting & Reporting, (7) External Audit. Two frameworks (2011, 2016) handled. |

---

### HOME PAGE — "Across Space" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 8 | **How much was spent in each region?** (choropleth map) | `expenditure_by_country_geo1_year` + `admin1_boundaries_gold` | **BOOST** microdata + **World Bank Official Boundaries** | `mega-boost`: Expenditure tagged with `geo1` (subnational region) during harmonization. Aggregated by country × region × year. `mega-indicators`: GeoJSON boundaries loaded, admin1 names harmonized across countries (extensive custom mappings for Ghana, Colombia, DRC, etc.). Joined by region name for map rendering. |
| 9 | **What percent of the population is living in poverty?** (choropleth map) | `subnational_poverty_rate` + `admin1_boundaries_gold` | **World Bank SPID/GSAP** + **World Bank Official Boundaries** | `mega-indicators`: Subnational poverty rates extracted from SPID and GSAP via World Bank DDH API. Income-level-specific thresholds applied ($3/day for LIC, $4.20 for LMC, $8.30 for UMC/HIC). Region names harmonized to match admin boundaries. |

---

### EDUCATION PAGE — "Over Time" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 10 | **What % was spent by govt vs household?** (horizontal stacked bar) | `edu_private_expenditure_by_country_year` | **International Comparison Program (ICP)** + **OECD** + **BOOST** | `mega-indicators`: ICP data from wbgapi databases (71, 62, 90) for 2005/2011/2017/2021. OECD education spending via SDMX API. Private share = ICP spending ÷ (ICP + govt spending). `mega-boost`: Public education spending from BOOST functional classification. Combined in `cross_country_aggregate_dlt.py`. |
| 11 | **How has govt spending on education changed over time?** (stacked bar + line) | `expenditure_by_country_func_econ_year` (filtered to Education) | **BOOST** microdata | `mega-boost`: BOOST data filtered to `func = 'Education'`. Split by `admin0` (Central vs Regional). Decentralization ratio computed. |
| 12 | **How has education outcome changed?** (dual-axis line) | `global_data_lab_hd_index` + `learning_poverty_rate` + education expenditure | **Global Data Lab (UNDP)** + **World Bank** + **BOOST** | `mega-indicators`: (1) School attendance rate (6–17 year-olds) from Global Data Lab via R `gdldata` package → HDI dataset. (2) Learning poverty rate from wbgapi (`SE.LPV.PRIM`). Dashboard overlays per capita education spending from BOOST. |
| 13 | **Operational vs Capital Spending** (education) (stacked bar) | `expenditure_by_country_func_econ_year` (filtered) | **BOOST** microdata | `mega-boost`: BOOST data filtered to `func = 'Education'`, grouped by economic categories (Wage bill, Capital, Goods & services, etc.). |

---

### EDUCATION PAGE — "Across Space" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 14 | **Where was education spending directed?** (donut chart) | `expenditure_by_country_func_econ_year` | **BOOST** microdata | `mega-boost`: Education expenditure split by `geo0` (Central vs Regional allocation). |
| 15 | **How much did the govt spend on different levels of education?** (treemap) | `expenditure_by_country_geo0_func_sub_year` | **BOOST** microdata | `mega-boost`: BOOST data with `func_sub` dimension (Pre-primary, Primary, Secondary, Tertiary, etc.). Aggregated by sub-function. |
| 16 | **Subnational Education Spending** (choropleth map) | `expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **BOOST** + **World Bank Boundaries** | `mega-boost`: Education expenditure by `geo1` region. Per capita computed using subnational population from `mega-indicators` (Global Data Lab or US Census). Joined with GeoJSON boundaries. |
| 17 | **Subnational School Attendance Rate** (choropleth map) | `expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **Global Data Lab (UNDP)** + **World Bank Boundaries** | `mega-indicators`: School attendance (6–17 year-olds) from Global Data Lab HDI dataset. Region names harmonized. `mega-boost`: Joined with expenditure data in `cross_country_aggregate_dlt.py`. |
| 18 | **Spending vs Outcomes Across Regions** (Sankey diagram) | `expenditure_and_outcome_by_country_geo1_func_year` | **BOOST** + **Global Data Lab** | `mega-boost`: Regions ranked by per capita education spending and by attendance rate. Sankey links ranks to show whether high-spending regions have better outcomes. Spearman correlation computed in dashboard. |

---

### HEALTH PAGE — "Over Time" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 19 | **What % was spent by govt vs household?** (horizontal stacked bar) | `health_private_expenditure_by_country_year` | **WHO Global Health Expenditure Database** + **BOOST** | `mega-indicators`: Out-of-pocket (OOP) and current health expenditure (CHE) from WHO GHO API (`https://ghoapi.azureedge.net/api/`). `mega-boost`: Public health spending from BOOST. Combined in aggregation pipeline. |
| 20 | **How has govt spending on health changed over time?** (stacked bar + line) | `expenditure_by_country_func_econ_year` (filtered to Health) | **BOOST** microdata | `mega-boost`: BOOST data filtered to `func = 'Health'`. Split by Central vs Regional. |
| 21 | **How has health outcome changed?** (dual-axis line) | `universal_health_coverage_index_gho` + health expenditure | **WHO Global Health Observatory** + **BOOST** | `mega-indicators`: UHC Service Coverage Index from WHO GHO API. Dashboard overlays per capita health spending from BOOST. |
| 22 | **Operational vs Capital Spending** (health) (stacked bar) | `expenditure_by_country_func_econ_year` (filtered) | **BOOST** microdata | `mega-boost`: BOOST data filtered to `func = 'Health'`, grouped by economic categories. |

---

### HEALTH PAGE — "Across Space" Tab

| # | Chart | Data Table | Source | Processing Pipeline |
|---|-------|-----------|--------|---------------------|
| 23 | **Where was health spending directed?** (donut chart) | `expenditure_by_country_func_econ_year` | **BOOST** microdata | `mega-boost`: Health expenditure split by Central vs Regional. |
| 24 | **How much did the govt spend on different health areas?** (treemap) | `expenditure_by_country_geo0_func_sub_year` | **BOOST** microdata | `mega-boost`: BOOST data with `func_sub` for health (Hospital services, Public health, Outpatient, etc.). |
| 25 | **Subnational Health Spending** (choropleth map) | `expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **BOOST** + **World Bank Boundaries** | `mega-boost`: Health expenditure by region, per capita using subnational population. |
| 26 | **Subnational UHC Index** (choropleth map) | `expenditure_and_outcome_by_country_geo1_func_year` + boundaries | **WHO GHO** + **World Bank Boundaries** | `mega-indicators`: UHC index from WHO. Note: this is typically national-level; subnational may use health index from Global Data Lab HDI dataset. |
| 27 | **Spending vs Outcomes Across Regions** (Sankey diagram) | `expenditure_and_outcome_by_country_geo1_func_year` | **BOOST** + **WHO/Global Data Lab** | Same pattern as education Sankey: regions ranked by health spending and health outcome, with Spearman correlation. |

---

### Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     EXTERNAL DATA SOURCES                          │
├──────────────┬──────────────┬───────────────┬───────────────────────┤
│ BOOST Excel  │ World Bank   │ WHO GHO API   │ Other Sources         │
│ (18 countries│ APIs (wbgapi)│ (Health data) │ - PEFA Secretariat    │
│  budget      │ - GDP        │ - OOP %       │ - Global Data Lab     │
│  microdata)  │ - Population │ - CHE         │ - ICP/OECD            │
│              │ - Poverty    │ - UHC Index   │ - US Census           │
│              │ - CPI        │               │ - Ember Energy        │
│              │ - Education  │               │ - WB Boundaries       │
└──────┬───────┴──────┬───────┴───────┬───────┴───────────┬───────────┘
       │              │               │                   │
       ▼              ▼               ▼                   ▼
┌──────────────┐ ┌─────────────────────────────────────────────────┐
│  mega-boost  │ │              mega-indicators                    │
│              │ │                                                 │
│ Excel→CSV    │ │ API calls → Bronze → Silver → Gold tables:     │
│ →Bronze      │ │  - population, gdp, cpi, poverty_rate          │
│ →Silver      │ │  - subnational_poverty_rate                    │
│ →Gold        │ │  - pefa_by_pillar                              │
│ (COFOG/econ  │ │  - learning_poverty_rate                       │
│  harmonize)  │ │  - global_data_lab_hd_index                    │
│              │ │  - universal_health_coverage_index_gho          │
│ boost_gold   │ │  - edu_private_spending, health_expenditure    │
│      │       │ │  - admin1_boundaries_gold                      │
│      ▼       │ │  - indicator_data_availability                 │
│ Aggregation  │ └─────────────────┬───────────────────────────────┘
│ DLT pipeline │                   │
│ (joins with  │◄──────────────────┘
│  indicators) │
│      │       │
│      ▼       │
│ 7 aggregate  │
│ tables       │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────────────────────────────────────┐
│                    rpf-country-dash                              │
│  Dash/Plotly app reading from Databricks SQL                    │
│  27 charts across Home, Education, Health pages                 │
└──────────────────────────────────────────────────────────────────┘
```

---

### Source Legend

| Source | Organization | Access Method |
|--------|-------------|---------------|
| **BOOST** | World Bank DIME | Country Excel files → mega-boost ETL |
| **World Bank Open Data** | World Bank | wbgapi Python package |
| **WHO GHO** | World Health Organization | REST API (`ghoapi.azureedge.net`) |
| **PEFA** | PEFA Secretariat | Batch PDF download from pefa.org |
| **Global Data Lab** | UNDP / Radboud University | R `gdldata` package |
| **ICP** | World Bank | wbgapi databases 71, 62, 90 |
| **OECD** | OECD | SDMX REST API |
| **SPID/GSAP** | World Bank | DDH API download |
| **WB Boundaries** | World Bank | GeoJSON files |
| **US Census** | US Census Bureau | International population data |
| **CPI** | World Bank | wbgapi → `consumer_price_index` table |

Would you like me to commit this as a documentation file in one or more of the repos?
