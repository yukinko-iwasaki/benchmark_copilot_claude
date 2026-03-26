# RPF Country Dashboard — Data Source & Processing Summary

## Overview (Home) Page — "Over Time" tab

### 1. Total Expenditure (bar + line chart)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.boost.pov_expenditure_by_country_year` |
| **mega-boost processing** | `cross_country_aggregate_dlt.py` → `boost_gold` (union of per-country gold tables) → `expenditure_by_country_year` (aggregates executed/approved by country+year, joins CPI factor & population) → `pov_expenditure_by_country_year` (left-joins poverty rate) |
| **Original source: expenditure** | **BOOST Database** (World Bank) — country budget microdata Excel files ingested per-country via `{CC}_extract_microdata_excel_to_csv.py` → `{CC}_transform_load_dlt.py` in each country folder of **mega-boost** |
| **Original source: CPI** | **World Bank API** (`FP.CPI.TOTL`) — fetched in `mega-indicators/consumer_price_index.py` → `prd_mega.indicator.consumer_price_index` |
| **Fields shown** | Central expenditure (bar), Regional/decentralized expenditure (bar), Inflation-adjusted expenditure (line) |

### 2. Per Capita Expenditure + Poverty Rate (dual-axis chart)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `pov_expenditure_by_country_year` table |
| **Additional source: Population** | **UN/World Bank/Eurostat** (`SP.POP.TOTL`) — fetched in `mega-indicators/population/national_population.py` → `prd_mega.indicator.population` |
| **Additional source: Poverty rate** | **World Bank Poverty & Inequality Platform** (`SI.POV.DDAY`, `SI.POV.LMIC`, `SI.POV.UMIC`) — fetched in `mega-indicators/poverty/poverty.py` → `prd_mega.indicator.poverty_rate`. Income-level-specific thresholds ($3.00 LIC, $4.20 LMC, $8.30 UMC/HIC) |
| **Fields shown** | Per capita expenditure (bar), Inflation-adjusted per capita (line), Poverty rate (dotted line, secondary axis) |

### 3. Spending by Functional Categories (stacked bar %)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.boost.expenditure_by_country_func_econ_year` → aggregated client-side to func-year |
| **mega-boost processing** | `boost_gold` → `expenditure_by_country_admin_func_sub_econ_sub_year` → `expenditure_by_country_func_econ_year` (joins population) → grouped by func in the dashboard's `fetch_func_data_once` callback |
| **Original source** | **BOOST Database** (functional COFOG classification). 10 categories: Social protection, Education, Health, etc. |

### 4. Budget Growth by Functional Category (line chart, YoY %)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `expenditure_by_country_func_econ_year`, filtered to `domestic_funded_budget` or `real_domestic_funded_budget` |
| **Processing** | Component `budget_increment_analysis.py` computes year-on-year % change and CAGR from the `domestic_funded_budget` column |
| **Original source** | **BOOST Database** — `approved` amounts, with foreign-funded budgets excluded |

### 5. Spending by Economic Categories (stacked bar %)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `expenditure_by_country_func_econ_year` → aggregated client-side to econ-year |
| **Original source** | **BOOST Database** (economic classification: Wage bill, Capital expenditures, Goods & services, etc.) |

### 6. PEFA Overall Score + Poverty Rate (dual-axis)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.indicator.pefa_by_pillar` |
| **mega-indicators processing** | `pefa/pefa_transform_load.py` — reads manually uploaded bronze tables (`pefa_2011_bronze`, `pefa_2016_bronze`), maps letter grades to numeric scores, averages per-pillar indicators, writes `prd_mega.indicator.pefa_by_pillar` |
| **Original source** | **PEFA Secretariat** (pefa.org batch downloads) — 2011 & 2016 frameworks |
| **Fields shown** | Average pillar score (line), Poverty rate (dotted line, secondary axis) |

### 7. PEFA Pillar Heatmap

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `pefa_by_pillar` table |
| **Fields shown** | 7 pillars (Budget reliability, Transparency, Asset/Liability, Policy-based budgeting, Predictability/control, Accounting/reporting, External audit) × assessment years, color-coded by score |

---

## Overview (Home) Page — "Across Space" tab

### 8. Regional Spending Choropleth (per capita or total)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.boost.expenditure_by_country_geo1_year` |
| **mega-boost processing** | `boost_gold` → `expenditure_by_country_geo1_func_year` (joins subnational population) → `expenditure_by_country_geo1_year` (aggregated across funcs) |
| **Additional source: Subnational population** | Country-specific scripts in `mega-indicators/population/{CC}/` → `prd_mega.indicator.subnational_population`. Sources vary by country (national statistics offices, census.gov, Global Data Lab) |
| **Additional source: Admin boundaries** | `mega-indicators/geo/admin_boundaries_dlt.py` → `prd_mega.indicator.admin1_boundaries_gold`. Derived from **geoBoundaries** dataset |
| **Additional source: Disputed boundaries** | `prd_mega.indicator.admin0_disputed_boundaries_gold` from same geo pipeline |

### 9. Subnational Poverty Rate Choropleth

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.indicator.subnational_poverty_rate` |
| **mega-indicators processing** | `poverty/subnational_poverty/subnational_poverty_index_extract_transform.py` — fetches from **SPID** (World Bank DDH API `DR0092191`) and **GSAP** (`DR0052555`), combines with GSAP taking precedence for overlap years → `subnational_poverty_index_transform_load_dlt.py` → `subnational_poverty_rate` |
| **Original source** | **World Bank SPID & GSAP** (Subnational Poverty & Inequality Database + Global Subnational Atlas of Poverty) |

---

## Education Page — "Over Time" tab

### 10. Who Pays for Education? (Public vs. Private bar charts)

| Aspect | Detail |
|---|---|
| **Dashboard queries** | Public: from `expenditure_by_country_func_year` (func="Education"). Private: `prd_mega.boost.edu_private_expenditure_by_country_year` |
| **mega-boost processing (private)** | `cross_country_aggregate_dlt.py` → `edu_private_expenditure_by_country_year` = ICP total education spending minus BOOST public education expenditure, CPI-adjusted |
| **Original source: Public** | **BOOST Database** |
| **Original source: Private (total edu spending)** | **International Comparison Program (ICP)** — World Bank databases 71, 62, 90 — fetched in `mega-indicators/education/education_spending_icp.py` → `prd_mega.indicator.edu_spending` |

### 11. Public Spending & Education Outcome (dual-axis)

| Aspect | Detail |
|---|---|
| **Dashboard queries** | Public spending: same as above. Outcome: `prd_mega.indicator.learning_poverty_rate` |
| **mega-indicators processing** | `education/learning_poverty.py` — fetches `SE.LPV.PRIM` from **World Bank Education Stats** (db=12) → `learning_poverty_rate` |
| **Original source** | **World Bank & UNESCO Institute for Statistics (UIS)** — Learning Poverty indicator |

### 12. Operational vs. Capital Spending — Education (stacked area %)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `expenditure_by_country_func_econ_year` filtered to func="Education", then grouped by econ category into: Wage bill, Capital expenditures, Non-wage recurrent |
| **Original source** | **BOOST Database** (economic × functional cross-classification) |

---

## Education Page — "Across Space" tab

### 13. Centrally vs. Geographically Allocated Education Spending (pie + treemap)

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.boost.expenditure_by_country_geo0_func_sub_year` (func="Education") |
| **Processing** | Pie chart shows Central vs Regional split of geo-tagged spending; Treemap breaks down by `func_sub` (Primary, Secondary, Tertiary education, etc.) × geo0 |
| **Original source** | **BOOST Database** (geo + sub-functional classification) |

### 14. Subnational Education Spending Map + Education Outcomes Map

| Aspect | Detail |
|---|---|
| **Dashboard query** | `prd_mega.boost.expenditure_and_outcome_by_country_geo1_func_year` |
| **mega-boost processing** | `expenditure_by_country_geo1_func_year` left-joined with `global_data_lab_hd_index` on (country, adm1_name, year). Education outcome = `attendance_6to17yo` (school attendance rate for ages 6-17) |
| **Original source: Spending** | **BOOST Database** |
| **Original source: Education outcome** | **Global Data Lab** (Subnational HDI) — extracted in `mega-indicators/human_development/global_data_lab_hdi_extract.r`, transformed in `mega-indicators/human_development/global_data_lab_hid_transform_load_dlt.py` → `prd_mega.indicator.global_data_lab_hd_index`. Uses `attendance` field from the GDL education dataset |

---

## Health Page — "Over Time" tab

### 15. Who Pays for Healthcare? (Public vs. Private bar charts)

| Aspect | Detail |
|---|---|
| **Dashboard queries** | Public: `expenditure_by_country_func_year` (func="Health"). Private: `prd_mega.boost.health_private_expenditure_by_country_year` |
| **mega-boost processing (private)** | `cross_country_aggregate_dlt.py` → `health_private_expenditure_by_country_year` = CHE (Current Health Expenditure in LCU) × OOP% / 100, CPI-adjusted |
| **Original source: Public** | **BOOST Database** |
| **Original source: Private** | **WHO Global Health Expenditure Database** (GHO API: `GHED_OOPSCHE_SHA2011`, `GHED_CHEGDP_SHA2011`) — fetched in `mega-indicators/health/health_expenditure.py` → `prd_mega.indicator.health_expenditure`. CHE in LCU derived from CHE%GDP × GDP. GDP from `mega-indicators/gdp.py` |

### 16. Public Spending & Health Outcome (dual-axis)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Outcome: `prd_mega.indicator.universal_health_coverage_index_gho` |
| **mega-indicators processing** | `health/sdg_health.py` — fetches `SH.UHC.SRVS.CV.XD` from **World Bank Health Nutrition & Population Stats** (db=16) |
| **Original source** | **WHO Global Health Observatory** — Universal Health Coverage (UHC) Service Coverage Index (SDG 3.8.1) |

### 17. Operational vs. Capital Spending — Health (stacked area %)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same as Education counterpart but func="Health" |
| **Original source** | **BOOST Database** |

---

## Health Page — "Across Space" tab

### 18. Centrally vs. Geographically Allocated Health Spending (pie + treemap)

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `expenditure_by_country_geo0_func_sub_year` (func="Health") |
| **Original source** | **BOOST Database** |

### 19. Subnational Health Spending Map + Health Outcomes Map

| Aspect | Detail |
|---|---|
| **Dashboard query** | Same `expenditure_and_outcome_by_country_geo1_func_year` table. Health outcome = `health_index` from GDL |
| **Original source: Spending** | **BOOST Database** |
| **Original source: Health outcome** | **Global Data Lab** (Subnational HDI — `healthindex`) |

---

## Cross-cutting data used across all maps

| Dataset | Table | Source | mega-indicators script |
|---|---|---|---|
| Country metadata | `prd_mega.indicator.country` | **World Bank API** (`wbgapi`) + currency from `prd_corpdata` | `country.py` |
| Admin1 boundaries | `prd_mega.indicator.admin1_boundaries_gold` | **geoBoundaries** | `geo/admin_boundaries_dlt.py` |
| Disputed boundaries | `prd_mega.indicator.admin0_disputed_boundaries_gold` | Same pipeline | Same |
| CPI (inflation adj.) | `prd_mega.indicator.consumer_price_index` | **World Bank API** (`FP.CPI.TOTL`) | `consumer_price_index.py` |
| National population | `prd_mega.indicator.population` | **UN/WB/Eurostat** (`SP.POP.TOTL`) | `population/national_population.py` |
| Subnational population | `prd_mega.indicator.subnational_population` | Varies by country | `population/{CC}/` (per-country scripts) |

---

## Summary of original data sources

| Source | Used for |
|---|---|
| **BOOST Database** (World Bank) | All expenditure/budget data (functional, economic, geographic breakdowns) |
| **World Bank Poverty & Inequality Platform** | National poverty rates |
| **World Bank SPID + GSAP** | Subnational poverty rates |
| **World Bank CPI API** | Inflation adjustment (real expenditure) |
| **UN/WB/Eurostat Population** | Per capita calculations (national) |
| **Country statistics offices / Census** | Subnational population |
| **International Comparison Program (ICP)** | Total education spending (for private edu expenditure calc) |
| **WHO Global Health Expenditure Database** | Health expenditure & OOP spending (for private health expenditure calc) |
| **WHO Global Health Observatory** | UHC Index (health outcome) |
| **World Bank/UNESCO UIS** | Learning Poverty Rate (education outcome) |
| **Global Data Lab** | Subnational HDI (health index, education index, school attendance) |
| **PEFA Secretariat** | Budget institution quality scores |
| **geoBoundaries** | Admin boundary GeoJSON for map rendering |
| **World Bank API (wbgapi)** | Country metadata, income levels, currencies |
