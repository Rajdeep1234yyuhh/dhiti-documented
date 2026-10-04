# FAQ

Quick answers to the questions people ask most about Dhiti. If you want more depth, each answer links into the [documentation](README.md#documentation).

**Want to try it yourself?** Dhiti is live at **[www.dhiti.ai](https://www.dhiti.ai)**.

## About the product

### Where can I try Dhiti?

Go to **[www.dhiti.ai](https://www.dhiti.ai)**. It runs in your browser, so there's nothing to download or install. You sign in with email and password or with Google, and one credit gets you one full assessment session.

### What is Dhiti?

Dhiti is an AI career counsellor that works as a web app. It gets to know you through a conversation and scores 18 traits, each with a confidence level. It takes your constraints into account and ranks 30 future-resilient roles. Then it tests you on 25 workplace scenarios from the role you pick, and gives you a report with a roadmap. See [Product overview](docs/01-product-overview.md).

### Is this psychological or medical advice?

No. Dhiti is a career guidance tool. It doesn't give medical, psychological or financial advice.

### How long does a session take?

<!-- TODO(owner): typical total session length -->

The exam on its own takes up to 60 minutes.

### What if I don't know which role or industry to pick?

Just choose "I'm not sure". If it's the role you're unsure about, the 25 scenarios are split as evenly as possible across your top three roles, and whichever one you score best on is used for your roadmap. If it's the industry, you get the scenarios in their generic, industry-neutral form.

### What if my answers are short, or I don't know?

That's completely fine. Short, unsure and half-formed answers all count. Dhiti only asks again if an answer is gibberish or off-topic, and never more than twice for the same question.

### What do I get at the end?

You get a report. It covers your best-fit roles, what was ruled out and why, how you did on the exam, an AI-resilience summary and a three-phase roadmap (Foundation → Build → Launch). You can export it to PDF or have the results emailed to you. See [Report and roadmap](docs/07-report-and-roadmap.md).

## The exam

### Is the exam marked by AI?

No. Scoring is deterministic. A scenario counts as correct only if the options you selected exactly match the correct set, and there's no partial credit on "select N" questions, just like real certification exams. The AI's job is to rewrite scenarios into the industry you chose, and checks in code make sure it can't change which answers are correct. See [Scenario exam engine](docs/06-scenario-exam-engine.md).

### Can I see the correct answers?

Yes, once your exam has been scored. You can go back through every scenario and see your answer, the correct answer and a rationale for every option.

### Does leaving the exam tab affect my score?

No. The number of times you leave the tab is recorded, but it isn't used in scoring. The 60-minute deadline is set by the server, and there's a generous grace window, so a tab that slipped into the background for a moment won't count against you.

### Why aren't example questions published?

If real scenarios or answers were published, candidates could prepare for the exact questions, and the exam would stop meaning anything.

## Access

### Does it work outside India?

Yes. Pricing is regional.

### Can colleges or companies use it?

Yes. Get in touch through the lead form on the pricing page at [www.dhiti.ai](https://www.dhiti.ai).

### How do I sign in?

With email and password, or with Google. If you sign up by email, you'll confirm it with a 6-digit code.

### Is my data shared?

<!-- TODO(owner): link the privacy policy at https://www.dhiti.ai -->

## Technology

### Which AI model does it use?

Anthropic Claude. Claude Sonnet 4.6 handles every LLM call in the assessment. See [Tech stack](docs/10-tech-stack.md).

### Is the code open source?

No. Dhiti is a commercial product, and its source code is private. This repository explains what it does and how it was engineered, without source code, prompts, exam content or tuned weights.

<!-- TODO(owner): optional code-access line, e.g. "A code walkthrough is available to hiring teams on request." -->

### Why is there no installation guide?

Because there's nothing to install. Dhiti runs in the browser at [www.dhiti.ai](https://www.dhiti.ai), and this repository is documentation only.
