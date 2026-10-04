# Dhiti.ai

An AI career counsellor. It gets to know you through a conversation, takes the limits of your real life into account, and then tests you on the actual work of the role you choose.

### Try it live: **[www.dhiti.ai](https://www.dhiti.ai)**

Dhiti runs in your browser, so there's nothing to install. Open the site and sign in with email or Google to get started.

<!-- TODO(owner): confirm whether to link the product video (https://youtu.be/aIo-PEiSqQA) here -->

## What it is

Dhiti is a web app that does the job of a career counsellor. The candidate talks with it about real moments from their life, and from that conversation Dhiti scores them on 18 traits, each with a confidence level. It then asks about the practical limits on their options (time, money, whether they can move) and ranks 30 future-resilient roles across 6 families of work.

The candidate picks a role and an industry and answers 25 realistic workplace scenarios from that role, rewritten for that industry. At the end they get a report covering their best-fit roles, what was ruled out and why, how they did on the real work, and a staged roadmap.

## How a session works

1. **A conversation.** Dhiti asks about real moments from your life and follows up on what you tell it.
2. **Your shape.** Eighteen traits, each scored with how much evidence your answers gave.
3. **Your life.** Three quick questions: the hours you can spare, whether others rely on your income, and whether you can move.
4. **The real work.** Twenty-five scenarios from the role you pick, on a 60-minute clock that starts when you do.
5. **Your report.** Your best-fit roles, what was ruled out and why, and a roadmap.

The best way to understand it is to go through a session yourself at **[www.dhiti.ai](https://www.dhiti.ai)**.

## Engineering highlights

Each item starts with the problem it solved.

- **Closing the tab used to stop career matching.** The four matching steps now run on the server as a single run. It streams progress, checkpoints after every step and can pick up where it left off. More in [Career matching pipeline](docs/05-career-matching-pipeline.md).
- **Personalised questions couldn't be allowed to break the answer key.** Claude rewrites validated seed scenarios into the candidate's industry, and code checks make sure no correct answer can move. If a rewrite fails, the candidate simply gets the validated original. More in [Scenario exam engine](docs/06-scenario-exam-engine.md).
- **Serverless functions are killed outright at 300 s.** Exam generation works to a 250 s budget, split into required and best-effort phases, so every candidate still gets an exam. More in [Reliability, cost and performance](docs/08-reliability-cost-performance.md).
- **Duplicate exam generations were the biggest avoidable LLM cost.** A lock now allows one generation per session: about 30% lower LLM cost per assessment (an estimate, modelled from token counts and list prices, not from billing data). More in [Engineering decisions](docs/11-engineering-decisions.md).
- **Scores had to be reproducible and hard to game.** Scoring is deterministic, by exact match on the set of options the candidate selected. The answer key and the deadline both stay on the server. More in [Scenario exam engine](docs/06-scenario-exam-engine.md).
- **Worst-case LLM cost per session needed a hard bound.** Each session has ceilings on its LLM calls. They sit well above what any real session uses, cap the worst case, and alert the operators if tripped. More in [Reliability, cost and performance](docs/08-reliability-cost-performance.md).
- **Four trait tables had drifted apart.** Every trait now comes from one canonical registry, and leaving a trait out is a compile error. More in [Engineering decisions](docs/11-engineering-decisions.md).
- **Core logic was hard to test because it depended on a database and an LLM.** The core is now written as pure modules that have their dependencies passed in, and about 170 Vitest cases cover it. More in [Tech stack](docs/10-tech-stack.md).

## Measured results

These are given exactly as measured, caveats and all. The method and context are in [Results and measurements](docs/13-results-and-measurements.md).

1. **Exam generation 53% faster in the worst case and 27% faster at p90.** Raising regrounding concurrency from 10 to 25, plus the 250 s time budget, took worst-case generation time from **399 s to 189 s** and p90 from **259 s to 189 s**. The median moved from 147 s to 137 s. *Measured from production session timestamps, from role selection to scenarios ready: 36 runs before, 8 runs after. A small sample after the change.*
2. **Runs exceeding the 300 s serverless limit went from 3 of 36 to 0 of 8.** *Same small sample. Some of the slow early runs may have been in local development, which has no such limit.*
3. **Removed a median 46 s wait.** Role selection now opens as soon as the top-3 shortlist exists, instead of waiting for the LLM enrichment step. That step's median duration was **45.7 s** across 83 runs.
4. **69% fewer session-activity database writes.** The 60-second throttle was replayed against **4,305 real answer timestamps**: 4,305 writes became 1,349. *This covers activity-timestamp writes only, not all database writes.*

## Tech stack

Next.js 15 · React 19 · TypeScript · Tailwind CSS 4 · Framer Motion · GSAP · Chart.js · Vercel serverless functions · Anthropic Claude (Sonnet 4.6) · Google Cloud Firestore · NextAuth.js · Razorpay · Nodemailer · Vitest. The full breakdown is in [Tech stack](docs/10-tech-stack.md).

## Documentation

| # | Page | What it covers |
|---|---|---|
| 01 | [Product overview](docs/01-product-overview.md) | What Dhiti is, who it's for, and the principles behind it |
| 02 | [User journey](docs/02-user-journey.md) | The session phase by phase, from the candidate's side |
| 03 | [System architecture](docs/03-system-architecture.md) | Components, data flow and request lifecycle |
| 04 | [Trait assessment](docs/04-trait-assessment.md) | The adaptive, conversational assessment of 18 traits |
| 05 | [Career matching pipeline](docs/05-career-matching-pipeline.md) | The 4-step server-side scoring pipeline |
| 06 | [Scenario exam engine](docs/06-scenario-exam-engine.md) | The personalised 25-scenario exam, regrounded for each industry |
| 07 | [Report and roadmap](docs/07-report-and-roadmap.md) | What the final report contains and how it's built |
| 08 | [Reliability, cost and performance](docs/08-reliability-cost-performance.md) | Time budgets, locks, call ceilings and usage tracking |
| 09 | [Security, privacy and integrity](docs/09-security-privacy-integrity.md) | Access control, exam integrity, payments and regulated roles |
| 10 | [Tech stack](docs/10-tech-stack.md) | Technologies, code structure and testing |
| 11 | [Engineering decisions](docs/11-engineering-decisions.md) | Twelve decision records: context, decision, trade-offs |
| 12 | [Taxonomy reference](docs/12-taxonomy-reference.md) | Traits, personas, families of work, roles and industries |
| 13 | [Results and measurements](docs/13-results-and-measurements.md) | Measured improvements, with method and caveats |
| — | [Timeline](TIMELINE.md) | How the product evolved |
| — | [FAQ](FAQ.md) | Common questions |

## Why there's no code here

Dhiti is a commercial product, and the source code is private. This repository is here to explain what the product does and how it was engineered. You won't find source code, prompts, exam content or tuned scoring weights. Publishing any of those would let candidates game the assessment, or let someone copy the product.

<!-- TODO(owner): optional code-access line, e.g. "A code walkthrough is available to hiring teams on request." -->

## Author

<!-- TODO(owner): name and role (e.g. "Founder & sole engineer") -->
<!-- TODO(owner): company or brand line, if a company name should appear -->
<!-- TODO(owner): GitHub profile, LinkedIn and public contact email -->

---

© 2026 <!-- TODO(owner): owner name -->. All rights reserved. See [LICENSE.md](LICENSE.md).
