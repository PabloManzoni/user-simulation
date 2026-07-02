# Profile vocabulary — the controlled menu for generated synthetic users

> **Source (pinned):** `synthetic-user-builder` @ commit `e6196351ea40d5e5f201330c67eeddd6484491f9`
> (e619635, 2026-06-29) — files `src/ai/genericOptions.ts`, `src/ai/behaviorAxes.ts`,
> `src/lib/validation.ts`, `src/state/types.ts`, `src/lib/export.ts`.
> **Profile file format: version 1** (`toProfileFile`).
> If any of those builder files change, update this file AND re-run the round-trip test
> (plan, Bloque A step 4). Do not edit the pools here independently of the builder.

The synthetic-profile-generator MUST build profiles by **choosing from these pools**. Custom items (outside a
pool) are allowed only when the business genuinely demands it — justify each one in `selfCheck.customJustification`.
Custom items go in the slice's `custom` array AND in `selected`.

---

## Behavior axes (5 axes, positions 0–4)

Each axis position maps to EXACTLY one statement. The chosen statements (one per axis) form
`decisionBehavior.selected` (5 items), and the numeric positions form `behaviorAxes`.
**Both must be produced and must be consistent with each other.**

### pace — Skims ↔ Reads thoroughly
- 0: Skims very fast, grabbing only the most prominent signal.
- 1: Scans quickly for the first clear signal.
- 2: Balances quick scanning with reading the key details.
- 3: Reads carefully before acting.
- 4: Reads everything thoroughly before deciding.

### priority — Speed ↔ Accuracy
- 0: Prioritizes speed over accuracy.
- 1: Leans toward speed, accepting some risk.
- 2: Balances speed and accuracy.
- 3: Leans toward accuracy over speed.
- 4: Prioritizes accuracy, even if it's slow.

### verification — Rarely checks ↔ Double-checks
- 0: Rarely double-checks anything.
- 1: Seldom double-checks unless something looks off.
- 2: Double-checks when a decision feels risky.
- 3: Double-checks critical information.
- 4: Double-checks almost everything before acting.

### trust — Trusting ↔ Skeptical
- 0: Trusts what the interface shows without question.
- 1: Generally trusts on-screen labels and indicators.
- 2: Trusts the interface but notices clear inconsistencies.
- 3: Skeptical of automated states and recommendations.
- 4: Distrusts the interface until evidence is clear.

### escalation — Self-reliant ↔ Escalates
- 0: Avoids escalation; resolves things on their own.
- 1: Prefers to handle it alone, escalating only as a last resort.
- 2: Escalates when genuinely stuck.
- 3: Escalates when unsure rather than guess.
- 4: Escalates quickly whenever confidence drops.

**Polarization**: a contrasted pair = same role, opposite positions on 1–2 chosen axes (e.g. one at
pace 0 / trust 1, the other at pace 4 / trust 4). Keep the other axes similar so the contrast is
attributable.

---

## Generic pools

### Roles
End user · Power user · Admin · Manager · Reviewer · External user · New user

### Expertise scales
- **Domain expertise**: Low · Medium · High · Expert
- **Technical proficiency**: Low · Medium · High · Power user
- **Product type familiarity**: First time user · Occasional user · Regular user · Daily user · Power user
- **Exact product familiarity**: Unknown · None · Low · Medium · High

### Information needs
Status · Severity or priority · Ownership · Timestamp / recency · Next action · Evidence ·
Confirmation · Required fields · Missing information · Policy or rule · Threshold · Risk or impact ·
Progress / completion · Error reason

### Constraints
- Can only use information visible in the interface
- Cannot inspect future screens before reaching them
- Cannot use backend logic or internal product knowledge
- Cannot rely on undeclared business rules
- Decides only from what the current screen communicates

### Forbidden assumptions
- Cannot assume backend logic
- Cannot assume internal business rules
- Cannot assume disabled elements have a specific cause
- Cannot infer meaning from color alone
- Cannot assume information is up to date unless recency is shown
- Cannot assume an item is active or current unless shown
- Cannot assume something is complete unless confirmation is shown
- Cannot assume missing data means there is no issue
- Cannot assume a default value is correct
- Cannot assume the next action unless visible or clearly implied
- Cannot use knowledge from future screens
- Cannot inspect steps that have not been reached
- Cannot compensate for unclear labels
- Cannot mentally fix missing interface information

### Friction triggers
Missing status · Unclear next action · Conflicting information · Unclear labels · Too many steps ·
Too many required fields · Dense terminology · Unclear error message · Disabled action with no
explanation · No confirmation after action · Too much visual noise · No priority signal · No clear
owner or responsibility · No sense of how recent the data is · Ambiguous importance or urgency ·
Repeated information · Unexpected empty state · No visible progress · Inconsistent terminology

### Emotional behaviors
Starts focused · Starts skeptical · Starts confident · Starts rushed · Confidence increases with
clear feedback · Confidence decreases with missing context · Frustration increases with repetition ·
Trust decreases with inconsistency · Fatigue increases with long flows · Anxiety increases when risk
is unclear · Relief appears after confirmation · Doubt persists if the final state is unclear

### Abandonment rules
- Stops if required information is missing
- Escalates when they can't confirm the outcome
- Asks a teammate when terminology is unclear
- Continues but loses trust
- Completes the task but remains unsure
- Abandons if the next action is not visible
- Abandons after repeated errors
- Avoids risky action without confirmation
- Uses another tool if the product does not provide enough evidence
- Marks task unresolved if there is no final confirmation

### Suitable tasks
Review status · Make a decision from displayed information · Detect if action is needed · Interpret
system feedback · Identify missing information · Review operational exceptions · Complete a bounded
workflow · Evaluate confidence in a decision

### Unsuitable tasks
Admin configuration · Deep compliance audit · Expert technical setup · Backend troubleshooting ·
Long term strategic analysis · Creating system rules · Tasks requiring unavailable institutional
knowledge · Tasks outside the selected role

---

## Quality gates (from the builder's validation.ts — the architect MUST meet these)

**INVALID — blocks the profile, must be fixed before anything else:**
- **Task/navigation language** anywhere in the profile. Literal detector phrases: `click`, `tap`,
  `open the menu`, `go to screen`, `go to the`, `select the tab`, `complete this task`,
  `find shipment`, `press button`, `press the`, `navigate to`, `scroll to`, `the alert`.
  A profile describes BEHAVIOR, never what to click or where to go. Task-shaped input belongs in the
  task, never in the profile.
- **Backend knowledge**: phrases like `knows the backend`, `knows internal`, `knows the business
  rule`, `knows the api`, `knows the database`, `aware of the backend`.

**Hard thresholds (retry until met):**
- Forbidden assumptions: **≥ 4** selected
- Constraints: **≥ 3** selected
- Total signals (decisionBehavior + informationNeeds + constraints + forbiddenAssumptions +
  frictionTriggers): **≥ 12**
- Role description written as **behavior** (verbs like decide, scan, trust, assume, hesitate,
  escalate, review), not only demographics (age, city, marital status = "reads like a bio")
- At least 1 suitable task type; role selected; all expertise levels set

## Free-text classification rule

When curated free text is being placed into the framework, the possible buckets are: role ·
expertise · behavior axes · information needs · constraints · forbidden assumptions · friction
triggers · emotional behaviors · abandonment rules · **task** · **global context**. Anything that
sounds like navigation or an objective goes to **task** — never into the profile.

---

## Output artifact 1 — Markdown profile (`<slug>.md`)

Exact section order (matches the builder's export; content of each section derives from the
selections). Start with `# Synthetic User Profile`, then a fenced code block with a simple 7-line
ASCII face (cosmetic — the builder regenerates its own on export), then each section as
`Title:\n<content>\n`:

Profile name · Reusable role · Role summary · Domain expertise · Technical proficiency ·
Product type familiarity · Exact product familiarity · Primary motivation · Decision style ·
Attention pattern · Trust pattern · Behavior under pressure · Tolerance for ambiguity ·
Common wrong assumptions (bullets) · Required explicit information (bullets) · Constraints (bullets) ·
Behavioral rules (bullets) · Emotional progression rules (bullets) · Abandonment and escalation rules
(bullets) · Forbidden assumptions (bullets) · Suitable task types (bullets) · Unsuitable task types
(bullets) · Calibration notes · Profile quality check · Validation notes (bullets)

**Derivation rule (keeps the .md identical to what the builder exports from the .builder.json):**
- **Behavioral rules** bullets = the friction triggers, derived exactly like the builder: the lines
  of `frictionTriggers.generated` if you filled it, else `frictionTriggers.selected` verbatim. Do NOT
  put the axis statements here (they live in the Decision style prose and in the json's
  `decisionBehavior`).
- **Emotional progression rules** bullets = lines of `emotionalBehavior.generated` if filled, else
  `emotionalBehavior.selected` verbatim. Whatever you choose, the .md and the json MUST use the same
  source.
- **Profile quality check** = `Strong` (you must only ship profiles that pass all gates). Put the
  gate numbers and any pair/provenance metadata in **Validation notes**.

## Output artifact 2 — Builder file (`<slug>.builder.json`)

```json
{
  "app": "synthetic-user-builder",
  "version": 1,
  "profile": {
    "profileName": "<slug>",
    "primaryMotivation": "<paragraph>",
    "behaviorAxes": { "pace": 0, "priority": 0, "verification": 0, "trust": 0, "escalation": 0 },
    "generated": {
      "primaryMotivation": "<same as above>",
      "decisionStyle": "<paragraph derived from the 5 axis statements>",
      "attentionPattern": "<paragraph>",
      "trustPattern": "<paragraph>",
      "behaviorUnderPressure": "<paragraph>",
      "toleranceForAmbiguity": "<paragraph>",
      "commonWrongAssumptions": ["<bullet>", "..."],
      "calibrationNotes": "<paragraph>",
      "unsuitableTaskTypes": ["<from pool>", "..."]
    },
    "productContext": {
      "clientName": "<business name>",
      "productName": "<product name>",
      "businessDomain": "<domain>",
      "manualDescription": "<business summary from research>",
      "knownPrimaryUsers": "<comma-separated user types found>",
      "knownWorkflows": "<key workflows observed>",
      "knownRiskAreas": "<risk areas if any, else empty string>",
      "researchMode": "manual",
      "aiSummary": "<business summary>",
      "aiConfidence": "medium",
      "researched": true,
      "researchFailed": false,
      "aiSuggestions": null,
      "aiSource": null
    },
    "role": {
      "selected": ["<role name>"],
      "descriptions": { "<role name>": "<behavioral description>" },
      "custom": ["<role name if not from the generic pool, else empty>"]
    },
    "expertise": {
      "domainExpertise": "<scale>",
      "technicalProficiency": "<scale>",
      "productTypeFamiliarity": "<scale>",
      "exactProductFamiliarity": "<scale>",
      "interpretation": "<paragraph>"
    },
    "decisionBehavior": { "selected": ["<5 axis statements>"], "custom": [], "generated": "<the 5 statements joined>" },
    "informationNeeds": { "selected": ["..."], "custom": [], "generated": "" },
    "constraints": { "selected": ["..."], "custom": [], "generated": "" },
    "forbiddenAssumptions": { "selected": ["..."], "custom": [], "generated": "" },
    "frictionTriggers": { "selected": ["..."], "custom": [], "generated": "" },
    "emotionalBehavior": { "selected": ["..."], "custom": [], "generated": "" },
    "abandonmentRules": { "selected": ["..."], "custom": [], "generated": "" },
    "taskSuitability": {
      "suitable": ["<from pool>"],
      "unsuitable": ["<from pool>"],
      "customSuitable": [],
      "customUnsuitable": []
    }
  }
}
```

Omit `builtSignature`. Every field above must be present — the builder's editor expects the complete
structure. Custom items appear in BOTH `custom` and `selected` of their slice.
