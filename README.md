# B Game 

B Game is the current working name for a SaaS product built to rethink entrepreneurship training in the age of AI.
This is the public showcase for this SaaS product.

## Why this repository exists

The production codebase is private because it contains business logic, internal prompts, scoring formulas, and a market simulation engine I do not want to open-source.

This repository is meant to document the product:

- the problem B Game solves and the design choices behind it
- how the platform is structured under multi-tenant, competitive pressure
- how AI is integrated, bounded, and made operationally trustworthy
- what engineering standards guide the implementation

## What B Game is

B Game is a digital, multi-team entrepreneurship simulation game designed to place learners in the position of launching a digital solution in a simulated competitive market.

By the end of a session, the learner is able to:

- conduct a user discovery process, empathy, unbiased interviews, insight extraction, aligned with the principles of The Mom Test.
- translate field signals into prioritized needs using product management methods and into a clearly articulated value proposition.
- build a business model, using a simplified Business Model Canvas, that is coherent with the value proposition and check for strategic inconsistencies.
- make financial and operational decisions under constraints, including time budget, cash position, and product/marketing prioritization.
- read market results, identify causes, and adjust strategy across multiple iterations.
- present the project to a simulated external investor and defend the coherence of the application.

Teams move through a structured journey:

1. Personas
2. Interviews and empathy synthesis
3. Need prioritization
4. Complexity assessment (MVP definition)
5. Prototype testing
6. Business model design
7. Financial planning
8. Funding decisions
9. Multi-round market simulation

The product is both:

- a guided learning workflow based on Design Thinking and lean startup principles.
- a deterministic round-based simulation system

## Why this project exists

AI changes what it means to learn a professional skill.

When students can generate answers, slides, business plans, or financial assumptions in seconds, the challenge is no longer just to give them access to tools. The challenge is to design learning environments where AI becomes a lever for reasoning, not a shortcut around it.

B Game was built from that premise.

It explores a different way to train entrepreneurship skills: not by asking students to produce isolated deliverables, but by placing them inside a structured, collaborative, and competitive system where their choices have consequences.

Several design choices follow from this:

- learners work in teams
- the product introduces friction, so AI cannot be used as a passive copy-paste machine
- each step forces tradeoffs, not just task completion
- early decisions propagate into later consequences
- teams compete in a shared market, creating engagement and comparison
- AI provides bounded feedback, but does not replace judgment

The platform guides teams through a complete entrepreneurial journey: personas, interviews, need prioritization, prototype testing, business model design, finance, funding, and competitive market rounds.

The goal is not to tell students whether their idea is good. 

The goal is to help them mobilize judgment, coherence, prioritization, financial reasoning, user understanding, and strategic tradeoffs in a context where AI is available but not sufficient.

B Game treats AI as part of the learning environment, not as the answer engine.

## Field-tested V1

The first version of B Game has been field-tested with 75 business school students during 6-hour sessions in their entrepreneurship major.

This real classroom usage helped validate several core learning mechanics:

- collaborative work as the default learning mode
- enough friction to prevent passive copy-paste use of AI
- system thinking and tradeoffs over isolated task execution
- competitive pressure to increase engagement

The takeaway is that AI-enabled education does not only require access to better tools. It requires learning environments where AI is available, but where students still have to reason, decide, commit, and face the consequences of their choices.

## Product highlights

Because B Game is built as a learning environment, the system has to be more than a chat interface or a content generator.

It needs to:
- make student decisions visible
- preserve the consequences of those decisions
- prevent the workflow from being optimized through copy-paste
- keep AI feedback bounded and non-prescriptive
- make outcomes explainable to teachers or coaches
- keep market results reproducible across runs

This is why the implementation is built around:

- strict multi-tenant isolation
- role-based permissions
- Supabase RLS policies
- typed service boundaries
- schema-validated AI outputs
- deterministic simulation logic
- frozen round snapshots
- admin supervision and auditability
- Strict separation between UI, actions, services, and DAL

## Engineering snapshot

A few properties of the private codebase, chosen because they say something:
- market simulation is fully deterministic and verified by golden tests on a stable input set
- financial logic uses integer arithmetic end-to-end; no native float touches money
- tenant isolation is enforced both in application code and in database row-level policies, with 38 migrations supporting that evolution
- AI provider exhaustion (primary and fallback both failing) is a covered test case, not an emergent one
- round results are append-only, with `update` and `delete` intentionally not exposed at the database level

Code-wise, the project is around ~480 files under `web/app` and `web/lib`, with 80 test files across services, AI boundary behavior, finance, DAL, and simulation.

## Tech stack

- Next.js 14 App Router
- TypeScript strict mode
- Tailwind CSS
- shadcn/ui
- Supabase Postgres + Auth + RLS
- Zod
- OpenAI-based AI layer behind a centralized wrapper
- Vercel

## Architecture principles

- `UI -> Actions -> Services -> DAL`
- No database access from UI
- No business logic in route components
- No AI calls outside the dedicated AI layer
- No student `teamId` in URLs
- Team context always derived server-side

More detail:
- [PRODUCT_DESIGN.md](./PRODUCT_DESIGN.md)
- [AI_DESIGN.md](./AI_DESIGN.md)
- [WORKING_METHOD.md](./WORKING_METHOD.md)
- [ARCHITECTURE.md](./ARCHITECTURE.md)
- [CODE_EXCERPTS.md](./CODE_EXCERPTS.md)

## AI usage philosophy

The AI layer is not used as a black box that "decides" for the user.

Key design choices:

- prompts are centralized
- outputs are schema-validated
- unsafe or malformed outputs fail closed
- provider routing supports a primary model, an optional fallback provider, and a forced-provider override for incidents and testing
- prototype feedback follows an "AI mirror" posture rather than prescriptive advice
- product logic stays outside the model layer

This matters because in enterprise settings, useful AI is rarely about raw generation. It is about bounded behavior, predictable integration, and operational trust.

## Selected public-safe code excerpts

The internal repository remains private, but I included a small set of trimmed excerpts that expose the product's key boundaries and guarantees:

- mutation discipline
- server-side tenant scoping
- service and DAL separation
- input and AI-output validation
- provider routing and degraded mode
- deterministic simulation
- snapshot-based historization

See [CODE_EXCERPTS.md](./CODE_EXCERPTS.md).
