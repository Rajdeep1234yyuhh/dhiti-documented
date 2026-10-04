# 10 · Tech stack

Dhiti is written in TypeScript on Next.js 15 and React 19. It runs as Vercel serverless functions, uses Anthropic Claude for anything involving language, and keeps its data in Google Cloud Firestore. The code is organised around a pure, dependency-injected core with a thin server layer handling the I/O, and that's what makes it testable without a database. Content is kept as versioned data, not as database rows.

## Technologies

| Layer | Technology | Used for |
|---|---|---|
| Framework | **Next.js 15** (App Router, Turbopack in development) | Pages, routing and API routes |
| UI | **React 19**, **TypeScript** | Interface |
| Styling | **Tailwind CSS 4** | Styling, with light and dark themes |
| Motion | **Framer Motion**, **GSAP** | Interface animation |
| Charts | **Chart.js** | Charts in the interface |
| Hosting | **Vercel serverless functions** | API routes, with long-running routes up to 300 s |
| AI | **Anthropic Claude Sonnet 4.6** | Every LLM call in the assessment |
| AI (older flow) | **Anthropic Claude Haiku 4.5** | An older, survey-style flow |
| Data | **Google Cloud Firestore**, via the **Firebase Admin SDK** | All persistence, accessed only from the server |
| Auth | **NextAuth.js** | Email and password, or Google sign-in |
| Password hashing | **bcrypt** | Stored credentials |
| Payments | **Razorpay** | Payments, with server-computed regional prices |
| Email | **SMTP via Nodemailer** | Sign-up codes and other email |
| Speech | Browser speech recognition | Voice dictation in chat |
| Testing | **Vitest** | Unit tests |

## Code structure

### Pure, dependency-injected core

These parts of the core are written as pure modules, with their dependencies passed in:

- pipeline orchestration
- quota maths
- session activity
- metrics aggregation
- LLM pricing
- email validation

The database, the LLM client and the quota checks are injected, not imported. That means the core can be unit-tested with no database and no network. Around it sits a thin server layer that does the I/O. It reads the request, checks identity and session ownership, plugs in the real dependencies, and writes the response.

The problem this solved: logic that talks straight to a database and an LLM is slow and brittle to test. With injection, a test just passes in stand-ins for the database and the LLM, then checks exactly what the orchestration did. For example, it can confirm that a superseded run stops writing.

### One canonical trait registry

Every representation of a trait comes from a single registry row. If a trait gets left out somewhere, that's a **compile error**, not a silent bug. See [ADR-09](11-engineering-decisions.md#adr-09-one-canonical-trait-registry).

### Content as versioned data

Role packs, the capability library and the knowledge corpus are JSON files in version control, so changing content means a reviewable diff. Offline scripts batch-generate content and clean up phrasing. One cleanup across the whole bank replaced templated phrasing in the seed scenarios. Everything those scripts generate or rewrite has to pass the same validation as live generation before it's merged.

## Testing

There are **about 170 automated test cases across 15 test files**, run with Vitest. Here's what they cover:

| Area | What's tested |
|---|---|
| Server-side pipeline | Orchestration, checkpoint reuse, supersession, quota refusal |
| Streaming | The streaming event protocol |
| Exam | The exam generation lock |
| Pipeline state | The pipeline state reducer |
| Quotas | Session quotas, user quotas, quota alerts |
| Session activity | Activity-write throttling |
| Email | Email-address validation, email verification, sign-up mail |
| Metrics | Metrics aggregation |
| Cost | LLM cost and pricing maths |

## See also

- [System architecture](03-system-architecture.md)
- [Engineering decisions](11-engineering-decisions.md)
