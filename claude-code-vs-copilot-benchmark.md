# Claude Code vs. GitHub Copilot: A Real Benchmark on Large Multi-Repo Codebases

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
- GitHub Copilot with Claude Opus 4.6 — **200K token context window cap**
- Claude Code with Claude Opus 4.6 — **1M token context window**

Same model. Same task. Same repos. Different context limits.

---

## The Codebase Size

Before looking at results, here's what we were asking each tool to read:

| Repo | Total Characters | Est. Tokens |
|------|-----------------|-------------|
| `rpf-country-dash` | ~1,228,000 | ~307,000 |
| `mega-boost` | ~432,000 | ~108,000 |
| `mega-indicators` | ~105,000 | ~26,000 |
| **Total** | **~1,765,000** | **~441,000** |

This number matters. The full codebase is ~441K tokens — **more than double GitHub Copilot's 200K context cap**, but comfortably within Claude Code's 1M limit.

---

## What Happened

### Claude Code — Single Run

Claude Code's internal orchestration is not fully transparent — it likely spawned per-repo agents, but the exact mechanism is not confirmed. What is observable is the output: it systematically covered all three repos with full depth in a single coherent run, fitting within its 1M context window (~441K tokens used).

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
- ✅ Consistent, single coherent document

### GitHub Copilot — Run 1

**Output:**
- ❌ 20 of 27 charts documented — stopped mid-health section
- ❌ Missing charts 21–27 entirely (health outcomes, health maps, health subnational ranking)
- ❌ Missing OECD SDMX API as a source
- ❌ No system architecture overview
- ❌ No transformation logic — only pipeline structure (table names, script names)

### GitHub Copilot — Run 2

On a second attempt, Copilot spawned 4 agents and divided the work across them. Notably, **each of these agents had a 1M token context window** — Copilot's 200K cap applies to its standard chat interface, but the multi-agent orchestration mode allocates larger per-agent contexts.

**Output:**
- ✅ All 27 charts documented
- ⚠️ Transformation depth shallower than Claude Code
- ⚠️ Non-deterministic — different outcome from identical prompt on run 1

---

## The Key Finding

Here is where it gets interesting. When we investigated why Copilot run 2 succeeded, we found that each of the 4 agents also had a 1M token context window. Copilot's **200K cap applies to its standard chat interface** — when it escalates to multi-agent mode, each agent gets a larger context budget.

So in multi-agent mode, the context window size wasn't the differentiator in terms of raw capability. Both tools, at their best, were running on 1M token contexts.

**The real differentiator is reliability and controllability:**

| | Copilot Run 1 | Copilot Run 2 | Claude Code |
|---|---|---|---|
| Charts covered | 20/27 | 27/27 | 27/27 |
| Agents used | unclear | 4 | unknown, likely per-repo |
| Transformation depth | low | moderate | high |
| Reproducible? | — | different from run 1 | consistent |
| Work division controlled by user? | ❌ | ❌ | ✅ |

---

## Why the 1M Context Window Still Matters

Even though both tools can access 1M token contexts, Claude Code's single large context provides something the multi-agent approach cannot: **structural guarantee of completeness**.

Think of it this way. If you need to understand a system that spans three interconnected repos, you can either:

- **Option A:** Put the whole system in front of one person who reads everything and reasons across it
- **Option B:** Split it across four people who each read a portion and try to coordinate

Option B *can* work — but whether it does depends entirely on how intelligently the work was divided, whether the division happened to put related files in the same agent's context, and whether the agents' outputs were coherently stitched together. You have no visibility into or control over any of this.

With 441K tokens of codebase and a 1M context window, Claude Code fits everything in Option A. There is no orchestration gamble. Completeness is structurally guaranteed, not lucky.

> **The 1M context window isn't what makes Claude Code produce better output — it's what makes reliable output structurally guaranteed rather than dependent on opaque multi-agent orchestration you can't control or predict.**

---

## When Does This Actually Matter?

This benchmark is most relevant for tasks where:

1. **Data or logic spans multiple repos** — transformation logic in one repo, schema definitions in another, business rules in a third
2. **You need the full picture to answer correctly** — missing one repo means missing sources, missing transformations, missing the seam where bugs hide
3. **Reproducibility matters** — documentation, onboarding materials, audit trails, anything that needs to be consistent across runs

It matters less for:
- Single-file work or tasks contained within one repo
- Inline code completion where Copilot's editor integration is faster
- Tasks where "good enough on most runs" is acceptable

---

## Honest Caveats

- We ran this benchmark once per tool (with one Copilot retry). A rigorous benchmark would run multiple times and measure consistency statistically.
- We cannot directly observe what each Copilot agent received in its context — the multi-agent orchestration is a black box.
- Claude Code's internal orchestration is similarly opaque — we cannot confirm whether it used per-repo agents or a different strategy.
- Transformation depth in Copilot run 2 was not exhaustively compared against Claude Code output chart by chart.
- This is one task type. Results may differ for other task categories.

---

## Bottom Line

For large, multi-repo codebase tasks at the World Bank — pipeline documentation, data lineage tracing, cross-repo debugging — Claude Code's single large context produces more complete, more consistent, and more trustworthy output than Copilot's multi-agent approach.

Not because the model is different. Not necessarily because one is smarter. But because fitting the whole problem in one context removes the orchestration uncertainty that makes multi-agent output non-deterministic.

When reliability matters, context window size is the structural guarantee — not just a spec sheet number.

---

## Raw Outputs

The full outputs from each tool are included for reference:

- [Claude Code output](./claudecode.md)
- [GitHub Copilot output](./githubcopilot.md)

---

*Tested on: RPF Country Dashboard, mega-boost, mega-indicators — DIME/DECDI, World Bank*
*Model: Claude Opus 4.6 (both tools)*
*Date: March 2026*
