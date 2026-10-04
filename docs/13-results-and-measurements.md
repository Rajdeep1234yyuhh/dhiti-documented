# 13 · Results and measurements

These are the measured effects of specific engineering changes, each with how it was measured and where the measurement falls short. Some of the samples are small. The caveats are part of each result, so please read them together. One figure is a modelled estimate rather than a measurement, and it's labelled that way everywhere it appears.

## Summary

| # | Result | Change behind it | Basis and caveat |
|---|---|---|---|
| 1 | Exam generation 53% faster in the worst case and 27% faster at p90 | Concurrency 10 → 25, plus 250 s budget | Production session timestamps, 36 runs before, 8 after. A small sample after the change. |
| 2 | Runs exceeding the 300 s serverless limit went from 3 of 36 to 0 of 8 | Same | Same small sample. Some slow early runs may have been in local development, which has no such limit. |
| 3 | Removed a median 46 s wait | Role selection unblocked from enrichment | Median enrichment-step duration of 45.7 s across 83 runs |
| 4 | 69% fewer session-activity database writes | 60-second activity-write throttle | Replay of 4,305 real answer timestamps. Activity-timestamp writes only, not all database writes. |
| 5 | About 30% lower LLM cost per assessment (**an estimate**) | Generation lock | Modelled from token counts and list prices, not from billing data |

---

## Result 1: exam generation time

**Exam generation 53% faster in the worst case and 27% faster at p90.** Raising regrounding concurrency from 10 to 25, plus the 250 s time budget, took worst-case generation time from **399 s to 189 s** and p90 from **259 s to 189 s**. The median moved from 147 s to 137 s. *Measured from production session timestamps, from role selection to scenarios ready: 36 runs before, 8 runs after. A small sample after the change.*

| | Before (36 runs) | After (8 runs) |
|---|---|---|
| Worst case | 399 s | 189 s |
| p90 | 259 s | 189 s |
| Median | 147 s | 137 s |

**What changed.** Before, with 10 workers and 25 scenarios, each worker ran about 2.5 rewrites back to back. Now, with 25 workers, the pass takes only as long as the single slowest scenario. The 250 s budget also makes the duplicate audit best-effort, so the audit can't push a run past the platform limit. See [ADR-05](11-engineering-decisions.md#adr-05-raise-generation-concurrency-from-10-to-25) and [ADR-06](11-engineering-decisions.md#adr-06-work-to-a-time-budget-under-a-hard-platform-limit).

## Result 2: runs over the 300 s limit

**Runs exceeding the 300 s serverless limit went from 3 of 36 to 0 of 8.** *Same small sample. Some of the slow early runs may have been in local development, which has no such limit.*

**Why it matters.** In production, the platform kills any run that reaches 300 s. There's no graceful stop.

## Result 3: role-selection wait

**Removed a median 46 s wait.** Role selection now opens as soon as the top-3 shortlist exists, instead of waiting for the LLM enrichment step. That step's median duration was **45.7 s** across 83 runs.

**What changed.** The shortlist is already final after step 3 of the matching pipeline (constraint feasibility). Step 4, the role-map enrichment, adds personalised detail, but it doesn't change which three roles are on the shortlist. So there's no reason to make the candidate wait for it. See [Career matching pipeline](05-career-matching-pipeline.md#how-the-pipeline-runs).

## Result 4: session-activity writes

**69% fewer session-activity database writes.** The 60-second throttle was replayed against **4,305 real answer timestamps**: 4,305 writes became 1,349. *This covers activity-timestamp writes only, not all database writes.*

**Method.** Real answer timestamps were fed through the throttle logic, which writes the last-activity timestamp at most once per 60 seconds per session. The writes that came out the other end were counted. See [ADR-11](11-engineering-decisions.md#adr-11-throttle-activity-writes).

## Result 5: LLM cost per assessment (estimate)

**About 30% lower LLM cost per assessment (an estimate).** This comes mainly from the generation lock stopping duplicate scenario generations. *Modelled from token counts and list prices, not from billing data.*

**Why it's an estimate.** It's a model, not a bill: token counts priced at list prices. See [ADR-07](11-engineering-decisions.md#adr-07-lock-generation-once-per-session).

---

## How these are measured

Results 1 and 2 come from production session timestamps, timed from role selection to scenarios ready, with the local-development caveat on result 2. Result 3 is the median duration of the enrichment step across 83 runs.

Result 4 works differently. Instead of comparing two live periods, it replays real timestamps through the new logic.

Result 5 is a cost model built from token counts and list prices. Every Claude call records its step label, model, tokens, computed cost, latency and outcome, which is what makes the model possible (see [Reliability, cost and performance](08-reliability-cost-performance.md#llm-usage-tracking)).

## What this page doesn't include

Only engineering measurements are reported here. There are no user numbers, revenue or conversion figures.
