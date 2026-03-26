---
layout: post
title: "Claude Code vs. GitHub Copilot: A Real Benchmark on Large Multi-Repo Codebases"
date: 2026-03-25
author: DIME/DECDI team, World Bank
description: "A real-world benchmark comparing Claude Code and GitHub Copilot on a large, multi-repo data lineage documentation task."
---

*A benchmark from the DIME/DECDI team at the World Bank*

---

## The Question We Actually Wanted to Answer

Most AI coding tool comparisons focus on "who writes better code?" That's the wrong question for a lot of real work. Our question was more specific:

> **When the task requires understanding a large, multi-repo codebase end-to-end, which tool produces more complete and reliable output?**

We tested this on a real task — documenting the full data lineage of the RPF Country Dashboard, a Dash/Plotly application backed by two processing repos (`mega-boost` and `mega-indicators`) running on Databricks.

---

## The Task

**Prompt (identical for both tools):**

> *"Here are three repos: `rpf-country-dash` for visualization, `mega-boost` and `mega-indicators` for processing the underlying data. I would like you to summarize how each of the data used in the dashboard is created and tag them with their source when appropriate. Please make a summary chart by chart on the data source and how it is processed."*

**Tools tested:**
- GitHub Copilot with Claude Opus 4.6 — on the org laptop
- Claude Code with Claude Opus 4.6 — on the org laptop

Same model. Same task. Same repos. Same machine.

---

## The Codebase Size

Before looking at results, here's what we were asking each tool to read:

| Repo | Total Characters | Est. Tokens |
|------|-----------------|-------------|
| `rpf-country-dash` | ~1,228,000 | ~307,000 |
| `mega-boost` | ~432,000 | ~108,000 |
| `mega-indicators` | ~105,000 | ~26,000 |
| **Total** | **~1,765,000** | **~441,000** |

This number matters. The full codebase is ~441K tokens — more than double Copilot's 200K context window.

---

## What Happened

### GitHub Copilot — Org Laptop

Copilot processed the task with a single 200K context window. The org's cost optimization system prompt had disabled multi-agent orchestration, forcing Copilot to attempt a 441K token task through a 200K context — mathematically guaranteed to be incomplete.

| | |
|---|---|
| **Agents** | 0 (disabled by cost optimization system prompt) |
| **Context** | 200K (single context) |
| **Charts covered** | ~19/27 |
| **Transformation depth** | Low |

**Output:**
- ❌ ~19 of 27 charts documented — stopped mid-health section
- ❌ Missing the last 8–10 charts entirely (health outcomes, health maps, health subnational ranking)
- ❌ Missing OECD SDMX API as a source
- ❌ No system architecture overview
- ❌ No transformation logic — only pipeline structure (table names, script names)

### Claude Code — Org Laptop

Claude Code spawned 4 agents to process the three repos. Each agent had a 1M token context window.

| | |
|---|---|
| **Agents** | 4 |
| **Context per agent** | 1M |
| **Charts covered** | 27/27 |
| **Transformation depth** | High |

**Output:**
- ✅ All 27 charts documented
- ✅ System architecture diagram showing how the three repos relate
- ✅ "Key Transformations" column with actual processing logic — e.g.:
  - Poverty threshold logic varying by income level (`$3.65/$6.85/$24.35`)
  - PEFA grade conversion (`A+ to D` → numeric `4.5 to 1.0`, harmonized across 2011 & 2016 frameworks)
  - 34+ country-specific boundary name corrections
  - ICP database IDs (71, 62, 90) and unit conversion from billion LCU
- ✅ All external sources captured including OECD SDMX API, Global Data Lab R API with auth token
- ✅ Supporting tables section with script-level lineage

### Primary Comparison

| | Copilot (Org Laptop) | Claude Code |
|---|---|---|
| Agents | 0 (disabled by system prompt) | 4 |
| Context per agent | 200K (single context) | 1M each |
| Charts covered | ~19/27 | 27/27 |
| Transformation depth | low | high |

---

## Why Copilot Failed on the Org Laptop

The org laptop's cost optimization system prompt disabled Copilot's multi-agent orchestration. This forced it into a single 200K context against a 441K token codebase. The result was structurally inevitable: it couldn't fit the full codebase, so it couldn't document all of it.

This is not an indictment of Copilot's underlying capability. It's a demonstration of what happens when **org-level configurations silently constrain tool behavior**. The tool was handicapped by a system prompt it didn't choose, and the user had no visibility into this constraint.

---

## Supplementary Observation: Personal Laptop

To understand whether the org result reflected a Copilot limitation or a configuration issue, we re-ran the same prompt on a personal laptop without the cost optimization system prompt.

| | |
|---|---|
| **Agents** | 4 |
| **Context per agent** | 200K each |
| **Charts covered** | 27/27 |
| **Transformation depth** | Moderate (shallower than Claude Code) |

**Key takeaways from the personal laptop run:**

- Copilot spawned 4 agents when unconstrained — confirming the cost optimization system prompt was the cause of the org laptop's 0-agent behavior, not a Copilot capability limitation
- With 4 agents, Copilot covered all 27 charts — the completeness gap disappears when multi-agent is enabled
- Transformation depth was still shallower than Claude Code's output — processing logic described at pipeline level but missing specific parameter values and edge cases

**This result is not a valid org comparison** — it was a different environment with a different configuration. But it adds important nuance: even when agent count is equalized (both tools using 4 agents), context window size per agent (200K vs. 1M) appears to independently affect output depth.

---

## What This Tells Us

Two things happened in this benchmark:

**1. Org configuration silently broke Copilot.** The cost optimization system prompt disabled multi-agent orchestration, forcing a 441K token task through a 200K context window. The user had no indication this was happening. If your organization uses system prompts that constrain Copilot's agent behavior, you may be getting degraded results without knowing it.

**2. Context window per agent matters for depth.** The personal laptop result — where both tools used 4 agents — shows that 200K per agent produces shallower output than 1M per agent on the same task. When tracing a data pipeline end-to-end across repos, an agent may need to hold files from multiple repos simultaneously. With 200K, it has to summarize or skip. With 1M, there is headroom.

> **In the org environment, Copilot's cost optimization disabled multi-agent orchestration, leaving it with a single 200K context against a 441K token codebase — mathematically guaranteed to be incomplete. Claude Code with 4 agents at 1M each had no such constraint.**
>
> **The personal laptop result adds nuance: even when Copilot gets 4 agents, 200K per agent produces shallower output than 1M per agent — suggesting context window size per agent matters independently of agent count.**

---

## When Does This Actually Matter?

This benchmark is most relevant for tasks where:

1. **Data or logic spans multiple repos** — transformation logic in one repo, schema definitions in another, business rules in a third
2. **You need depth, not just structure** — knowing that "PEFA scores are transformed" is less useful than knowing they convert letter grades A+ through D to numeric 4.5 to 1.0 across two framework versions
3. **Your org may constrain tool behavior** — system prompts, cost optimization settings, and enterprise configurations can silently limit what tools can do

It matters less for:
- Single-file work or tasks contained within one repo
- Inline code completion where Copilot's editor integration is faster
- Tasks where structural-level documentation is sufficient

---

## Honest Caveats

- The org vs. personal laptop difference was discovered after the fact, not designed as a controlled experiment.
- We cannot observe exactly how each tool divided work across its agents — the orchestration is opaque in both cases.
- Transformation depth was assessed qualitatively, not with a systematic scoring rubric.
- This is one task type (data lineage documentation across 3 repos). Results may differ for other task categories.
- A rigorous benchmark would run multiple times per configuration and measure consistency statistically.

---

## Bottom Line

For large, multi-repo codebase tasks at the World Bank — pipeline documentation, data lineage tracing, cross-repo debugging — Claude Code produced more complete and deeper output than Copilot on the org laptop.

The primary cause was an org-level cost optimization that silently disabled Copilot's multi-agent orchestration, forcing a 441K token task through a single 200K context. Claude Code, with 4 agents at 1M context each, had no such constraint.

The supplementary personal laptop test confirms Copilot *can* match Claude Code's coverage when multi-agent is enabled — but even then, 200K per agent produces shallower results than 1M per agent.

**Check your org's system prompts.** And when depth matters, context window size per agent is not a spec sheet number — it's the variable that determines whether you get structural summaries or actionable documentation.

---

## Raw Outputs

The full outputs from each tool are included for reference:

- [Claude Code output]({{ site.baseurl }}/raw-outputs/claudecode/)
- [GitHub Copilot output — Org Laptop]({{ site.baseurl }}/raw-outputs/githubcopilot/)
- [GitHub Copilot output — Personal Laptop]({{ site.baseurl }}/raw-outputs/githubcopilot-run2/)
- [Claude Code output — Run 2]({{ site.baseurl }}/raw-outputs/claudecode-run2/)

---

*Tested on: RPF Country Dashboard, mega-boost, mega-indicators — DIME/DECDI, World Bank*
*Model: Claude Opus 4.6 (both tools)*
*Date: March 2026*
