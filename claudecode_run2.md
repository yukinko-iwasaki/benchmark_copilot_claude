Here's the complete chart-by-chart summary:

---

# Dashboard Data Sources: Chart-by-Chart Summary

The dashboard (`rpf-country-dash`) reads all data from Databricks (`prd_mega`) under two schemas:
- **`boost`** — Expenditure data, processed by **`mega-boost`**
- **`indicator`** — External indicators, fetched by **`mega-indicators`**

---

## Overview Page — "Over Time" Tab

### 1. Total Expenditure Over Time
**"How has total expenditure changed over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | `boost.pov_expenditure_by_country_year` |
| **Processing (mega-boost)** | Per-country BOOST Excel → `{code}_boost_gold` → `boost_gold` (union all countries) → `expenditure_by_country_year` (group by country+year, join CPI + population) → `pov_expenditure_by_country_year` (join poverty rate) |
| **Primary source** | **BOOST** public finance microdata (Ministries of Finance Excel workbooks) |
| **Supporting sources** | **CPI** — WB API `FP.CPI.TOTL` (`mega-indicators/consumer_price_index.py`); **Population** — WB API `SP.POP.TOTL` (`mega-indicators/population/national_population.py`) |
| **Displayed** | `expenditure` (Central vs Regional stacked bars), `real_expenditure` (inflation-adjusted line) |

### 2. Per Capita Expenditure + Poverty Rate
**"How has per capita expenditure changed over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | Same: `boost.pov_expenditure_by_country_year` |
| **Processing** | `per_capita_expenditure = expenditure / population`; `per_capita_real_expenditure = real_expenditure / population` |
| **Sources** | Same as #1 plus **Poverty Rate** — WB Poverty & Inequality Platform (`SI.POV.DDAY`, `SI.POV.LMIC`, `SI.POV.UMIC`) via `mega-indicators/poverty/poverty.py`; threshold varies by income level |
| **Displayed** | `per_capita_expenditure` (bar), `per_capita_real_expenditure` (line), `poverty_rate` (secondary axis) |

### 3. Functional Breakdown
**"How has sector prioritization changed over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_func_econ_year` (aggregated to func level client-side) |
| **Processing** | `boost_gold` → `expenditure_by_country_admin_func_sub_econ_sub_year` → `expenditure_by_country_func_econ_year` (join population) |
| **Primary source** | **BOOST** — `func` column mapped to COFOG categories |
| **Displayed** | `expenditure` per func as % of total per year |

### 4. Budget Growth Rate by Function
**Year-on-year budget growth rate**
| Layer | Detail |
|---|---|
| **DB Table** | Same as #3 |
| **Processing** | `domestic_funded_budget` = non-foreign-funded `approved`; inflation-adjusted variant uses CPI |
| **Primary source** | **BOOST** — `approved` column + `is_foreign` flag |
| **Displayed** | YoY % change of `domestic_funded_budget` or `real_domestic_funded_budget` |

### 5. Economic Breakdown
**"How much was spent on each economic category?"**
| Layer | Detail |
|---|---|
| **DB Table** | Same: `boost.expenditure_by_country_func_econ_year` (aggregated to econ level) |
| **Primary source** | **BOOST** — `econ` column mapped to GFS economic categories |
| **Displayed** | `expenditure` per econ as % of total per year |

### 6. PEFA Overall Score
**"How did the overall quality of budget institutions change over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | `indicator.pefa_by_pillar` |
| **Processing (mega-indicators)** | `pefa/pefa_transform_load.py` — downloads and transforms PEFA assessment data |
| **Primary source** | **PEFA Secretariat** — [pefa.org/assessments/batch-downloads](https://www.pefa.org/assessments/batch-downloads) |

### 7. PEFA by Pillar Heatmap
**"How did various pillars of the budget institutions change over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | Same: `indicator.pefa_by_pillar` |
| **Primary source** | **PEFA** assessments |

---

## Overview Page — "Across Space" Tab

### 8. Subnational Spending Map (Per Capita or Total)
**"How much was spent per person in each region?" / "How much was spent in each region?"**
| Layer | Detail |
|---|---|
| **DB Tables** | `boost.expenditure_by_country_geo1_year` + `indicator.admin1_boundaries_gold` + `indicator.admin0_disputed_boundaries_gold` |
| **Processing** | `boost_gold` → `expenditure_by_country_geo1_func_year` (join subnational population + CPI) → `expenditure_by_country_geo1_year` |
| **Primary source** | **BOOST** — `geo1` column (subnational geographic allocation) |
| **Supporting sources** | **Subnational Population** — national statistics offices or census.gov, per-country scripts in `mega-indicators/population/{CODE}/`; **Admin Boundaries** — geoBoundaries project via `mega-indicators/geo/admin_boundaries_dlt.py` |

### 9. Subnational Poverty Map
**"What percent of the population is living in poverty?"**
| Layer | Detail |
|---|---|
| **DB Table** | `indicator.subnational_poverty_rate` |
| **Processing** | `mega-indicators/poverty/subnational_poverty/` — country-specific scripts |
| **Primary source** | **World Bank PIPMaps** — [pipmaps.worldbank.org](https://pipmaps.worldbank.org); threshold: $3.00 LIC, $4.20 LMC, $8.30 UMC/HIC |

---

## Education Page — "Over Time" Tab

### 10. Public vs. Private Education Spending
**"What % was spent by the govt vs household?"**
| Layer | Detail |
|---|---|
| **DB Tables** | Public: `boost.expenditure_by_country_func_year` (Education); Private: `boost.edu_private_expenditure_by_country_year` |
| **Processing** | Private edu = total edu spending (ICP) minus BOOST public edu, CPI-adjusted |
| **Source (public)** | **BOOST** (func = "Education") |
| **Source (private)** | **WB International Comparison Program / UNESCO UIS** — `mega-indicators/education/education_spending_icp.py` → `indicator.edu_spending` |

### 11. Government Education Spending Over Time
**"How has govt spending on education changed over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_func_year` filtered to Education |
| **Primary source** | **BOOST** |
| **Supporting** | **CPI**, **Population** |

### 12. Education Outcome
**"How has education outcome changed?"**
| Layer | Detail |
|---|---|
| **DB Tables** | `indicator.global_data_lab_hd_index` (attendance) + `indicator.learning_poverty_rate` + education expenditure |
| **Source (attendance)** | **Global Data Lab** — [globaldatalab.org/shdi/](https://globaldatalab.org/shdi/about/) — via `mega-indicators/human_development/global_data_lab_hdi_extract.r` |
| **Source (learning poverty)** | **WB Data360** indicator `WB_LPGD_SE_LPV_PRIM_SD` — via `mega-indicators/education/learning_poverty.py` |
| **Displayed** | `per_capita_real_expenditure`, `attendance_6to17yo`, `learning_poverty_rate` |

### 13. Operational vs. Capital Education Spending
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_func_econ_year` filtered to Education |
| **Primary source** | **BOOST** — `func` + `econ` columns |

---

## Education Page — "Across Space" Tab

### 14. Central vs. Regional Education Spending + Sub-function Breakdown
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_geo0_func_sub_year` filtered to Education |
| **Primary source** | **BOOST** — `geo0`, `func_sub` (e.g., Primary Education, Secondary Education) |

### 15. Education Expenditure Map + Outcome Map
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_and_outcome_by_country_geo1_func_year` (Education) |
| **Source (expenditure)** | **BOOST** geo1-level |
| **Source (outcome)** | **Global Data Lab** — subnational `attendance_6to17yo` |

### 16. Education Subnational Rank Chart
| Layer | Detail |
|---|---|
| **DB Table** | Same as #15 — `rank_per_capita_real_exp` vs `rank_outcome_index` |

---

## Health Page — "Over Time" Tab

### 17. Public vs. Private Health Spending
**"What % was spent by the govt vs household?"**
| Layer | Detail |
|---|---|
| **DB Tables** | Public: `boost.expenditure_by_country_func_year` (Health); Private: `boost.health_private_expenditure_by_country_year` |
| **Processing** | OOP expenditure = CHE * OOP% / 100, CPI-adjusted |
| **Source (public)** | **BOOST** (func = "Health") |
| **Source (private)** | **WHO GHO** — CHE + OOP indicators — via `mega-indicators/health/health_expenditure.py` → `indicator.health_expenditure` |

### 18. Government Health Spending Over Time
**"How has govt spending on health changed over time?"**
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_func_year` filtered to Health |
| **Primary source** | **BOOST** + **CPI** + **Population** |

### 19. Health Outcome
**"How has health outcome changed?"**
| Layer | Detail |
|---|---|
| **DB Table** | `indicator.universal_health_coverage_index_gho` + health expenditure |
| **Source** | **WHO GHO** — [UHC Index of Service Coverage](https://www.who.int/data/gho/data/indicators/indicator-details/GHO/uhc-index-of-service-coverage) — via `mega-indicators/health/sdg_health.py` |

### 20. Operational vs. Capital Health Spending
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_func_econ_year` filtered to Health |
| **Primary source** | **BOOST** |

---

## Health Page — "Across Space" Tab

### 21. Central vs. Regional Health Spending + Sub-function Breakdown
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_by_country_geo0_func_sub_year` filtered to Health |
| **Primary source** | **BOOST** — `geo0`, `func_sub` |

### 22. Health Expenditure Map + Outcome Map
| Layer | Detail |
|---|---|
| **DB Table** | `boost.expenditure_and_outcome_by_country_geo1_func_year` (Health) |
| **Source (expenditure)** | **BOOST** geo1-level |
| **Source (outcome)** | **Global Data Lab** — subnational `health_index` |

### 23. Health Subnational Rank Chart
| Layer | Detail |
|---|---|
| **DB Table** | Same as #22 |

---

## Supporting Data: Complete Source Reference

| Databricks Table | External Source | mega-indicators Script |
|---|---|---|
| `indicator.consumer_price_index` | World Bank API (`FP.CPI.TOTL`) | `consumer_price_index.py` |
| `indicator.population` | World Bank API (`SP.POP.TOTL`) | `population/national_population.py` |
| `indicator.subnational_population` | National statistics offices / census.gov | `population/{CODE}/` per-country |
| `indicator.poverty_rate` | WB Poverty & Inequality Platform | `poverty/poverty.py` |
| `indicator.subnational_poverty_rate` | WB PIPMaps | `poverty/subnational_poverty/` per-country |
| `indicator.country` | WB API (`wbgapi`) + WB corporate currency table | `country.py` |
| `indicator.gdp` | WB & OECD National Accounts | `gdp.py` |
| `indicator.global_data_lab_hd_index` | Global Data Lab SHDI API | `human_development/global_data_lab_hdi_extract.r` + DLT |
| `indicator.learning_poverty_rate` | WB Data360 | `education/learning_poverty.py` |
| `indicator.edu_spending` | WB ICP / UNESCO UIS | `education/education_spending_icp.py` |
| `indicator.health_expenditure` | WHO GHO (CHE, OOP) | `health/health_expenditure.py` |
| `indicator.universal_health_coverage_index_gho` | WHO GHO (UHC index) | `health/sdg_health.py` |
| `indicator.pefa_by_pillar` | PEFA Secretariat | `pefa/pefa_transform_load.py` |
| `indicator.admin1_boundaries_gold` | geoBoundaries project | `geo/admin_boundaries_dlt.py` |
| `indicator.admin0_disputed_boundaries_gold` | geoBoundaries project | `geo/admin_boundaries_dlt.py` |
| `boost.boost_gold` | BOOST Excel workbooks (Ministries of Finance) | Per-country ETL in `mega-boost/{Country}/` |
| All `boost.*` aggregate tables | Derived from `boost_gold` | `mega-boost/cross_country_aggregate_dlt.py` |

The push via MCP failed due to permissions. Let me retry the git push.