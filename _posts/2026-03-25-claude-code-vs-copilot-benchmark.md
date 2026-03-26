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

Copilot processed the task with a single 200K context window and no sub-agents. We found what appeared to be a cost optimization system prompt on the org laptop that may have disabled multi-agent orchestration — though we cannot confirm this with certainty. Whatever the cause, Copilot did not spawn sub-agents and attempted the 441K token task through a single 200K context.

| | |
|---|---|
| **Sub-agents** | 0 |
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

Claude Code spawned 4 sub-agents. Three sub-agents appeared to read the repos in parallel (one per repo), then a fourth sub-agent handled the planning and writing of the final summary. Each sub-agent had a 1M token context window.

| | |
|---|---|
| **Sub-agents** | 4 (3 parallel readers + 1 planner/writer) |
| **Context per sub-agent** | 1M |
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
| Sub-agents | 0 | 4 (3 readers + 1 planner/writer) |
| Context per sub-agent | 200K (single context) | 1M each |
| Charts covered | ~19/27 | 27/27 |
| Transformation depth | low | high |

---

## Why Copilot Fell Short on the Org Laptop

Copilot did not use sub-agents on the org laptop. We found what appeared to be a cost optimization system prompt that may have been responsible for disabling multi-agent orchestration — but this is speculation, and we cannot confirm the exact cause. What we can confirm is the outcome: Copilot attempted a 441K token codebase through a single 200K context window, which was insufficient to cover the full task.

This may not reflect Copilot's full capability. If the org-level configuration was indeed the constraint, then the tool was operating under a limitation it didn't choose, and the user had no visibility into it.

---

## Supplementary Observation: Personal Laptop

To investigate whether the org result reflected a Copilot limitation or a configuration issue, we re-ran the same prompt on a personal laptop that did not have the cost optimization system prompt.

| | |
|---|---|
| **Sub-agents** | 4 |
| **Context per sub-agent** | 200K each |
| **Charts covered** | 27/27 |
| **Transformation depth** | Moderate (shallower than Claude Code) |

**Key takeaways from the personal laptop run:**

- Copilot spawned 4 sub-agents when run on the personal laptop — suggesting (though not proving) that the cost optimization system prompt on the org laptop may have been responsible for the 0-sub-agent behavior
- With 4 sub-agents, Copilot covered all 27 charts — the completeness gap disappears when multi-agent is enabled
- Transformation depth was still shallower than Claude Code's output — processing logic described at pipeline level but missing specific parameter values and edge cases

**This result is not a valid org comparison** — it was a different environment with a different configuration. But it adds important nuance: even when sub-agent count is equalized (both tools using 4 sub-agents), context window size per sub-agent (200K vs. 1M) appears to independently affect output depth.

---

## What This Tells Us

Two things emerged from this benchmark:

**1. Something on the org laptop prevented Copilot from using sub-agents.** We found a cost optimization system prompt that may have been responsible, but we cannot confirm this definitively. Whatever the cause, Copilot was left with a single 200K context against a 441K token codebase — mathematically insufficient. If your organization has similar configurations, it's worth checking whether they affect tool behavior.

**2. Context window per sub-agent matters for depth.** The personal laptop result — where both tools used 4 sub-agents — shows that 200K per sub-agent produces shallower output than 1M per sub-agent on the same task. When tracing a data pipeline end-to-end across repos, a sub-agent may need to hold files from multiple repos simultaneously. With 200K, it has to summarize or skip. With 1M, there is headroom.

> **On the org laptop, Copilot did not use sub-agents — possibly due to a cost optimization system prompt — leaving it with a single 200K context against a 441K token codebase. Claude Code with 4 sub-agents (3 parallel readers + 1 planner/writer) at 1M each had no such constraint.**
>
> **The personal laptop result adds nuance: even when Copilot gets 4 sub-agents, 200K per sub-agent produces shallower output than 1M per sub-agent — suggesting context window size per sub-agent matters independently of sub-agent count.**

---

## When Does This Actually Matter?

This benchmark is most relevant for tasks where:

1. **Data or logic spans multiple repos** — transformation logic in one repo, schema definitions in another, business rules in a third
2. **You need depth, not just structure** — knowing that "PEFA scores are transformed" is less useful than knowing they convert letter grades A+ through D to numeric 4.5 to 1.0 across two framework versions
3. **Your org may constrain tool behavior** — system prompts, cost optimization settings, and enterprise configurations may limit what tools can do without the user's knowledge

It matters less for:
- Single-file work or tasks contained within one repo
- Inline code completion where Copilot's editor integration is faster
- Tasks where structural-level documentation is sufficient

---

## Honest Caveats

- The org vs. personal laptop difference was discovered after the fact, not designed as a controlled experiment.
- The cost optimization system prompt is our best guess for why Copilot didn't use sub-agents on the org laptop, but we cannot confirm this is the definitive cause.
- We cannot observe exactly how each tool divided work across its sub-agents — the orchestration is opaque in both cases. The "3 readers + 1 planner/writer" pattern for Claude Code is based on observable behavior, not confirmed internals.
- Transformation depth was assessed qualitatively, not with a systematic scoring rubric.
- This is one task type (data lineage documentation across 3 repos). Results may differ for other task categories.
- A rigorous benchmark would run multiple times per configuration and measure consistency statistically.

---

## Bottom Line

For large, multi-repo codebase tasks at the World Bank — pipeline documentation, data lineage tracing, cross-repo debugging — Claude Code produced more complete and deeper output than Copilot on the org laptop.

On the org laptop, Copilot did not spawn sub-agents — possibly due to a cost optimization system prompt — and was left with a single 200K context against a 441K token codebase. Claude Code used 4 sub-agents (3 reading repos in parallel, 1 planning and writing the summary) with 1M context each.

The supplementary personal laptop test suggests Copilot *can* match Claude Code's coverage when sub-agents are enabled — but even then, 200K per sub-agent produces shallower results than 1M per sub-agent.

When depth matters on large codebases, context window size per sub-agent is not a spec sheet number — it's the variable that determines whether you get structural summaries or actionable documentation.

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
