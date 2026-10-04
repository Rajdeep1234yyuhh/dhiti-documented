# 01 · Product overview

Dhiti is an AI career counsellor delivered as a web app. It gets to know a candidate through conversation, takes the limits of their life into account, and recommends roles that will hold up as AI changes work. Then it tests the candidate on realistic scenarios from the role they pick. This page covers what the product does, who it's for and the principles behind it.

**Try it live at [www.dhiti.ai](https://www.dhiti.ai).**

<!-- TODO(owner): confirm whether to embed or link the product video (https://youtu.be/aIo-PEiSqQA) -->

## What Dhiti does

Everything starts with a conversation about real moments from the candidate's life. From there, Dhiti:

1. Scores them on **18 traits**. Each score has a confidence level and evidence quoted from what they said. ([Trait assessment](04-trait-assessment.md))
2. Asks three quick questions about the practical limits on their options: weekly hours available, whether others rely on their income, and whether they can relocate.
3. Ranks **30 future-resilient roles in 6 families of work**, and explains which roles were set aside and why. ([Career matching pipeline](05-career-matching-pipeline.md))
4. Tests them on **25 realistic workplace scenarios** from the role they pick, rewritten for the industry they choose. ([Scenario exam engine](06-scenario-exam-engine.md))
5. Writes a report with their best-fit roles, what was ruled out and why, how they did on the real work, and a staged roadmap. ([Report and roadmap](07-report-and-roadmap.md))

## Who it's for

People come to Dhiti at very different stages of life. Dhiti places each candidate in one of four personas and shapes the conversation to fit. Treat the age ranges as rough guides.

| Persona | Typical age | Who they are |
|---|---|---|
| Career Discovery | about 16–22 | Students and recent graduates exploring what fits |
| Career Validation | about 21–27 | Early-career people second-guessing a chosen path |
| Career Transition | about 24–35 | Working professionals switching without wrecking what they've built |
| Reinvention & Independence | about 35–45 | Experienced people building work on their own terms |

Colleges and enterprises can get in touch through a lead form on the pricing page, and the team follows up.

## Product principles

These principles shape how the product behaves, and they also shaped a lot of how it's built.

- **Curiosity first, setbacks last.** The first trait question is about something the candidate went deep on by choice. Questions about failure wait until the end of the first round, when there's some rapport.
- **"I don't know" is an answer.** Short, unsure and half-formed answers all count. Dhiti only re-asks when an answer is gibberish or off-topic, and never more than twice for the same question.
- **It asks how urgent this is.** Is the candidate just exploring, planning for the next 6–12 months, needing to move within 1–3 months, or in a crisis right now? Tone and plan adapt to the answer. Someone in crisis gets one steady next step, not a list.
- **It reads the room.** Dhiti adjusts its tone to the candidate's state, whether they're confused, afraid, burnt out, ambitious, curious or under pressure.
- **It's honest about how sure it is.** Every trait score carries a confidence level. Where the evidence is thin, the report says "you showed signs of" rather than "you are".
- **Your life is part of the answer.** Roles that don't fit the candidate's hours, finances or ability to relocate are set aside, and the report explains why.
- **You're tested on the real work.** A candidate who isn't sure which role to pick is tested across their top three.
- **It shows its working.** All six families of work are scored. The report explains what wasn't recommended and why. It frames this as a question of profile and constraints, never as those roles being lesser.

## What makes it different

Trait scores are grounded in what the candidate actually said. They come from the real moments the candidate described, and each score's evidence quotes those moments back in the second person ("You described…"). Every score also carries a confidence level, and the report softens its language wherever the evidence is thin.

The candidate's constraints shape the result too. A role can be a strong trait match and still be set aside, if the move into it doesn't fit their time, money or ability to relocate. When that happens, the report says so.

Dhiti also doesn't stop at a recommendation. The candidate works through 25 scenarios drawn from the role's actual decisions, set in the industry they chose, and afterwards reviews the rationale for every option.

Finally, the recommendations are shown, not just stated. The candidate sees which families and roles were ruled out, and why.

## Access

Sign-in is with email and password, or with Google. Access is credit-based: **one credit is one assessment session**. It's a paid product, with regional pricing.

## What Dhiti is not

Dhiti is a career guidance tool. It doesn't give medical, psychological or financial advice. Scenarios for regulated roles, such as clinical psychologist, are deliberately limited in scope (see [Security, privacy and integrity](09-security-privacy-integrity.md#regulated-roles)).

## See also

- [User journey](02-user-journey.md): the session from the candidate's side
- [Taxonomy reference](12-taxonomy-reference.md): every trait, role, family and industry
