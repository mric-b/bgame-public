# Product Design

This document explains the product reasoning behind B Game.
It is not a feature list. This document focuses on the design choices that shape the product.

## 1. The problem B Game is built around

Most business games are linear. They teach how to fill in spreadsheets and follow a predetermined storyline.

That is not how a real market works. This is not the way entrepreneurship works. 

A real market does not care about a team's intent, narrative, or methodology. It reacts to the choices that are actually made. Two teams with the same idea can end up in very different places depending on how their choices fit together.

B Game is built around that reality.

The product is designed so that:

- the student is forced to make decisions under partial information
- every decision has costs and benefits
- the system never tells a student that an idea is "good" or "bad"
- the only judge is a simulated market shared by all teams

This is not a stylistic choice. It is the foundation of the product.

## 2. Three core design principles

### 2.1 The system judges coherence, not ideas

The product never evaluates whether a persona is "the right one", whether an idea is "original", or whether a business model is "credible".

What it evaluates is whether a team's choices hold together:

- does the empathy work justify the prioritized needs?
- does the prototype actually address the pains identified?
- does the pricing match the promise?
- does the marketing allocation match the strategic narrative?

The product surfaces inconsistencies. It does not surface opinions.

This is intentional. It removes the temptation to optimize for what the system "wants to see", and forces students to defend their own logic.

### 2.2 The market is deliberately imperfect, and identical for everyone

The simulated market is not a model of any specific industry. It is a generic competitive environment with realistic levers (price, UX, marketing, churn) and a fixed total demand per session.

It is intentionally:

- blind to intent
- simplified
- sometimes counter-intuitive
- the same for every team in a session

This matters because the comparison between teams is meaningful. A team's performance is not absolute. It is relative to the choices made by the other teams in the same market.

Lowering price does not help if every competitor lowers it too. Spending more on marketing does not help if the offer is unclear. The product makes those tradeoffs visible.

### 2.3 No randomness after a round closes

Once a round is closed, the result is fully determined by the inputs.

The simulation engine is a deterministic function of the team snapshots taken at close. There is no randomness applied on top of the result, no hidden tie-breaker that varies between runs, no "luck" injected into the outcome.

This is a deliberate product property:

- it preserves educational credibility (a team can always be told why it got a given result)
- it enables full replay and audit
- it prevents the system from masking design flaws as "bad luck"

The student-facing surface absorbs uncertainty earlier in the workflow (through hidden persona traits, market reactions, and competitor visibility). The simulation itself stays predictable.

## 3. Why this matters beyond education

The same principles apply directly to any operational software where AI is used inside a real workflow:

- protect the system from being optimized against (hidden state, bounded outputs)
- make decisions and their consequences visible without prescribing what to do
- separate exploration (free, low-stakes) from commitment (frozen, auditable)
- ensure that automated outcomes can be explained and reproduced

B Game is a useful illustration of those principles because the constraints are real:

- multiple teams compete in the same simulated market at the same time
- teachers need to defend results in front of students
- the product cannot be allowed to drift, even slightly, between two runs of the same input

## 4. The student journey, seen as a product

The student workflow is not a sequence of forms. It is a sequence of commitments, each one constraining the next.

| Stage | What it does in product terms |
|---|---|
| Personas | Forces the team to write down hypotheses, not facts |
| Empathy interviews | Confronts hypotheses with simulated user behavior |
| Need prioritization | Forces tradeoffs between desire and discipline |
| Complexity assessment (MVP definition) | Anchors ambition into feasibility |
| Prototype | Tests whether the product matches the diagnosed pains |
| Business model | Forces the team to make its monetization explicit |
| Finance | Translates strategy into a 12-month cash trajectory |
| Funding | Confronts the team's narrative with an external investor |
| Market rounds | Confronts the team's choices with the market |

Two product properties hold across the journey:

- **explorations are reversible, commitments are not.** Students can iterate freely until they choose to validate or to launch a round. Once committed, the state is frozen.
- **early choices propagate.** Errors at the empathy stage have measurable consequences several stages later. The product does not forgive that retroactively, but it does explain it.

## 5. The role of friction

The product deliberately introduces friction at specific moments:

- the persona refuses to validate the team's idea, no matter how it is phrased
- a question that pitches a solution counts against the team
- a "Premium" complexity choice closes off cheap development modes
- frozen needs cannot be modified once a round closes
- the simulation result is shown only after the round, with explanations

This friction is not a usability defect. It is the product.

A product that always says yes is comfortable but useless for learning. A product that says yes only when the team's logic actually holds together is uncomfortable but transformative.

## 6. The role of AI inside the product

AI is not the product. The product is the workflow. AI is one of the components that operates inside the workflow.

This is described in detail in [AI_DESIGN.md](./AI_DESIGN.md), but the product-level summary is:

- AI is used only at points where bounded human-style feedback adds clear leverage
- AI never replaces the rules of the workflow
- AI never decides outcomes; it produces structured signals that the workflow consumes
- AI is held to a stricter contract than students

That last point is important. The system tolerates clumsy student questions. It does not tolerate unbounded model output entering product state.

## 7. Why these constraints matter

B Game is designed as a learning environment, where results have to be understandable.

That means the system must preserve trust:

- learners must understand that outcomes come from their choices
- teachers and coaches must be able to explain and defend results
- AI feedback must support reflection without giving away the answer
- market simulation must be reproducible once a round is closed
- committed inputs must remain auditable over time

These constraints shape the product architecture. AI is used where it improves realism and feedback quality, but the workflow remains structured, deterministic where needed, and supervised by teachers and coaches.
