# 09 · Security, privacy and integrity

Anything sensitive in Dhiti happens on the server: data access, pricing and exam scoring. Browsers never talk to the database directly. Every paid feature needs a valid paid session owned by the caller. The exam's answer key and deadline stay on the server, and payment amounts only ever come from prices computed there. This page describes those protections at the design level.

## Data access

The database is closed to browsers. Every read and write goes through a server route that checks who the signed-in user is.

The reason is how login works. Sign-in is handled by NextAuth.js rather than the database's own auth system, so the database has no browser identity it could write access rules against. Routing everything through the server solves that. On the server side, Dhiti reaches Google Cloud Firestore through the Firebase Admin SDK.

## Access to paid features

Access is credit-based, and one credit is one assessment session. Every paid, LLM-backed route requires a valid paid session owned by the caller.

Each session also has ceilings on its LLM calls. They're set several times above real use and put a bound on the worst-case cost of any session. If one is tripped, the operators are alerted. See [Reliability, cost and performance](08-reliability-cost-performance.md#per-session-llm-call-ceilings).

## Accounts

Candidates sign in with email and password, or with Google. Passwords are hashed with bcrypt.

Signing up by email means confirming a **6-digit code**. The code:

- expires after **10 minutes**
- allows at most **5 attempts**
- can only be resent after a **60-second cooldown**

Verification emails are rate-limited each hour, both per email address and per IP address. The sign-up form also catches common typos in email domains and suggests a correction. Sign-up codes are sent from a dedicated, authenticated domain mailbox, separate from operational email, so they land in the inbox.

## Payments

Payments go through **Razorpay**. **Prices are always computed on the server**, and the client never gets a say in the amount. Pricing is regional (India and international), based on the visitor's location. Coupons are checked against the account that's paying, and Indian GST is added to INR charges.

Colleges and enterprises go through a lead form instead, and the team follows up.

## Exam integrity

| Protection | How it works |
|---|---|
| **Answer key held on the server** | The browser never receives correctness or rationales before the candidate answers. The full review is only released after scoring. |
| **Deadline set by the server** | The 60-minute deadline is recorded on the server when the candidate starts. Starting late shortens the exam and never extends it. A generous grace window protects honest candidates whose tab slipped into the background. |
| **Score is final** | Scoring is deterministic (exact set match) and happens once. No further answers are accepted after that. |
| **Focus loss recorded, not scored** | The number of times the candidate leaves the exam tab is stored as soft integrity metadata. It doesn't affect the score. |
| **Answer key can't drift** | Personalised rewrites are validated in code, so they can't change which options are correct. See [Scenario exam engine](06-scenario-exam-engine.md#correctness-is-locked). |
| **Content isn't published** | Scenarios, options, rationales and tuned weights are kept out of all public material, this repository included. |

## Regulated roles

Two roles need extra care: **clinical psychologist** and **wellness coach**. Their scenarios are deliberately limited in scope. They test professional, ethical and practice-management judgment, using only fictional, de-identified cases. Results are never presented as clinical competence, licensure or certification.

## Not professional advice

Dhiti is a **career guidance tool**. It doesn't give medical, psychological or financial advice.

## Privacy

Dhiti's privacy policy is published on the live site, [www.dhiti.ai](https://www.dhiti.ai).

<!-- TODO(owner): link the privacy policy at https://www.dhiti.ai and summarise what data is stored and for how long, if you want that stated here -->

## See also

- [System architecture](03-system-architecture.md#why-the-database-is-closed-to-browsers)
- [FAQ](../FAQ.md)
