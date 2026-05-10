# AI Design

This document describes how AI is integrated into B Game.

It is not a prompt cookbook. The exact prompts, the full catalog of behavioral traits, and the scoring formulas are kept out of the public showcase on purpose. What this document covers is the *design* of the AI subsystem: roles, boundaries, contracts, and operational guarantees.

## 1. Design intent

The product does not rely on AI to "decide" anything that affects a team's outcome.

AI is used where it adds leverage that no other component can provide at scale: simulating realistic user feedback during interviews, producing structured signals on a prototype, or auditing the coherence of a funding application. Everything else, pricing logic, market simulation, scoring rules, permission checks, stays rule-based and deterministic.

The intent is simple:

- AI improves the quality of the workflow at specific touchpoints
- AI never becomes a source of unbounded influence on the simulation outcome
- AI failure modes degrade the product gracefully, never silently

## 2. The four AI roles

B Game uses AI in four distinct roles. Each role has its own contract.

### 2.1 AI as simulated user (interview persona)

When a team interviews a persona, an AI plays the role of that persona.

What this role does:

- responds in character to the team's questions
- resists hypothetical or solution-oriented questions
- rewards questions anchored in the persona's past behavior
- can be vague, evasive, hesitant, or short, depending on hidden behavioral state

What this role never does:

- validate the team's idea
- give advice
- explain the rules of the workflow
- describe its own behavior

The persona has an internal behavioral state that the team cannot see. That state is generated deterministically when the persona is created, so the same persona always behaves the same way in interviews. This protects the exercise from being gamed across attempts.

The interview surface also includes anti-abuse guards. Meta-questions about how the AI works are treated as in-character incomprehension. Aggressive or off-topic questions end the interview, and the team loses the corresponding interview slot.

### 2.2 AI as product reviewer (prototype signal)

When a team submits a prototype for testing, an AI reviews the screens and produces a structured signal.

The signal is descriptive, not prescriptive. It reports what a typical user would understand on each screen, what would remain unclear, what spontaneous question the screen would trigger, and whether each screen has a single clear focus.

Two properties of this role matter for product trust:

- **Bounded output.** The signal follows a strict schema. Free-form opinions are not part of the contract.
- **No advice.** The role is forbidden from telling the team what to fix. It mirrors back what is observable on the screen and stops there.

The signal is then consumed by the deterministic scoring layer. It does not directly affect the simulation outcome.

### 2.3 AI as funding auditor

When a team submits a funding application, an AI audits the coherence of the application against the rest of the team's work: did the verbatims justify the diagnosed pains? Do the prioritized needs follow from those pains? Does the product actually address those needs?

This role produces sub-scores along the coherence chain, plus a short structured rationale. The output is captured at submission time and frozen. The funding decision logic reads from that frozen audit. The AI is not called again at decision time.

This freeze property is important. It guarantees that the funding decision does not depend on a model's mood at the wrong moment.

### 2.4 AI as supervision aid for the teacher/coach

A separate, admin-only path produces a synthesis that helps the teacher/coach understand quickly where a given team performed well or poorly during the empathy stage.

This role is read-only with respect to the workflow. It produces a summary intended for human review. It does not influence student-facing scores or the simulation.

## 3. Cross-cutting guardrails

The same set of guardrails applies to every AI surface in the product.

### 3.1 Schema-validated outputs

Every AI response is parsed against a strict schema before it is allowed into product state. A response that does not match the schema is treated as a failure, not as raw text to be displayed. There is no "best effort" rendering of malformed output.

### 3.2 Anti-prescriptive contract

For roles that talk to students (interview persona, prototype reviewer), the system enforces an anti-prescriptive policy. Output that crosses into advice is rejected.

This is a product policy, not a stylistic one. The pedagogical value of the workflow depends on students owning their decisions. AI that offers solutions undermines that.

### 3.3 Fail-closed on unsafe inputs

For surfaces that consume images (prototype review), an upstream validation step decides whether the input is usable. If the validation fails, no AI signal is produced and the underlying state is rolled back. The system never lets a degraded input reach the analysis stage.

### 3.4 Provider routing

The AI layer sits behind a single internal interface. Concretely:

- one primary provider in normal operation
- one optional fallback provider, enabled by configuration
- one explicit override for incident handling, debugging, or behavior validation

If the primary provider fails, the fallback is attempted with the same prompt and the same schema. If both fail, the system returns a stable, named error rather than degrading into unstructured text.

This is tested. Provider exhaustion is a covered case, not an emergent one.

### 3.5 Determinism where determinism matters

Some AI-adjacent components have to behave deterministically to preserve product trust. These components do not depend on the model itself : they depend on stable seeds, ordered inputs, and pure functions.

Hidden state for personas is generated deterministically from a seed tied to the session, the team, and the persona. The same persona, in the same context, always behaves the same way. This is what allows replay and audit to mean something.

### 3.6 No leakage into logs

The AI layer sanitizes provider errors before logging. Signed URLs are never written to logs. Long error payloads are truncated. Only a small set of safe fields is preserved.

## 4. What students see and what they do not

A central design choice in the AI layer is the asymmetry of visibility.

Students see:

- the persona's in-character responses
- the prototype review signal in its descriptive form
- the funding decision and its high-level rationale

Students do not see:

- the persona's internal behavioral state
- alignment-level diagnostics on the prototype (admin-only)
- AI errors when a fallback signal is produced (logged, not surfaced as raw)
- their own insight score, to prevent score-gaming

Admins see significantly more, including diagnostic fields that let them understand whether a team's result is anchored in the team's actual work or in an edge case.

This asymmetry is intentional and is part of why the product behaves like a real environment rather than a tutoring system.

## 5. What the AI layer never does

To make the boundaries explicit:

- the AI layer does not write to the database directly
- the AI layer does not call business services
- the AI layer is not invoked from UI components
- the AI layer is not invoked from server actions outside its dedicated module
- the AI layer does not make decisions that affect the simulation outcome
- the AI layer does not generate content that bypasses schema validation

## 6. Why this design matters

- can we explain every output the AI produces?
- can we reproduce a result given the same inputs?
- does the AI fail in a way we can detect and contain?
- can we swap the provider without rewriting the product?
- can we be sure the AI does not silently drift the rest of the system?

The B Game AI layer is designed so that the answer to each of those questions is yes.

The implementation details are not what matters most here. What matters is that the design has been forced through the same constraints that all enterprise AI integrations face: structured contracts, bounded behavior, operational visibility, and graceful degradation.
