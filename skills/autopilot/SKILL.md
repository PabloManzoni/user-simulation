---
name: autopilot
description: Autopilot mode — discovers synthetic users for a live web app from just a URL: researches the business, proposes users + tasks for human approval, generates builder-compatible profiles, then runs one simulation per user and consolidates findings. Use when the user wants an automatic multi-user UX evaluation from a URL, or types /user-simulation:autopilot.
---

# Autopilot — automatic multi-user simulation

You orchestrate the **autopilot mode**: from a **URL** (plus an optional short business description),
you research the business, **propose** synthetic users with their tasks, let the human **curate** the
proposal in chat, generate the approved profiles (builder-compatible), then run the standard
simulation for each user×task pair and consolidate the findings.

The human checkpoint is the heart of this mode: **you never run simulations on a proposal the user
has not approved.** "Run as is" approval in one step is always available — that IS the full-auto
experience, contained in this mode.

This mode **composes** the existing pieces without modifying them: each run reuses the
`user-simulation` skill's loop (synthetic-screen-evaluator per screen, synthetic-flow-synthesizer per run). Profile
generation uses the `synthetic-profile-generator` subagent and the vocabulary in this skill's directory
(`vocabulary.md` — the source of truth for pools, axes, quality gates and file templates).

---

## On launch — run Step 0 checks silently first

Run the Step 0 checks immediately. Do NOT print anything unless a check fails. If both pass and the
URL was provided, start Step 1 directly. If the URL is missing, ask for it (and mention the optional
short business description).

## Step 0 — Preflight (HARD STOP — before ANYTHING else)

### 0a. Playwright browser available?

Check via ToolSearch `select:mcp__playwright__browser_navigate,mcp__playwright__browser_snapshot`.
If NOT found:

> **⚠️ Setup needed — Playwright browser**
>
> The simulator drives the browser through the **Playwright MCP** server, which isn't available yet.
>
> 1. In your terminal, run: `claude mcp add playwright -- npx @playwright/mcp@latest`
> 2. **Restart Claude Code** — MCP tools only load when the session starts.
> 3. Tell me when it's back and I'll continue.

**STOP here.**

### 0b. URL provided?

Autopilot needs the live app's URL. No profile is required — this mode creates them. If the URL is
missing, ask for it and STOP until provided.

---

## Step 1 — Research the business

Navigate the site with Playwright: the home page plus 2–4 more pages chosen by this priority:

1. Pages showing the **primary user workflows** (dashboard, creation flows, booking, search…).
2. Pages with **user segmentation signals** (roles, plans/pricing, permissions, "who it's for").
3. **Onboarding or role selection** screens.

Produce: a business summary (what the product is, who it serves, key workflows, risk areas), the
apparent audience, and segment signals. Optionally complement with WebSearch.

**Confidence gate:** if the site gives weak signals (thin content, app behind a login, ambiguous
positioning) → ask the user for the short business description BEFORE proposing, and fold it in.

Keep it light: accessibility snapshots only, no screenshots; discard each snapshot once its signal
is extracted.

## Step 2 — Proposal

Present EXACTLY this structure (fill the placeholders; 2–5 users typically — propose what the
research honestly supports, not a fixed number):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍  AUTOPILOT PROPOSAL — <domain>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**What I understood:** <2-3 sentences: the business, its audience, where the evaluation value is>

**Proposed users:**

**1. <Role name>** — <one line: who they are and how they behave>
   · Expertise: <domain>/<technical> · <product familiarity>
   · Tasks: ① <task> ② <task>

**2. <Role name>** — ...

**3a/3b. <Role> (contrasted pair)** — <why this pair: what the contrast will reveal>
   · 3a <pole A one-line> · 3b <pole B one-line>
   · Shared task: ① <task>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Edit freely in natural language — for example:
· "sacá el usuario 2" · "al 1 agregale la tarea X" · "cambiale el rol al 3"
· or add context: "los usuarios suelen estar apurados" (I'll place it in the right spot)
Or say **"corré tal cual"** to start right away.
Estimated duration once approved: ~<N×3-5> min (<N> sequential runs).
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Rules for the proposal:
- **Polarized pairs only when the business shows audience heterogeneity** on some behavioral axis —
  and ALWAYS with the reason stated. Pair = same role, opposite positions on 1–2 axes.
- Tasks must be concrete, observable on the site, and phrased in natural language. Tasks live
  OUTSIDE profiles — they are never written into a profile.
- Each user must make sense WITH their tasks (a follower checks results; an organizer verifies
  brackets — never crossed).

## Step 3 — Curation loop

Apply the user's edits and re-show the updated proposal (same template). Rules:

- **Free-text classification**: when the user adds context, classify it into the framework buckets
  (role · expertise · behavior axes · information needs · constraints · forbidden assumptions ·
  friction triggers · emotional behaviors · abandonment rules · **task** · **global context**).
  The classification must be VISIBLE: it appears as new marked items in the updated proposal —
  never absorbed silently.
- **Task language goes to tasks**: anything that sounds like navigation or an objective becomes a
  task, never profile content.
- **Ambiguity → ask**: if the input fits more than one bucket, use AskUserQuestion with the
  candidate buckets as options (e.g. "«no le gusta esperar» — emotional pattern or abandonment
  rule?"). Never classify doubtful input silently.
- **Global vs per-user**: decide whether the context affects one user or all profiles; if not
  obvious, ask.
- Exit the loop ONLY on explicit approval ("dale", "corré tal cual", "aprobado", etc.).

## Step 4 — Generate profiles

For each approved user, spawn the `synthetic-profile-generator` subagent (Agent tool,
`subagent_type: "synthetic-profile-generator"`) passing as text: the business summary, the user's spec (role,
one-line, expertise hints, axis positions if polarized + pair partner slug, curated context), the
assigned tasks (context only), and the ABSOLUTE PATH to this skill's `vocabulary.md`. Architects for
different users can run in parallel (they don't touch the browser).

For each returned profile, the orchestrator:

1. **Verifies the quality gates itself** (never trust selfCheck alone): the INVALID detectors
   (task/navigation language, backend knowledge — lists in vocabulary.md) **block**: a profile with
   an INVALID finding never proceeds. Hard thresholds (forbidden ≥ 4, constraints ≥ 3, total signals
   ≥ 12, behavioral role description): up to **2 retries** passing the concrete issues back to the
   architect; if still failing → show the profile with a warning and ask (use as-is / discard).
2. **Writes both files**: `profiles/<slug>.md` and `profiles/<slug>.builder.json`, where
   `<slug> = <domain>-<role>[-<variant>]` (e.g. `padelpro-player-veteran`). Create `profiles/` if
   needed. Never overwrite an existing profile without asking.
3. Shows a one-line summary per profile: slug · validation verdict · files written.

## Step 5 — Role-task coherence check

Before running anything, check every task against its profile's Suitable/Unsuitable task types.
On a mismatch:

- (a) If another approved user has the task as suitable → offer to reassign it.
- (b) If nobody fits → offer to drop that user×task pair OR run it anyway with explicit
  confirmation.
- (c) Whatever the decision, record it (it goes in that run's report and in the consolidated one).

---

## Step 6 — Runs (sequential, one browser)

For each approved user×task pair, execute the `run` skill's Steps 1–4 exactly as written
there (Prepare → Open the app → perceive/decide/execute loop with `synthetic-screen-evaluator` →
synthesis with `synthetic-flow-synthesizer`). Same step cap (15), same stop conditions, same report format. Additional rules for
batch mode:

- **One run at a time** — a single Playwright browser; never interleave runs.
- **Context hygiene**: discard each Playwright snapshot right after executing that step's action.
  Keep per step ONLY the evaluator JSON (`{ action, clarityLevel, doubtDetected, reason, abandoned,
  estimatedTimeSeconds, emotionalState, memory }`).
- After each run: save the individual report to `results/<YYYY-MM-DD>-<slug>.md`, then retain only
  the report path + its Result/Steps line — drop the step list from working context. The
  the consolidation step reads reports FROM DISK.
- Between runs, print exactly one progress line:
  `Run 2/4 — <slug>: <Result> (N steps, ~Xs)`.
- If a run dies for technical reasons (site down, browser error), record it as
  `Run failed (technical)` with the error, skip to the next run, and report it in Step 8 — a
  technical failure is NOT a UX abandonment and must never be reported as one.

## Step 7 — Consolidation

Spawn the `synthetic-discovery-synthesizer` subagent (Agent tool, `subagent_type: "synthetic-discovery-synthesizer"`) passing as text:

- the **business summary** (from Step 1),
- the **run list** — for each run: `{ slug, role, isPolarized, pairPartner, axes, task, result,
  steps, timeSeconds, reportPath }`,
- the instruction to Read each `reportPath` from disk.

Save the returned report to `results/<YYYY-MM-DD>-autopilot-<domain>.md` (same date as everything
else in this run; `<domain>` = host without TLD).

## Step 8 — Deliver

Do **NOT** paste any full report in chat. Show exactly this structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍✅  AUTOPILOT COMPLETE — <domain>
📄  Consolidated report: results/<YYYY-MM-DD>-autopilot-<domain>.md
📄  Individual reports: results/<YYYY-MM-DD>-<slug>.md (×N)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| User | Result | Steps | ~Time |
|------|--------|-------|-------|
| <slug> | <Goal reached / Abandoned at step N> | N | Xs |

**What happened:**
- <One sentence: the strongest CONVERGENT finding — what hit every user>
- <One sentence: the sharpest DIVERGENCE — which role or pole suffered something the others didn't>
- <One sentence: Fix this first — pulled verbatim from the consolidated report>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POTENTIAL IMPROVEMENTS (consolidated)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<The consolidated Potential improvements table from the report>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Orchestrator rules

- **Approval first, no exceptions.** Research and proposal are free; simulations and profile files
  are not created until the human approves the proposal.
- **Never invent users** the research doesn't support — fewer honest users beat a padded list.
- **The proposal is the contract**: what was approved is exactly what gets generated and run.
- **Visible classification, always**: silent absorption of user context is a failure.
- **Tasks never enter profiles.** The architect receives them as context only; the orchestrator
  keeps the user×task mapping.
- **Same date for all artifacts** of one autopilot run.
- Reuse, don't reimplement: the per-run loop belongs to the `run` skill; the profile
  format belongs to `vocabulary.md`; the report format belongs to `synthetic-flow-synthesizer`.
