# DT Fellow Assessment Tool

A browser-based AI tool that analyzes supervisor transcripts and produces structured performance assessments for DT Fellows — scoring them on a 1-10 rubric, mapping their work to business KPIs, identifying assessment gaps, and generating follow-up questions.

---

## How to run

No installation or build step required.

1. Clone or download this repository
2. Open `index.html` in any modern browser
3. Paste a supervisor transcript, add the Fellow's name and organization, and click **Analyze**

The tool calls the Anthropic Claude API directly from the browser. It works out of the box in the Claude.ai artifact environment where the API key is handled automatically.

---

## File structure

```
dt-fellow-assessment-tool/
├── index.html               # Complete standalone tool — UI + API call + rendering
├── sample-transcripts.json  # 3 annotated sample transcripts with expected scores
├── rubric.json              # Full rubric, KPI definitions, dimensions, and bias documentation
└── README.md                # This file
```

---

## What the tool assesses

Every transcript is evaluated across 5 output sections:

### 1. Score (1–10)
Maps to the DT rubric across three bands:

| Band | Range | What it means |
|---|---|---|
| Need Attention | 1–3 | Fellow requires active intervention |
| Productivity | 4–6 | Task execution is present; systems building is not |
| Performance | 7–10 | Fellow is identifying or solving problems beyond their assigned scope |

### 2. Survivability test
The single most important diagnostic: *If the Fellow left tomorrow, would any system they built continue running?* The tool flags this explicitly as passed or failed.

### 3. Evidence
Quotes extracted from the transcript, tagged by dimension (execution, systems building, KPI impact, change management) and signal (positive/negative), with an interpretation of what each quote means for the score.

### 4. KPI mapping
Maps plain supervisor language to the 8 DT KPIs: Lead Generation, Lead Conversion, Upselling, Cross-selling, NPS, PAT, TAT, and Quality. Each mapped KPI is tagged as system-level or personally delivered.

### 5. Gaps and follow-up questions
Flags which of the 4 assessment dimensions are missing from the transcript and generates targeted questions the interviewer should ask next.

---

## Prompt engineering approach

### The core problem
Supervisors are honest but systematically biased. Their language reflects perception, not always reality. A glowing transcript can hide a 5. A critical transcript can hide a 7. The prompt is engineered to correct for this gap.

### Bias corrections built into the prompt

**Helpfulness bias** — "She handles all my calls now" sounds like an 8. The prompt explicitly instructs the model to recognize task absorption as 5–6, not high performance.

**Presence bias** — "Always on the floor" is treated as neutral evidence. Physical presence does not raise the score without accompanying systems evidence.

**Halo/horn effect** — The prompt instructs the model to isolate individual stories and assess the full tenure independently, not let one event color the entire assessment.

**Recency bias** — The prompt flags language like "just last week" as a recency signal and instructs the model to probe for full-tenure evidence.

**Dependency trap** — A Fellow who is absorbing the supervisor's workload (running their meetings, handling their calls) without building self-sustaining systems is capped at 5–6 and flagged with a dependency warning.

### The 6 vs 7 boundary
This is the most important scoring decision and the most common failure point for naive prompts. The prompt makes the distinction explicit:

- Score 6: takes initiative *within* assigned scope — reliable, trusted, but reactive
- Score 7: *expands* scope independently — identifies problems the supervisor had not noticed or asked about

The prompt tests for this with the language signal: did the supervisor say "I asked him to..." (score 6) or "she came to me with something I hadn't noticed" (score 7)?

### Hallucination guardrails

**Grounding requirement** — Every score must be justified with a direct quote or near-quote from the transcript. The model is instructed not to infer evidence that is not present.

**Gap flagging over assumption** — When a dimension (especially change management) has no evidence in the transcript, the model must flag it as a gap and generate a follow-up question. It is not permitted to assume the dimension is fine.

**JSON-only output** — The system prompt instructs the model to return only valid JSON with no prose, no markdown, and no backticks. This forces structured output and eliminates narrative drift.

**Temperature implicit** — The model is called with default settings but the prompt is written to be deterministic: rubric values are enumerated, scoring rules are explicit, and the output schema is fixed. This reduces variance in scoring.

**Negative prompting** — The prompt explicitly lists what the model must not do:
- Do not let presence language raise the score
- Do not treat task absorption as systems building
- Do not let one positive story inflate the full score
- Do not assume change management is present if the transcript does not mention it

### Where I disagreed with the AI default behavior

During testing, the model's default tendency was to score generously — it would pick up on positive supervisor sentiment and mirror it in the score. For Anil (Prabhat Foods), an unprompted model gave scores of 8–9 based on the supervisor's warmth. The explicit dependency trap instruction and the survivability test were added specifically to counter this.

For Meena (Lakshmi Textiles), the model's first pass gave a 4 because it mirrored the supervisor's critical tone. Adding explicit presence bias correction and instruction to separate supervisor sentiment from systems evidence brought this to the correct score of 7.

---

## Sample transcript results

The tool is validated against 3 annotated transcripts. Expected scores:

| Fellow | Organization | Expected score | Common wrong score | Why wrong score fails |
|---|---|---|---|---|
| Karthik | Veerabhadra Auto | 6 | 8 | Presence bias — "always on floor" + warm tone. No self-sustaining system. |
| Meena | Lakshmi Textiles | 7 | 4 | Presence bias (inverted) — supervisor critical of laptop use. Misses 3 Layer 2 systems. |
| Anil | Prabhat Foods | 5–6 | 9 | Halo effect + helpfulness bias. Explicit survivability failure in transcript itself. |

---

## Tech stack

- Vanilla HTML, CSS, JavaScript — no framework, no build step
- Anthropic Claude API (`claude-sonnet-4-20250514`) via `fetch`
- No backend, no database, no authentication layer

---

## Assignment context

Built as part of the DT CultureTech software developer role simulation assignment. The tool addresses Part A and Part B of the assignment brief.

The core mandate: build a tool that assesses DT Fellows on the two-layer model (execution vs systems building), the 1–10 rubric, the 8 KPIs, the 4 assessment dimensions, and the 5 supervisor biases — and does so without being fooled by the three transcript traps described in the brief.