# 11 · Engineering decisions

These are the decisions that did the most to shape how Dhiti is built. Each one is written as a lightweight architecture decision record (ADR): the context (what the problem was), the decision, and the consequences and trade-offs that came with it. They're grouped by area rather than by date.

| # | Decision | Area |
|---|---|---|
| [01](#adr-01-inject-retrieval-directly-instead-of-using-a-vector-database) | Inject retrieval directly instead of using a vector database | Exam |
| [02](#adr-02-deterministic-multiple-choice-scoring-instead-of-an-llm-grader) | Deterministic multiple-choice scoring instead of an LLM grader | Exam |
| [03](#adr-03-reground-validated-seeds-instead-of-generating-questions-from-scratch) | Reground validated seeds instead of generating from scratch | Exam |
| [04](#adr-04-move-pipeline-orchestration-from-the-browser-to-the-server) | Move pipeline orchestration from the browser to the server | Reliability |
| [05](#adr-05-raise-generation-concurrency-from-10-to-25) | Raise generation concurrency from 10 to 25 | Performance |
| [06](#adr-06-work-to-a-time-budget-under-a-hard-platform-limit) | Work to a time budget under a hard platform limit | Reliability |
| [07](#adr-07-lock-generation-once-per-session) | Lock generation once per session | Cost |
| [08](#adr-08-abuse-ceilings-not-throttles) | Abuse ceilings, not throttles | Cost |
| [09](#adr-09-one-canonical-trait-registry) | One canonical trait registry | Correctness |
| [10](#adr-10-set-difficulty-from-pre-constraint-fit) | Set difficulty from pre-constraint fit | Assessment |
| [11](#adr-11-throttle-activity-writes) | Throttle activity writes | Cost |
| [12](#adr-12-use-honest-confidence-everywhere) | Use honest confidence everywhere | Product |

---

## ADR-01: Inject retrieval directly instead of using a vector database

**Context.** To reground a scenario into an industry, the model needs grounding facts about both the role and the industry. The knowledge corpus is about 1,100 chunks spread across 46 packs (30 role packs and 16 industry packs).

**Decision.** Put the relevant packs straight into the prompt, with no vector database and no embeddings.

**Consequences and trade-offs.**
- There's no embedding pipeline that has to be kept in sync with the content.
- There are no retrieval misses, because the model sees everything relevant for that role and industry.
- The corpus stays as versioned JSON, so it gets reviewed like any other change.
- *Trade-off:* if the corpus grows a lot, this approach will have to change. The retrieval layer sits behind a small interface, so it can be swapped out when that day comes.

## ADR-02: Deterministic multiple-choice scoring instead of an LLM grader

**Context.** An earlier internal design planned 3 free-text scenarios, graded by an LLM.

**Decision.** Ship 25 validated multiple-choice scenarios instead, scored by exact set match. A scenario is correct only if the selected set equals the correct set. Overall score = correct / total × 5.

**Consequences and trade-offs.**
- Scores are reproducible and instant.
- There's no grader to drift.
- Scoring costs nothing in LLM spend.
- A full rationale review is possible, because every option has a rationale.
- *Trade-off:* the exam measures judgment between options, not open-ended writing.

## ADR-03: Reground validated seeds instead of generating questions from scratch

**Context.** Candidates should be tested in the industry they chose, on the traits they most need tested. But if an LLM writes questions from scratch, the answer keys could come out ambiguous or simply wrong.

**Decision.** Start from hand-authored, validated seed scenarios, and let the LLM change only the *setting*: the industry, the target trait and the phrasing. Code-level checks guarantee it can't change *which answers are correct*. IDs, format and select count stay the same, there are 4–6 options each with text and a rationale, the correct options can't move, and the number of correct options equals the select count.

**Consequences and trade-offs.**
- Personalisation never compromises the answer key.
- If a rewrite still fails after retries, the candidate gets the validated original. A failed LLM call never blocks a candidate.
- Pattern checks catch lazy, repetitive phrasing that validation alone would let through.
- *Trade-off:* the exam can only be as varied as the seed bank (900 scenarios). New kinds of situations need new seeds.

## ADR-04: Move pipeline orchestration from the browser to the server

**Context.** The browser used to chain the four career-matching steps as four separate requests. Closing the tab stopped the chain. And an error from one step could be passed along as if it were the next step's input.

**Decision.** Run all four steps as one server-side request. It streams NDJSON events, saves a checkpoint after each step, and has exactly one owning run at a time.

**Consequences and trade-offs.**
- The run survives a closed tab.
- A reload joins the run that's already in progress instead of starting another one.
- A resumed session reuses the completed steps (as long as they form an unbroken prefix) and only computes what's missing.
- Re-answering a constraint question starts a newer run that takes over, and the older one stops writing.
- The stream always ends in exactly one of `done`, `superseded` or `error`.
- *Trade-off:* the server now owns a long-lived request.

## ADR-05: Raise generation concurrency from 10 to 25

**Context.** Every exam needs 25 scenario rewrites. With 10 workers, each worker ended up running about 2.5 jobs back to back.

**Decision.** Run 25 workers, one per scenario.

**Consequences and trade-offs.**
- The pass is now only as slow as the single slowest scenario, instead of being held up by a queue.
- Rate limits weren't a concern. One wave uses under 5% of the tightest API limit.
- *Trade-off:* tail latency now sets the duration outright, and that's exactly why ADR-06 exists.
- For the measured effect of this together with ADR-06, see [result 1](13-results-and-measurements.md#result-1-exam-generation-time).

## ADR-06: Work to a time budget under a hard platform limit

**Context.** The serverless platform cuts functions off at 300 s. It's a hard kill, not a graceful stop.

**Decision.** Work to a 250 s budget, split into phases. Regrounding (with fallback) is mandatory. The duplicate audit and regeneration are best-effort, and they're skipped if time runs short.

**Consequences and trade-offs.**
- The candidate always gets an exam.
- *Trade-off:* on a slow run, that batch may miss its near-duplicate audit.
- For the measured effect, see [results 1 and 2](13-results-and-measurements.md#result-2-runs-over-the-300-s-limit).

## ADR-07: Lock generation once per session

**Context.** A second request could start a second exam generation while the first one was still running. Those duplicate generations were the single biggest avoidable LLM cost.

**Decision.** Store a single generation lock with the session, with an expiry matched to the platform's maximum function duration.

**Consequences and trade-offs.**
- A second request can't start a duplicate generation.
- Because the lock expires, it can never outlive the longest possible run.
- For the estimated effect, see [result 5](13-results-and-measurements.md#result-5-llm-cost-per-assessment-estimate). It's an estimate, not a figure from billing.

## ADR-08: Abuse ceilings, not throttles

**Context.** Every paid route requires a valid paid session. On top of that, the cost of a single session needs a hard upper bound.

**Decision.** Add per-session LLM call ceilings in 5 buckets (scenario generation, trait questions, chat turns, scoring, report), plus an overall ceiling. Each one sits several times above what a real, complete session uses, reloads and retries included.

**Consequences and trade-offs.**
- The worst-case cost of a session is bounded.
- Real candidates are never affected. If a real candidate does trip a ceiling, the ceiling is wrong, not the candidate.
- A tripped ceiling emails the operators, with a 15-minute cooldown so one incident doesn't flood the inbox.
- *Trade-off:* whenever a normal session's call pattern changes, the ceilings need another look.

## ADR-09: One canonical trait registry

**Context.** Four hand-maintained trait tables had drifted apart, and the final report ended up showing the wrong labels.

**Decision.** Derive every trait representation from one registry row, typed so that leaving out a trait is a **compile error**.

**Consequences and trade-offs.**
- Labels can't drift between the assessment and the report.
- Adding or renaming a trait happens in one place.
- A missing trait fails the build, instead of shipping as a silent bug.

## ADR-10: Set difficulty from pre-constraint fit

**Context.** The exam's difficulty mix depends on how well the candidate fits the role. Logistics like hours, money and relocation pull a role's constrained score down, but they say nothing about the candidate's judgment.

**Decision.** Set the difficulty mix from trait fit *before* constraints are applied, then let experience level nudge it.

**Consequences and trade-offs.**
- Someone who's short on time but a strong fit still gets hard content.
- The exam measures judgment, not circumstances.

## ADR-11: Throttle activity writes

**Context.** The session's last-activity timestamp was being written on every single answer.

**Decision.** Write it at most once per 60 seconds per session.

**Consequences and trade-offs.**
- Far fewer writes. See [result 4](13-results-and-measurements.md#result-4-session-activity-writes).
- *Trade-off:* the stored timestamp can lag real activity by up to a minute. Against a 2-hour idle expiry, that doesn't matter.

## ADR-12: Use honest confidence everywhere

**Context.** On its own, a trait score sounds just as certain whether it rests on one vague answer or several detailed stories.

**Decision.** Make confidence (0–1) part of every trait score, and soften the language when evidence is thin: "you showed signs of", not "you are". This product principle is enforced in the scoring output itself.

**Consequences and trade-offs.**
- The report never claims more than the conversation actually showed.
- Confidence also drives the adaptive phase. Follow-ups go to the traits with the weakest evidence, and the assessment can end early once every trait has been asked about and reached 0.70.
- *Trade-off:* some reports read more tentatively. That's by design.
