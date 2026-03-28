# OAT Engine — Content Generation Guide

**Give this document to any LLM alongside the OAT Engine artifact. It tells the LLM exactly how to produce content packs you can paste into the Content Manager.**

---

## How to use this guide

Upload or paste this document into a conversation with an LLM (Claude, GPT, etc.) alongside the OAT Engine artifact. Then ask something like:

- "Make me a content pack on trig identities for the OAT Engine"
- "Generate 20 problems covering alkene reactions, all four phases"
- "I need a quick review set on probability — just Phases 1 and 3"

The LLM should produce a JSON array you can copy, paste into the Content Manager's "Add Content" tab, validate, and load.

---

## Schema reference

Every problem is a JSON object with these **required** fields:

```json
{
  "id": "trig-unit-circle-worked-1",
  "subject": "trig",
  "topic": "Unit Circle",
  "subtopic": "Special Angles",
  "conceptId": "unit-circle",
  "phase": 1,
  "difficulty": 1,
  "problem_text": "What is sin(30°)?",
  "steps": [],
  "answer": "sin(30°) = 1/2",
  "explanation": "On the unit circle, 30° corresponds to the point (√3/2, 1/2). The y-coordinate is the sine value."
}
```

### Field rules

| Field | Type | Rules |
|-------|------|-------|
| `id` | string | Unique across ALL loaded content. Use pattern: `topic-phase-number` (e.g., `"trig-unit-circle-worked-1"`) |
| `subject` | string | Any short label. This becomes a filter button on the home screen. Use the same string for all problems in one subject area (e.g., `"trig"`, `"ochem"`, `"anatomy"`, `"contracts"`) |
| `topic` | string | Groups problems on the Topics screen. One level below subject (e.g., `"Unit Circle"`, `"SN1/SN2/E1/E2"`) |
| `subtopic` | string | Shown as a badge during study. More specific than topic (e.g., `"Special Angles"`, `"Quadrant Signs"`) |
| `conceptId` | string | **Critical for phase gating.** All problems about the same concept across all four phases MUST share the same `conceptId`. The app won't show Phase 2 problems until Phase 1 with the same `conceptId` is studied. Use short slugs: `"unit-circle"`, `"sn2"`, `"chain-rule"` |
| `phase` | integer | 1, 2, 3, or 4. See phase system below |
| `difficulty` | integer | 1 through 5. Affects scheduling order and ADHD difficulty calibration |
| `steps` | array | Contents depend on phase. See phase system below |
| `answer` | string | The correct answer, shown after grading |
| `explanation` | string | Brief explanation of WHY the answer is correct |

### Optional fields (Phase 1 only)

| Field | Type | Purpose |
|-------|------|---------|
| `self_explanation` | string | A deeper "why" question shown after the worked example. Prompts the learner to think before revealing the answer |
| `self_explanation_answer` | string | The answer to the self-explanation question. Required if `self_explanation` is present |

---

## The four-phase system

This is the core pedagogical model. Each concept should have problems at multiple phases, all sharing the same `conceptId`. The app enforces phase gating — the learner must complete earlier phases before unlocking later ones.

### Phase 1: Worked Example
**The learner reads, not solves.** Every step is visible. The goal is schema acquisition — building a mental model of how this type of problem works.

- All steps have `"hidden": false`
- Difficulty should be 1
- Include `self_explanation` and `self_explanation_answer` to prompt deeper processing
- The "I've studied this" button advances the learner without grading accuracy

```json
{
  "id": "chain-rule-worked-1",
  "subject": "calc",
  "topic": "Derivatives",
  "subtopic": "Chain Rule",
  "conceptId": "chain-rule",
  "phase": 1,
  "difficulty": 1,
  "problem_text": "Find the derivative of f(x) = (3x + 1)⁵",
  "steps": [
    { "text": "Identify the outer function: u⁵, where u = 3x + 1.", "hidden": false },
    { "text": "Differentiate the outer function: d/du(u⁵) = 5u⁴.", "hidden": false },
    { "text": "Differentiate the inner function: d/dx(3x + 1) = 3.", "hidden": false },
    { "text": "Multiply (chain rule): f'(x) = 5(3x + 1)⁴ · 3 = 15(3x + 1)⁴.", "hidden": false }
  ],
  "self_explanation": "Why do we multiply by the derivative of the inner function?",
  "self_explanation_answer": "The chain rule accounts for how fast the inner function changes. If the inner function changes faster (larger derivative), the whole composite function changes faster. Multiplying by du/dx scales the outer rate of change by the inner rate of change.",
  "answer": "f'(x) = 15(3x + 1)⁴",
  "explanation": "Chain rule: differentiate the outside, keep the inside, multiply by the derivative of the inside."
}
```

### Phase 2: Faded Completion
**Some steps are given, some are hidden.** The learner sees the setup, then must complete the solution. The hidden steps have a `reveal_text` that shows the answer when the learner taps "Reveal."

- Early steps: `"hidden": false` (given to the learner)
- Later steps: `"hidden": true` with a `"reveal_text"` field
- Difficulty should be 2
- Typically fade from the end backward — give the setup, hide the conclusion

```json
{
  "id": "chain-rule-faded-1",
  "subject": "calc",
  "topic": "Derivatives",
  "subtopic": "Chain Rule",
  "conceptId": "chain-rule",
  "phase": 2,
  "difficulty": 2,
  "problem_text": "Find the derivative of g(x) = sin(x²)",
  "steps": [
    { "text": "Identify the outer function: sin(u), where u = x².", "hidden": false },
    { "text": "Differentiate the outer function: d/du(sin(u)) = cos(u).", "hidden": false },
    { "text": "Now apply the chain rule: multiply by the derivative of the inner function and simplify.", "hidden": true, "reveal_text": "d/dx(x²) = 2x. So g'(x) = cos(x²) · 2x = 2x·cos(x²)." }
  ],
  "answer": "g'(x) = 2x·cos(x²)",
  "explanation": "Outer derivative cos(x²) times inner derivative 2x."
}
```

### Phase 3: Stepwise Retrieval
**All steps are hidden questions.** The learner sees the problem, thinks about each step, then reveals it to check. This is the breakthrough format — the learner predicts each step before seeing it.

- ALL steps have `"hidden": true` with `"reveal_text"`
- Each step's `"text"` is a QUESTION the learner should answer mentally before revealing
- Difficulty should be 3–4
- Keep to 3–5 steps

```json
{
  "id": "chain-rule-step-1",
  "subject": "calc",
  "topic": "Derivatives",
  "subtopic": "Chain Rule",
  "conceptId": "chain-rule",
  "phase": 3,
  "difficulty": 3,
  "problem_text": "Find the derivative of h(x) = e^(4x)",
  "steps": [
    { "text": "What is the outer function and what is the inner function?", "hidden": true, "reveal_text": "Outer: e^u. Inner: u = 4x." },
    { "text": "What is the derivative of the outer function?", "hidden": true, "reveal_text": "d/du(e^u) = e^u. The exponential function is its own derivative." },
    { "text": "Apply the chain rule — what is h'(x)?", "hidden": true, "reveal_text": "h'(x) = e^(4x) · 4 = 4e^(4x). Inner derivative is 4." }
  ],
  "answer": "h'(x) = 4e^(4x)",
  "explanation": "e^(4x) stays, multiplied by the derivative of 4x, which is 4."
}
```

### Phase 4: Full Problem
**No steps. No hints.** The learner types their answer, reveals the correct answer, and self-grades. This is where spacing and interleaving operate at full power.

- `"steps"` must be an empty array: `[]`
- Difficulty should be 3–5
- These are the problems that get interleaved across topics

```json
{
  "id": "chain-rule-full-1",
  "subject": "calc",
  "topic": "Derivatives",
  "subtopic": "Chain Rule",
  "conceptId": "chain-rule",
  "phase": 4,
  "difficulty": 4,
  "problem_text": "Find the derivative of f(x) = ln(cos(x)). Simplify.",
  "steps": [],
  "answer": "f'(x) = -tan(x)",
  "explanation": "Chain rule: (1/cos(x)) · (-sin(x)) = -sin(x)/cos(x) = -tan(x)."
}
```

---

## Building a concept across phases

A well-built concept has problems at all four phases sharing the same `conceptId`. The app gates progression — the learner can't skip to Phase 4 without working through Phase 1 first.

**Minimum viable concept:** 1 problem per phase (4 total).
**Recommended:** 1–2 Phase 1, 1–2 Phase 2, 2–3 Phase 3, 3–5 Phase 4. More Phase 4 problems give the interleaving engine material to work with.

### Difficulty scaling within Phase 4

Use difficulty 3 for straightforward applications, 4 for problems requiring discrimination (which technique? which rule?), and 5 for multi-step or combined-concept problems.

---

## Interleaving guidelines

The engine interleaves across topics automatically. To make this work well:

- Create multiple concepts within the same subject
- Phase 4 problems across different topics should have similar surface features but require different strategies — this is where the discrimination benefit is strongest
- Example: for calculus, Phase 4 problems mixing chain rule, product rule, and quotient rule force the learner to identify which rule applies, not just execute it

---

## Output format

Output MUST be a valid JSON array. Start with `[`, end with `]`. Every string in double quotes. No trailing commas. No comments.

```json
[
  { "id": "...", "subject": "...", ... },
  { "id": "...", "subject": "...", ... }
]
```

The user will paste this directly into the app's Content Manager and tap "Validate." If validation fails, the app shows specific error messages per problem.

---

## Common mistakes to avoid

1. **Duplicate IDs.** Every `id` must be unique across all loaded content, not just within this batch. Use descriptive prefixes: `"trig-pythagorean-worked-1"` not `"q1"`.

2. **Mismatched conceptIds across phases.** If Phase 1 uses `"conceptId": "unit-circle"` but Phase 3 uses `"conceptId": "unitcircle"`, the phase gate breaks — the app thinks they're unrelated concepts.

3. **Phase 4 with non-empty steps.** Phase 4 problems must have `"steps": []`. Any steps will be ignored but the validator may flag them.

4. **Phase 2/3 missing reveal_text.** Every step with `"hidden": true` MUST have a `"reveal_text"` field. The validator rejects hidden steps without it.

5. **Forgetting the subject field.** Every problem needs a `subject` string. This drives the home screen filter buttons.

6. **Making Phase 3 steps into statements instead of questions.** Phase 3 steps should be questions the learner answers mentally before revealing. "The derivative of sin(x) is cos(x)" is a Phase 1 step. "What is the derivative of sin(x)?" is a Phase 3 step.

---

## Quick-start prompt template

Copy and modify this prompt:

```
I'm using a spaced repetition study app with a four-phase learning system.
Generate a content pack on [TOPIC] for the subject [SUBJECT].

Cover these concepts:
- [concept 1]
- [concept 2]
- [concept 3]

For each concept, create:
- 1 Phase 1 worked example (all steps visible, include self_explanation)
- 1 Phase 2 faded completion (first steps given, last steps hidden)
- 1 Phase 3 stepwise retrieval (all steps are hidden questions)
- 3 Phase 4 full problems (no steps, varied difficulty 3-5)

Output as a single JSON array I can paste directly into the app.
Use [SUBJECT] as the subject field for all problems.
Make sure all problems for the same concept share the same conceptId.
```
