# Code Excerpts

These excerpts are selected to expose the structure and guarantees of the product without exposing sensitive business logic.

The goal is to show where the boundaries are:

- where tenant isolation is enforced
- where business logic lives
- where AI is allowed to act
- how simulation stays reproducible
- how the data model preserves history

## Reading map

| Concern | Excerpts |
|---|---|
| Mutation contract | 1, 2 |
| Tenant isolation | 2, 3, 5 |
| Business rules | 4 |
| Input validation | 6 |
| AI guardrails | 7, 8 |
| Controlled AI variability | 8 |
| Provider resilience | 9 |
| Deterministic simulation | 10 |
| Historical auditability | 11 |

## 1. Standardized action wrapper

Source pattern: `web/lib/actions/actionUtils.ts`

```ts
export type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; error: string; code?: string };

export async function executeAction<T>(
  actionFn: () => Promise<T>
): Promise<ActionResult<T>> {
  try {
    const data = await actionFn();
    return { success: true, data };
  } catch (error) {
    const message =
      error instanceof Error ? error.message : "An unexpected error occurred.";
    const code =
      error instanceof Error
        ? (error as Error & { code?: string }).code
        : undefined;
    return { success: false, error: message, ...(code && { code }) };
  }
}
```

Guarantee demonstrated:

- mutation results have a stable UI-facing contract
- errors are normalized across server actions

## 2. Thin controller action

Student actions receive `sessionId` and form input only. They never receive `teamId` from the client.
Source pattern: `web/app/(protected)/student/[sessionId]/besoins/actions.ts`

```ts
export async function addNeedAction(
  sessionId: string,
  input: unknown
): Promise<ActionResult<Need>> {
  return executeAction(async () => {
    const { teamId } = await requireTeamContextForSession(sessionId);

    const parsed = addNeedInputSchema.safeParse(input);
    if (!parsed.success) {
      throw new Error(parsed.error.issues[0]?.message ?? "Invalid input.");
    }

    const result = await needsService.addNeed(sessionId, teamId, parsed.data);

    revalidatePath(`/student/${sessionId}/besoins`);
    return result;
  });
}
```

Guarantee demonstrated:

- keep the action as a controller, not a business layer
- show validation, permission resolution, service call, and UI invalidation in one place

## 3. Team context resolved server-side

Source pattern: `web/lib/auth/sessionContext.ts`

```ts
export async function requireTeamContextForSession(
  targetSessionId: string
): Promise<SessionContext & { teamId: number }> {
  const context = await getSessionContextForSession(targetSessionId);

  if (!context) {
    throw new Error(SESSION_CONTEXT_ERRORS.UNAUTHENTICATED);
  }

  if (context.teamId == null) {
    throw new Error(SESSION_CONTEXT_ERRORS.TEAM_REQUIRED);
  }

  return context as SessionContext & { teamId: number };
}
```

Guarantee demonstrated:

- avoid trusting client-supplied tenant identifiers
- make tenant boundaries explicit at the action boundary

## 4. Business rule in the service layer

Source pattern: `web/lib/services/needsService.ts`

```ts
export async function addNeed(
  sessionId: string,
  teamId: number,
  input: CreateNeedInput
): Promise<Need> {
  await assertEmpathyValidated(sessionId, teamId);

  let painId: number | null = null;
  if (input.justificationType === "PAIN") {
    if (!input.painId) {
      throw new Error(NEEDS_SERVICE_ERRORS.PAIN_REQUIRED);
    }
    await assertPainBelongsToPersona(
      sessionId,
      teamId,
      input.painId,
      input.personaId
    );
    painId = input.painId;
  }

  return needsDb.createNeed(sessionId, teamId, {
    persona_id: input.personaId,
    need_text: input.needText,
    justification_type: input.justificationType,
    pain_id: painId,
    priority: input.priority ?? null,
  });
}
```

Guarantee demonstrated:

- keep domain invariants outside UI and DAL
- make business rules explicit and testable

## 5. Typed DAL scoped by tenant context

Source pattern: `web/lib/db/needs.ts`

```ts
export async function listNeedsByTeam(
  sessionId: string,
  teamId: number,
  filters?: {
    personaId?: number;
    priority?: "MUST" | "SHOULD" | "COULD" | "WONT" | null;
    isFrozen?: boolean;
  }
): Promise<Need[]> {
  const supabase = createUserClient();

  let query = supabase
    .from("needs")
    .select()
    .eq("session_id", sessionId)
    .eq("team_id", teamId)
    .order("created_at", { ascending: true });

  if (filters?.personaId !== undefined) {
    query = query.eq("persona_id", filters.personaId);
  }

  if (filters?.priority !== undefined) {
    query =
      filters.priority === null
        ? query.is("priority", null)
        : query.eq("priority", filters.priority);
  }

  if (filters?.isFrozen !== undefined) {
    query = query.eq("is_frozen", filters.isFrozen);
  }

  const { data, error } = await query;
  if (error) throwDbError("listNeedsByTeam", error);
  return data ?? [];
}
```

Guarantee demonstrated:

- keep DB access centralized and typed
- every query is scoped by `session_id` and `team_id`
- tenant filtering is centralized in the DAL

## 6. Input validation contract

Source pattern: `web/lib/schemas/needs.ts`

```ts
export const addNeedInputSchema = z.object({
  personaId: z.number().int().positive(),
  needText: z.string().min(1, "Le texte du besoin est requis."),
  justificationType: z.enum(["PAIN", "TECH_LEGAL", "COMFORT"]),
  painId: z.number().int().positive().nullable().optional(),
  priority: z.enum(["MUST", "SHOULD", "COULD", "WONT"]).nullable().optional(),
});
```

Guarantee demonstrated:

- make the write contract explicit before the service layer

## 7. AI output contract and anti-prescriptive guardrail

Source patterns:
- `web/lib/ai/schemas/valuePrototypeSignal.schema.ts`
- `web/lib/ai/__tests__/valuePrototypeSignal.schema.test.ts`

```ts
export const ReadabilitySignalSchema = z.object({
  readability_understood: z.string().max(300),
  readability_not_understood: z.string().max(300),
  readability_user_question: z.string().max(200),
});

export const FocusAnalysisArraySchema = z.array(
  z.object({
    screen_order: z.number().int().positive(),
    focus_verdict: z.enum(["FOCUSED", "MULTIPLE_INTENTS", "UNCLEAR"]),
    explanation: z.string().max(300),
  })
);

export const FORBIDDEN_WORDS = [
  "améliorer",
  "corriger",
  "optimiser",
  "devrait",
  "il faudrait",
  "je conseille",
  "vous devriez",
  "recommande",
] as const;
```

Guarantee demonstrated:

- constrain AI output to a product contract
- keep the "AI mirror" descriptive rather than prescriptive

## 8. Deterministic hidden state for AI personas

Source pattern: `web/lib/ai/traits.engine.ts`

```ts
function hashStringToUint32(input: string): number {
  let h = 2166136261;
  for (let i = 0; i < input.length; i++) {
    h ^= input.charCodeAt(i);
    h = Math.imul(h, 16777619);
  }
  return h >>> 0;
}

function mulberry32(seed: number) {
  return function () {
    let t = (seed += 0x6d2b79f5);
    t = Math.imul(t ^ (t >>> 15), t | 1);
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

export function generateHiddenTraits(args: {
  seed: string;
  constraints: PersonaConstraints;
  session: TraitsSessionConfig;
}): { seed: string; traits: HiddenTraitRecord[] } {
  const rng = mulberry32(hashStringToUint32(args.seed));

  // Constraints filter the eligible trait pool; the compatibility logic
  // itself is intentionally not part of this public excerpt.
  const core = pickOne(rng, compatible(CORE_TRAITS, args.constraints));
  const traits: HiddenTrait[] = [core];

  if (
    args.session.enableEmotionalTraits &&
    rng() < (args.session.emotionalProbability ?? DEFAULT_EMOTIONAL_PROBABILITY)
  ) {
    const emoList = compatible(EMOTION_TRAITS, args.constraints);
    if (emoList.length > 0) traits.push(pickOne(rng, emoList));
  }

  return {
    seed: args.seed,
    // Trait records are projected to a stable shape used by the AI layer.
    // Field names and the full trait catalog are intentionally omitted here.
    traits: traits.map(toPublicTraitRecord),
  };
}
```

Note:

- the full trait catalog, selection weights, and prompt injection details are intentionally omitted from the public showcase.
- see [AI_DESIGN.md](./AI_DESIGN.md) for the design rationale behind hidden state.

Guarantee demonstrated:

- persona behavior is stable for the same seed and session context
- hidden behavioral state is generated server-side and stays invisible to students
- controlled variability does not rely on runtime randomness
- replay and audit remain meaningful because the hidden state is reproducible

## 9. Fail-closed AI boundary and provider routing

Source patterns:
- `web/lib/ai/valuePrototypeAnalyzer.ts`
- `web/lib/ai/providerRouter.ts`
- `web/lib/ai/__tests__/openaiJson.test.ts`

```ts
const parsed = ReadabilitySignalSchema.safeParse(rawParsed);
if (!parsed.success) {
  throw new Error("AI_RESPONSE_INVALID_JSON");
}

if (!containsNoForbiddenWords(allText)) {
  const forbidden = findForbiddenWords(allText);
  throw new Error(`AI_RESPONSE_FORBIDDEN_WORDS: ${forbidden.join(", ")}`);
}
```

```ts
function readProviderConfig(): ProviderConfig {
  const primaryRaw = process.env.AI_PRIMARY_PROVIDER ?? "openai";
  const fallbackRaw = process.env.AI_FALLBACK_PROVIDER ?? "gemini";
  const forceRaw = process.env.AI_FORCE_PROVIDER?.trim() ?? "";

  return {
    primaryProvider: isAiProviderName(primaryRaw) ? primaryRaw : "openai",
    fallbackEnabled: process.env.AI_FALLBACK_ENABLED === "true",
    fallbackProvider: isAiProviderName(fallbackRaw) ? fallbackRaw : "gemini",
    forceProvider: isAiProviderName(forceRaw) ? forceRaw : null,
  };
}
```

```ts
it("throws AI_PROVIDER_EXHAUSTED when OpenAI and Gemini both fail", async () => {
  process.env.AI_FALLBACK_ENABLED = "true";
  process.env.AI_FALLBACK_PROVIDER = "gemini";
  await expect(
    callOpenAIJson([{ role: "user", content: "test" }])
  ).rejects.toThrow("AI_PROVIDER_EXHAUSTED");
});
```

Guarantee demonstrated:

- malformed AI output is rejected before it reaches product state
- prescriptive AI wording is treated as a contract violation
- provider fallback is tested as an operational failure mode

## 10. Deterministic simulation and an explicit test

Source patterns:
- `web/lib/simulation/mciEngine.ts`
- `web/lib/simulation/__tests__/mciEngine.golden.test.ts`

```ts
/**
 * Pure deterministic engine.
 *
 * Determinism guarantees:
 * - snapshots are sorted by team_id ASC before any computation,
 * - no time/random APIs,
 * - stable largest-remainder allocation tie-broken by team_id ASC.
 */
export function simulateRound(
  sessionParams: SessionParams,
  snapshots: readonly TeamRoundSnapshot[],
  prevResultsByTeam: Readonly<Record<number, PrevRoundState>>,
  adjustmentsByTeam: Readonly<Record<number, RoundAdjustments>> = {}
): readonly TeamRoundResult[] {
  const ordered = [...snapshots].sort((a, b) => a.team_id - b.team_id);
  // deterministic computation only
}
```

```ts
it("is deterministic for identical inputs and stable to snapshot ordering", () => {
  const session = makeSession(1, { market_size: 17 });
  const s1 = makeSnapshot(1, { m_effectif_cents: 20_000 });
  const s2 = makeSnapshot(2, { m_effectif_cents: 10_000 });

  const prev = {
    1: makePrev({ active_paying_end: 3 }),
    2: makePrev({ active_paying_end: 7 }),
  };

  const runA = simulateRound(session, [s1, s2], prev);
  const runB = simulateRound(session, [s2, s1], prev);
  const runC = simulateRound(session, [s1, s2], prev);

  expect(runA).toEqual(runB);
  expect(runA).toEqual(runC);
});
```

Guarantee demonstrated:

- keep the market engine reproducible and auditable

## 11. Snapshot table for historization

Source pattern: `schema.sql`

```sql
create table if not exists public.finance_round_snapshot (
  id uuid primary key default gen_random_uuid(),
  session_id uuid not null references public.game_sessions(id) on delete cascade,
  team_id bigint not null references public.teams(id) on delete cascade,
  round integer not null check (round > 0),
  finance_state_id uuid not null references public.finance_state(id) on delete restrict,

  insight_score integer not null,
  market_fit_score integer not null,
  offer_snapshot jsonb not null,
  marketing_snapshot jsonb not null,
  projection_snapshot jsonb not null,
  round_costs jsonb not null default '{}'::jsonb,
  solution_penalty jsonb not null default '{}'::jsonb,

  closed_at timestamptz not null default timezone('utc', now()),
  created_at timestamptz not null default timezone('utc', now())
);
```

Note:

- In the production security model, `finance_round_snapshot` is append-only: select and insert are allowed, while update and delete are intentionally not exposed.

Guarantee demonstrated:

- preserve round inputs as explicit historical artifacts
- keep replay and post-round analysis possible
