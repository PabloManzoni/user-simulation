---
name: simulation-auto
description: The inferred simulation mode (autopilot) — from just a URL it researches the business, asks only "one screen or a whole flow?" (and, for a flow, which of the inferred tasks), builds 3 realistic synthetic users, runs them all, consolidates the findings and returns a short action summary. Say "let me edit the users" for the full curation mode. Use when the user wants a multi-user UX evaluation inferred from a URL, or types /user-simulation:simulation-auto.
---

# Simulation-auto — automatic multi-user simulation

You orchestrate the **inferred simulation mode** (autopilot): from a **URL** you research the
business, infer **3 synthetic users** who would really use it, generate their builder-compatible
profiles, run the standard simulation for each, consolidate the findings, and deliver a short action
summary.

**It's automatic.** The only questions are **"one screen or a whole flow?"** and, for a flow,
**"which task?"** (inferred options + a custom one). Everything else is decided by you. Answering
those questions IS the approval to run.

This mode **composes** the existing pieces without modifying them: each run reuses the
`simulation-run` skill's loop (synthetic-screen-evaluator per screen, synthetic-flow-synthesizer per
run). Profile generation uses the `synthetic-profile-generator` subagent and the vocabulary in this
skill's directory (`vocabulary.md` — the source of truth for pools, axes, quality gates and file
templates).

**Browser:** follow `../user-simulation/browser.md` (relative to this skill's base directory) — it
picks whatever browser is already available, nothing to install, and defines `OPEN`, `READ`, `ACT`,
`WAIT`, `SHOT`.

---

## Modes

- **Handoff** (args start with `handoff`, from `/user-simulation`): `url`, `scope`, `task`, `date`,
  `lang`, `deliver` are already answered — **ask nothing** — Step 2's questions are already answered.
- **Direct** (`/user-simulation:simulation-auto [url]`): ask for the URL if missing, then the Step 2
  questions. Nothing else.
- **Edit users** (hardcore — only when the user asks: *"let me edit the users"*, *"curate"*,
  `edit` in the args): after Step 1, use the proposal + curation loop in **Appendix A** instead of
  Step 3's automatic pick.

## Step 1 — Research the business (light)

Pick the driver per `browser.md` §1 and `OPEN(url)`. If the app can't be seen, show `browser.md` §4
and STOP.

Read the landing page plus **1–2** more pages, by this priority: primary user workflows
(dashboard, creation, booking, search…) → user segmentation signals (roles, plans/pricing, "who it's
for") → onboarding / role selection. Produce a **business summary** (what the product is, who it
serves, key workflows, risk areas), the apparent audience, and segment signals.

If the signals are weak (thin content, app behind a login), infer from what you have — the domain,
the copy, the page titles — and say it in the business summary. Don't stop to ask.

Keep it light: `READ` only, no screenshots; drop each page once its signal is extracted.

## Step 2 — Scope and task (the only questions)

Skip whatever handoff already answered. Otherwise ONE AskUserQuestion call:

- header `Scope` — *"One screen or a whole flow?"*: `A flow` (one task end to end) · `One screen`
  (only the screen at the URL).

If **flow**: infer **3 tasks** a real user of THIS site would come to do — concrete, observable on
the site, in the user's words — and ask a second AskUserQuestion, header `Task`, with the 3 tasks
as options (most central first, Recommended). "Other" = the user's custom task.

If **screen**: no task question — each user's goal for that screen is derived from their profile
(see Step 6).

## Step 3 — Pick the 3 users (automatic)

Choose **3 users** who would realistically do this task / visit this screen, chosen to **maximize
what the run reveals**:

- Different roles or expertise levels the business supports (e.g. first-timer · regular · power
  user), all plausible for THIS task — never a user who wouldn't do it.
- When the research shows audience heterogeneity on a behavioral axis, make **two of the three a
  contrasted pair** (same role, opposite positions on 1–2 axes — see vocabulary.md) and note why.
- Never invent a segment the research doesn't support; if only two roles are real, the third is a
  pole of a pair.

Print them — no question, the run continues immediately:

```
👥 Users: U1 <role> — <one line> · U2 <role> — <one line> · U3 <role> — <one line>
```

## Step 4 — Generate profiles

For each user, spawn the `synthetic-profile-generator` subagent (Agent tool,
`subagent_type: "synthetic-profile-generator"`) passing as text: the business summary, the user's
spec (role, one-line, expertise hints, axis positions if polarized + pair partner slug, any curated
context), the task (context only), and the ABSOLUTE PATH to this skill's `vocabulary.md`. Run the
three in parallel (they don't touch the browser).

For each returned profile, the orchestrator:

1. **Verifies the quality gates itself** (never trust selfCheck alone): the INVALID detectors
   (task/navigation language, backend knowledge — lists in vocabulary.md) **block**. Hard thresholds
   (forbidden ≥ 4, constraints ≥ 3, total signals ≥ 12, behavioral role description): up to
   **2 retries** passing the concrete issues back. Still failing → if only thresholds fail, use it
   and note it in that run's report; if an INVALID finding persists, drop that user, continue with
   the others, and note it. (In Edit-users mode: ask instead.)
2. **Writes both files**: `user-simulation-tests/simulation/profiles/<slug>.md` and
   `user-simulation-tests/simulation/profiles/<slug>.builder.json`, where
   `<slug> = <domain>-<role>[-<variant>]` (e.g. `padelpro-player-veteran`). Create the folder if
   needed. Never overwrite an existing profile: add `-2`, `-3`… to the slug.
3. **Role-task check**: the users were picked for the task, so a mismatch with the profile's
   Unsuitable task types should not happen; if it does, run anyway and note it in the report.

## Step 5 — Runs (sequential, one browser)

For each user, execute the `simulation-run` skill's Steps 1–4 exactly as written there (Prepare →
Open the app → perceive/decide/execute loop with `synthetic-screen-evaluator` → synthesis with
`synthetic-flow-synthesizer`). Same step caps (15 for a flow, 3 for one screen), same stop
conditions, same report format. In **screen** scope, the task each user receives is *"Look at this
screen and do what you came here to do"*, framed by that user's own goal. Additional batch rules:

- **One run at a time** — a single browser; never interleave runs.
- **Context hygiene**: drop each `READ` right after executing that step's action. Keep per step ONLY
  the evaluator JSON.
- After each run: save the report to `user-simulation-tests/simulation/results/<YYYY-MM-DD>-<slug>.md`,
  then keep only the report path + its Result/Steps line.
- Between runs, print exactly one progress line: `Run 2/3 — <slug>: <Result> (N steps, ~Xs)`.
- If a run dies for technical reasons (site down, browser error), record it as
  `Run failed (technical)`, skip to the next run, and report it — a technical failure is NOT a UX
  abandonment. Same for `Read-only limit`.

## Step 6 — Consolidation

Spawn the `synthetic-autopilot-synthesizer` subagent (Agent tool,
`subagent_type: "synthetic-autopilot-synthesizer"`) passing as text: the business summary, the run
list — for each run `{ slug, role, isPolarized, pairPartner, axes, task, result, steps, timeSeconds,
reportPath }` — and the instruction to Read each `reportPath` from disk.

Save the returned report to `user-simulation-tests/simulation/results/<YYYY-MM-DD>-autopilot-<domain>.md`
(same date as everything else in this run; `<domain>` = host without TLD).

## Step 7 — Deliver

- **`deliver=none`** (handoff inside a combined run): print one line —
  `✔ Simulation done — 3 users · consolidated report saved` — and return to the caller the users
  list `{ label U1…U3, role, result, steps, reportPath, profilePath }` and the consolidated path.
  Stop here.
- Otherwise spawn the `summary-synthesizer` subagent (`subagent_type: "summary-synthesizer"`)
  passing as text: `LANGUAGE` (handoff `lang`, else the conversation's language), `META` (url,
  domain, date, test=simulation, users=inferred, scope + task, driver), `USERS` (U1…U3 in the order
  printed in Step 3), `REPORTS` (`consolidatedPath`), `LINK_BASE: user-simulation-tests/` — paths
  relative to it. Save the reply to
  `user-simulation-tests/<YYYY-MM-DD>-<domain>-simulation-summary.md` and show exactly:

```
✅ Done — <domain>

<the summary, verbatim>

📄 Summary saved: user-simulation-tests/<filename>
```

Do **NOT** paste any full report in chat — the summary links to all of them.

---

## Orchestrator rules

- **Two questions max** (scope, then task for a flow). Everything else is decided automatically —
  except in Edit-users mode, where the curation loop is the point.
- **Never invent users** the research doesn't support.
- **Tasks never enter profiles.** The architect receives them as context only; the orchestrator
  keeps the user×task mapping.
- **Same date for all artifacts** of one run.
- Reuse, don't reimplement: the per-run loop belongs to the `simulation-run` skill; the profile
  format belongs to `vocabulary.md`; the report formats belong to the synthesizer agents.

---

## Appendix A — Edit-users mode (proposal + curation)

Only when the user asked to edit the users. After Step 1 (and Step 2's scope/task), present:

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
Edit freely in natural language — "remove user 2" · "add task X to user 1" ·
"these users are usually in a hurry" — or say **"run as is"**.
Estimated duration once approved: ~<N×3-5> min (<N> sequential runs).
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Curation rules:

- 2–5 users, as many as the research honestly supports; each user may have its own tasks.
- **Free-text classification**: classify added context into the framework buckets (role · expertise
  · behavior axes · information needs · constraints · forbidden assumptions · friction triggers ·
  emotional behaviors · abandonment rules · **task** · **global context**), shown as new marked items
  in the updated proposal — never absorbed silently.
- **Task language goes to tasks**, never into profile content.
- **Ambiguity → ask** with AskUserQuestion (candidate buckets as options).
- **Global vs per-user**: decide whether context affects one user or all; if not obvious, ask.
- Exit ONLY on explicit approval ("go ahead", "run as is", "approved"…). Then continue at Step 4,
  with one run per approved user×task pair; on a role-task mismatch offer to reassign the task, drop
  the pair, or run anyway — and record the decision in the reports.
