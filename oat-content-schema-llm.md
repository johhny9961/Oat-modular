# OAT Engine — Content Schema (LLM Reference)

Generate a JSON array of study problems. Each problem is an object with these exact fields:

## Required fields
- `id` (string): unique, pattern `"topic-phase-n"` e.g. `"trig-unit-circle-w1"`
- `subject` (string): short label e.g. `"trig"`, `"ochem"`, `"anatomy"`
- `topic` (string): grouping label e.g. `"Unit Circle"`
- `subtopic` (string): specific label e.g. `"Special Angles"`
- `conceptId` (string): slug shared across all phases of one concept e.g. `"unit-circle"`. Phase gating depends on this — Phase 2 won't unlock until Phase 1 with the same conceptId is studied.
- `phase` (integer): 1, 2, 3, or 4
- `difficulty` (integer): 1–5
- `problem_text` (string): the question
- `steps` (array): see phase rules below
- `answer` (string): correct answer
- `explanation` (string): brief — under 30 words

## Phase rules

**Phase 1** (worked example): all steps `{"text":"...","hidden":false}`. Optional: add `self_explanation` and `self_explanation_answer` strings to the problem. Difficulty 1. Steps under 25 words each.

**Phase 2** (faded): early steps `hidden:false`, later steps `{"text":"...","hidden":true,"reveal_text":"..."}`. Difficulty 2.

**Phase 3** (stepwise retrieval): ALL steps `hidden:true` with `reveal_text`. Each step `text` must be a question. Difficulty 3–4.

**Phase 4** (full problem): `"steps":[]`. Difficulty 3–5.

## Example — one concept, all four phases

```json
[
  {"id":"chain-rule-w1","subject":"calc","topic":"Derivatives","subtopic":"Chain Rule","conceptId":"chain-rule","phase":1,"difficulty":1,
   "problem_text":"Find f'(x) for f(x)=(3x+1)⁵",
   "steps":[
     {"text":"Outer: u⁵ where u=3x+1","hidden":false},
     {"text":"d/du(u⁵)=5u⁴","hidden":false},
     {"text":"Inner derivative: d/dx(3x+1)=3","hidden":false},
     {"text":"Chain rule: 5(3x+1)⁴·3=15(3x+1)⁴","hidden":false}
   ],
   "self_explanation":"Why multiply by the inner derivative?",
   "self_explanation_answer":"The chain rule scales the outer rate of change by how fast the inner function changes.",
   "answer":"15(3x+1)⁴","explanation":"Chain rule: outer derivative × inner derivative."},

  {"id":"chain-rule-f1","subject":"calc","topic":"Derivatives","subtopic":"Chain Rule","conceptId":"chain-rule","phase":2,"difficulty":2,
   "problem_text":"Find g'(x) for g(x)=sin(x²)",
   "steps":[
     {"text":"Outer: sin(u), u=x²","hidden":false},
     {"text":"d/du(sin(u))=cos(u)","hidden":false},
     {"text":"Apply chain rule and simplify.","hidden":true,"reveal_text":"Inner: 2x. g'(x)=cos(x²)·2x=2x·cos(x²)"}
   ],
   "answer":"2x·cos(x²)","explanation":"cos(x²) times inner derivative 2x."},

  {"id":"chain-rule-s1","subject":"calc","topic":"Derivatives","subtopic":"Chain Rule","conceptId":"chain-rule","phase":3,"difficulty":3,
   "problem_text":"Find h'(x) for h(x)=e^(4x)",
   "steps":[
     {"text":"What are the outer and inner functions?","hidden":true,"reveal_text":"Outer: eᵘ. Inner: u=4x."},
     {"text":"What is the derivative of eᵘ?","hidden":true,"reveal_text":"eᵘ — exponential is its own derivative."},
     {"text":"Apply chain rule — what is h'(x)?","hidden":true,"reveal_text":"e^(4x)·4=4e^(4x)"}
   ],
   "answer":"4e^(4x)","explanation":"e^(4x) times inner derivative 4."},

  {"id":"chain-rule-p1","subject":"calc","topic":"Derivatives","subtopic":"Chain Rule","conceptId":"chain-rule","phase":4,"difficulty":4,
   "problem_text":"Find f'(x) for f(x)=ln(cos(x)). Simplify.",
   "steps":[],
   "answer":"-tan(x)","explanation":"(1/cos(x))·(-sin(x)) = -tan(x)."}
]
```

## Validation rules that will reject output
- Duplicate `id` values
- Missing any required field
- `phase` not in [1,2,3,4]
- `difficulty` not in [1,2,3,4,5]
- `hidden:true` step without `reveal_text`
- `steps` not an array (use `[]` for Phase 4)
- Non-string values for string fields

Output ONLY the JSON array. No markdown fencing, no commentary.
