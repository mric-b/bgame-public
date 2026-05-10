# Working Method

## AI-assisted product operating model

I use AI as a structured operating system for product work, not as a shortcut for avoiding thinking.

The goal is to increase throughput while improving clarity, challenge quality, and implementation discipline.

## My role

I am the CPO.

That means I own:
- product direction
- user value
- scope decisions
- final tradeoffs
- coherence across the whole platform

AI helps me think faster and challenge assumptions, but I remain the decision-maker.

## How I use different models

### 1. ChatGPT as CTO sparring partner

I use ChatGPT as a technical and product counterpart when shaping features.

Typical use:

- clarify feature scope
- stress-test implementation choices
- challenge edge cases
- make architecture and product decisions more explicit

Role in my process:

- feature framing
- system thinking
- product-to-technical translation

### 2. Gemini as a strategic committee

I use Gemini as a structured strategic review layer before committing to a feature direction.

I generally frame it as a small committee made of:

- a business game pedagogy specialist
- a design thinking expert
- an early-stage investor

Depending on the topic, I may add extra specialists such as digital marketing.

What I expect from this step:

- challenge product relevance
- check whether a feature still serves the core vision
- avoid adding complexity that looks smart but weakens the product

### 3. Claude or Copilot as lead development partner

I use Claude or Copilot closer to implementation.

Typical use:

- define the roadmap
- turn a validated feature definition into code
- accelerate iteration once scope is clear

This works best because the product and architectural intent have already been sharpened upstream.

## Why I work this way

I have found that using several models with explicit roles is more useful than using one model for everything.

The benefits are:

- better challenge quality
- less accidental tunnel vision
- clearer separation between strategy, product definition, and implementation
- faster iteration without losing control of the vision

## Operating model

The workflow is deliberately staged.

Each stage has a different goal, a different AI role, and a different markdown artifact.

| Stage | Goal | AI setup | Output |
|---|---|---|---|
| Define | Frame the feature, constraints, and edge cases | ChatGPT as CTO counterpart, myself as CPO / keeper of vision | Feature framing markdown |
| Validate | Challenge product relevance and strategic fit | ChatGPT + Gemini as strategic review board | Strategic review markdown |
| Plan | Turn the validated direction into an implementation path | ChatGPT + Claude as lead development partner | Roadmap / sprint brief markdown |
| Implement | Build, review, and stabilize the feature | Claude / Copilot for implementation, ChatGPT for code review | Sprint delivery notes / recap markdown |

The point is not to ask every model the same question.

The point is to give each model enough context to contribute, but not so much that it simply reinforces the previous answer. This keeps the process useful as a source of contradiction, clarification, and execution discipline.

## Sprint-based delivery

The project was built in product sprints, close to how I worked with previous product and engineering teams.

A sprint was not a ticket batch. It was a coherent product capability. 

Each sprint usually covered:
- the pedagogical objective
- the user journey and UI states
- business rules and edge cases
- data model changes
- service, DAL, and action responsibilities
- AI boundaries when relevant
- tests and validation criteria
- known limitations, non-goals, and follow-up debt

This kept the work at the right level of granularity: large enough to deliver meaningful product value, but bounded enough to be specified, implemented, tested, and documented.

Each sprint produced markdown artifacts that made the work reviewable before, during, and after implementation: framing, strategic review, implementation roadmap, technical notes, and delivery recap.

## What this says about how I build

It is a deliberate way to:

- structure product thinking
- confront ideas before shipping them
- use AI as leverage inside a disciplined workflow
- stay accountable for the end result

This operating model is also why the project treats AI integration as an organizational problem, not only a technical one: the value comes from assigning roles, preserving accountability, and turning model outputs into reviewable artifacts.
