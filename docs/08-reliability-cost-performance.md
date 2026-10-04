# 08 · Reliability, cost and performance

A lot of Dhiti's work is long and LLM-heavy, and it runs on serverless functions that get killed outright at 300 s. Making that reliable took a few things. The work runs on the server with checkpoints, it stays inside a time budget under the hard limit, exam generation is locked to once per session, and every LLM call is capped and measured. Below, each mechanism comes with the problem it solved and, where there is one, its measured effect.

## Summary

| Mechanism | Problem it solved | Detail |
|---|---|---|
| Server-side pipeline orchestration | Closing the tab stopped career matching | [Below](#server-side-orchestration) and [05](05-career-matching-pipeline.md#how-the-pipeline-runs) |
| Early role selection | Candidates waited on an enrichment step they didn't need yet | [Below](#unblocking-role-selection) |
| Generation concurrency 10 → 25 | Each worker ran about 2.5 rewrites back to back | [Below](#exam-generation-concurrency) |
| 250 s time budget | The 300 s platform limit is a hard kill | [Below](#time-budget-under-a-hard-limit) |
| Generation lock | Duplicate generations were the biggest avoidable LLM cost | [Below](#one-generation-per-session) |
| Per-session call ceilings | Worst-case LLM cost per session needed a hard bound | [Below](#per-session-llm-call-ceilings) |
| LLM usage tracking | Cost and latency need to be visible per step | [Below](#llm-usage-tracking) |
| Activity-write throttle | A database write on every answer | [Below](#activity-write-throttle) |
| Fallback on failure | An LLM failure could block a candidate | [Below](#failure-handling) |

## Server-side orchestration

All four career-matching steps run in a single server-side request. As each step finishes, its result is saved as a checkpoint and streamed to the browser as an NDJSON event. The run carries on after the response is sent. A reload joins the run that's already in progress. And a newer run takes over from an older one, for example after a constraint answer changes. However it goes, the stream always ends in exactly one of `done`, `superseded` or `error`. The full detail is in [Career matching pipeline](05-career-matching-pipeline.md#how-the-pipeline-runs).

## Unblocking role selection

Role selection used to wait for step 4, the LLM enrichment step. But the shortlist the candidate chooses from already exists after step 3. So role selection now opens at that point, and enrichment finishes in the background.

> **Removed a median 46 s wait.** Role selection now opens as soon as the top-3 shortlist exists, instead of waiting for the LLM enrichment step. That step's median duration was **45.7 s** across 83 runs.

## Exam generation concurrency

Every exam needs 25 scenario rewrites. With 10 workers, each worker ran about 2.5 jobs back to back, so the total time was set by whichever worker's queue was slowest. With 25 workers, every rewrite starts at once, and the whole pass takes only as long as the single slowest scenario.

Rate limits were never the constraint here. One wave of 25 uses under 5% of the tightest API limit.

There is a trade-off. Tail latency now decides the duration outright, because one slow rewrite holds up the whole batch. That's the reason for the time budget below.

## Time budget under a hard limit

The serverless functions have a hard **300 s** limit. When a function hits it, it's killed on the spot, with no graceful stop, and whatever it was doing is lost. So the exam generation route works to a **250 s** budget:

| Phase | Status |
|---|---|
| Scenario selection and regrounding, with retries and fallback to the validated original | Mandatory |
| Duplicate audit and regeneration of flagged items | Best-effort: skipped if there isn't time |

Whatever happens, the candidate gets an exam.

Here's the measured effect of the concurrency change and the time budget together:

> **Exam generation 53% faster in the worst case and 27% faster at p90.** Raising regrounding concurrency from 10 to 25, plus the 250 s time budget, took worst-case generation time from **399 s to 189 s** and p90 from **259 s to 189 s**. The median moved from 147 s to 137 s. *Measured from production session timestamps, from role selection to scenarios ready: 36 runs before, 8 runs after. A small sample after the change.*

> **Runs exceeding the 300 s serverless limit went from 3 of 36 to 0 of 8.** *Same small sample. Some of the slow early runs may have been in local development, which has no such limit.*

## One generation per session

A generation lock, stored with the session, stops a second request from starting another generation while the first is still running. The lock expires after a period matched to the platform's maximum function duration, so it can never outlive the longest possible run. This mattered because duplicate generations were the single biggest avoidable LLM cost.

> **About 30% lower LLM cost per assessment (an estimate).** This comes mainly from the generation lock stopping duplicate scenario generations. *Modelled from token counts and list prices, not from billing data.*

## Per-session LLM call ceilings

Every paid route requires a valid paid session owned by the caller. On top of that, each session has ceilings on how many LLM calls it can make, which puts a hard bound on the worst-case cost of any session.

There are five buckets, plus **an overall ceiling** across all of them:

- scenario generation
- trait questions
- chat turns
- scoring
- report

These are **abuse ceilings, not throttles**. Each one sits several times above what a real, complete session uses, reloads and retries included. If a real candidate ever trips one, the ceiling is wrong, not the candidate.

When a ceiling is tripped, the operators get an email. There's a **15-minute cooldown** on those alerts, so a single incident doesn't flood the inbox. The actual ceiling values aren't published.

## LLM usage tracking

Every Claude call records:

- the step label
- the model
- tokens
- the computed cost
- latency, as a histogram
- the outcome

This bookkeeping runs **after the response has been sent**, so the candidate never waits on it. The admin Overview shows LLM cost, tokens and latency **per pipeline step**, next to daily metrics, weekly cohorts and payment reconciliation. The cost and pricing maths live in the pure core and have their own unit tests.

## Activity-write throttle

Each session keeps a last-activity timestamp. Rather than writing it on every answer, Dhiti writes it at most **once per 60 seconds** per session.

> **69% fewer session-activity database writes.** The 60-second throttle was replayed against **4,305 real answer timestamps**: 4,305 writes became 1,349. *This covers activity-timestamp writes only, not all database writes.*

## Failure handling

A few rules keep failures from reaching the candidate:

- A scenario rewrite that fails is retried, up to 3 attempts in total, and then falls back to the validated generic original. A failed LLM call never blocks a candidate.
- Non-answer checks run a free heuristic before any LLM call. See [Trait assessment](04-trait-assessment.md#handling-non-answers).
- Pipeline errors end the stream with an explicit `error` event, so an error can no longer be passed on as the next step's input.

## See also

- [Results and measurements](13-results-and-measurements.md): every measured result, with its method and caveats
- [Engineering decisions](11-engineering-decisions.md): ADR-04 to ADR-08 and ADR-11 cover these choices
