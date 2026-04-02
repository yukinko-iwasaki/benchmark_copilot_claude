---
layout: post
title: "Claude Code vs. Claude Agent in Copilot: A Real Benchmark on Large Multi-Repo Codebases"
date: 2026-03-25
author: Yukiko Suzuki
description: "A real-world benchmark comparing native Claude Code and the Claude Agent in GitHub Copilot on a large, multi-repo data lineage documentation task."
---


## The Question We Actually Wanted to Answer

Most AI coding tool comparisons focus on "who writes better code?" That's the wrong question for a lot of real work. Our question was more specific:

> **When the task requires understanding a large, multi-repo codebase end-to-end, which tool produces more complete and reliable output?**

We tested this on a real task — documenting the full data lineage of the public finance expenditure visualization dashboard, a Dash/Plotly application backed by two processing repos (`mega-boost` and `mega-indicators`) running on Databricks.

---

## The Task

**Prompt (identical for all tools):**

> *"Here are three repos: `rpf-country-dash` for visualization, `mega-boost` and `mega-indicators` for processing the underlying data. I would like you to summarize how each of the data used in the dashboard is created and tag them with their source when appropriate. Please make a summary chart by chart on the data source and how it is processed."*

**Tools tested:**
- Claude Agent in GitHub Copilot with Claude Opus 4.6 — on the org laptop (VS Code extension)
- Native Claude Code with Claude Opus 4.6 — via the browser (claude.ai), since Claude Code is not approved on the org laptop

Same model. Same task. Same repos — each frozen on a `benchmark` branch to ensure identical content across runs. The key difference: the VS Code extension caps context at 200K tokens, while native Claude Code operates with a 1M token context window (Max Plan Subscription).

---

## The Codebase Size

Before looking at results, here's what we were asking each tool to read:

| Repo | Total Characters | Est. Tokens |
|------|-----------------|-------------|
| `rpf-country-dash` | ~1,228,000 | ~307,000 |
| `mega-boost` | ~432,000 | ~108,000 |
| `mega-indicators` | ~105,000 | ~26,000 |
| **Total** | **~1,765,000** | **~441,000** |

This number matters. The full codebase is ~441K tokens — more than double the 200K context window available to the Claude Agent in VS Code.

---

## What Happened

### Claude Agent in GitHub Copilot — Org Laptop

The Claude Agent in Copilot produced a comprehensive summary covering all 27 charts in the dashboard, despite operating under a 200K context window cap imposed by the VS Code extension. The output was organized as a clean table-based format with columns for chart name, type, DB table, processing repo, and ultimate source.

| | |
|---|---|
| **Context window** | 200K (VS Code cap) |
| **Charts covered** | 27/27 |
| **Format** | Table-based, organized by page and tab |
| **Transformation depth** | Moderate — identifies pipeline structure and sources |
| **External sources** | Captured major sources (BOOST, World Bank API, WHO, PEFA, GDL, UNESCO/OECD) |

**Strengths:**
- Complete coverage — all 27 charts documented
- Clean, scannable table format with chart type, DB table, processing repo, and source in one view
- Data flow summary diagram included
- External data sources summary table at the end

**Limitations:**
- Brief summary of the data pipeline. 
- No mention of the key columns

### Native Claude Code — Browser (claude.ai/code)

Since Claude Code is not approved on the org laptop, we ran it via the browser version (claude.ai/code) with the same repos uploaded. Claude Code spawned 4 sub-agents — three appeared to read the repos in parallel (one per repo), then a fourth handled the planning and writing of the final summary.

| | |
|---|---|
| **Context window** | 1M |
| **Sub-agents** | 4 (3 parallel readers + 1 planner/writer) |
| **Charts covered** | 27/27 |
| **Transformation depth** | High — traces processing logic to specific scripts, parameters, and formulas |

**Strengths:**
- Complete coverage — all 27 charts documented
- Deeper transformation detail with mention of the key columns:
    - Native Claude Code:
    > "Aggregated from boost_gold by country and year. real_expenditure is inflation-adjusted using CPI from prd_mega.indicator.consumer_price_index. Foreign/domestic split comes from the is_foreign flag in the harmonized microdata."

    - Claude Agent in VS Code:
    > "BOOST Excel microdata (country government budget records) + CPI for inflation adjustment (mega-indicators -> World Bank API)"

- Detailed data description: e.g. Poverty threshold logic documented with specific income-level breakpoints (LIC $3.00, LMIC $4.20, UMIC/HIC $8.30)
- Full reference tables mapping every Databricks table to its processing script and original source
- Detailed Data Pipeline Summary — specific transformations are generalized and noted for boost data harmonization.

**Limitations:**
- Longer output — more reading required to find specific information
- Missed some details for special handling of specific countries (Albania) — if providing such detailed data descriptions, it is better to be thorough.

### Head-to-Head Comparison

| | Claude Agent in Copilot | Native Claude Code |
|---|---|---|
| Interface | VS Code extension (org laptop) | Browser (claude.ai) |
| Context window | 200K (VS Code cap) | 1M |
| Sub-agents | 4 | 4  |
| Charts covered | 27/27 | 27/27 |
| Transformation depth | Moderate | High |
| Output format | Tables | Narrative with reference tables |

Both tools achieved full coverage. The key difference was depth: Claude Code traced transformations down to specific scripts and their specific transformation logic, while the Claude Agent in Copilot documented the pipeline structure and sources at a higher level.

---

## Supplementary Observation: Local Agent with Claude LLM

As an additional data point, we also tested a local agent framework using Claude as the underlying LLM. This configuration performed significantly worse than either of the main tools.

The local agent did not spawn any sub-agents, attempting to process the entire multi-repo codebase in a single pass. The result was incomplete — it stopped partway through the task, leaving some sections of the dashboard undocumented.

This reinforces that agent orchestration — not just the underlying model — matters significantly for complex, multi-repo tasks. The same Claude model produced very different results depending on the agent framework managing it.

Sub-agents appear to be an enabled feature, but it is not clear what made the agent orchestration in this case different from the main two tools.

---

## What This Tells Us


> **Both tools covered all 27 charts. The difference was depth — driven by a 5x gap in context window (200K vs. 1M). Claude Code traced transformations to specific scripts while the Claude Agent in Copilot (constrained to 200K) documented the pipeline at a structural level. For tasks where knowing "PEFA scores are transformed" is sufficient, either tool works. For tasks where you need to know the scores convert letter grades A+ through D to numeric 4.5 to 1.0 across two framework versions — the larger context window made the difference.**

---

## A Note on Cost: Premium Request Quota

One practical consideration we hit during this benchmark: running Claude Opus 4.6 through GitHub Copilot burns premium requests at 3x the rate of standard models. A single large task like this one can consume a significant chunk of your monthly quota.

![GitHub Copilot premium request usage at 100.8%]({{ site.baseurl }}/assets/images/copilot-premium-quota.png)

*Our premium request quota after running the benchmark several times — fully consumed for the month.*

Two takeaways for teams using premium models in Copilot:

1. **Be mindful of request size.** Large, multi-repo tasks are expensive. Breaking a big task into smaller, targeted requests may produce better results even when the resources are limited.
2. **Match the model to the task.** Premium models like Opus are powerful but may be overkill for simple, contained tasks. Save them for work that genuinely benefits from deeper reasoning or larger context, and use standard models for routine work.

---

## Honest Caveats

- Claude Code was run via the browser (claude.ai), not as a local CLI tool, since it is not approved on the org laptop. Copilot was run as the VS Code extension. The tools had different interfaces for accessing the repos, which could affect behavior.
- We cannot observe exactly how each tool divided work internally — the orchestration is opaque in both cases. The "3 readers + 1 planner/writer" pattern for Claude Code is based on observable behavior, not confirmed internals.
- Transformation depth was assessed qualitatively, not with a systematic scoring rubric.
- This is one task type (data lineage documentation across 3 repos). Results may differ for other task categories.
- A rigorous benchmark would run multiple times per configuration and measure consistency statistically.

---


## Raw Outputs

The full outputs from each tool are included for reference:

- [Claude Agent in Copilot output]({{ site.baseurl }}/raw-outputs/claude-agent-copilot/)
- [Native Claude Code output]({{ site.baseurl }}/raw-outputs/native-claude-code/)
- [Local Agent with Claude LLM output (supplementary)]({{ site.baseurl }}/raw-outputs/local-agent-claude-llm/)

---

*Tested on: RPF Country Dashboard, mega-boost, mega-indicators — DIME/DECDI, World Bank*
*Model: Claude Opus 4.6 (all tools)*
*Date: March 2026*
