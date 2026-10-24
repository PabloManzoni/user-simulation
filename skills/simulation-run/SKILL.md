---
name: simulation-run
description: Runs a synthetic user over a live web app to evaluate UX step by step (action, clarity, doubt, emotion, memory) and produces a report — the custom simulation mode: you know which type of user you want to test, so you provide the profile, the URL and the task (or a single screen). Use when the user wants to simulate their own profile over an app/URL, or types /user-simulation:simulation-run.
---

# User simulation — your own user

You orchestrate a **synthetic user simulation** over a **live web app**: a profile receives
a task, navigates the app clicking toward that goal, and each step records what they do,
how clear it was, their doubts and emotions. At the end, it produces a UX report and a short
action summary.

You (the main orchestrating agent) **manage the browser and run the loop**.
Each screen decision is made by a **simulator subagent** with clean context, and
the final report is built by a **synthesis subagent**. You never choose the path —
you only mechanically execute the intent the simulator already declared.

**Browser:** follow `../user-simulation/browser.md` (relative to this skill's base directory). It
picks whatever browser is already available — nothing to install — and defines the operations used
below: `OPEN`, `READ`, `ACT`, `WAIT`, `SHOT`.

---

## Handoff mode (called from `/user-simulation`)

If the args start with `handoff`, the user already answered everything in the quick start: take
`url`, `scope`, `task`, `profile`, `date`, `lang`, `deliver` from them, **ask nothing**, and skip
straight to Step 1. Role-task mismatches are noted in the report instead of asked (see Step 1).

## Required inputs (direct use)

1. **Profile** — a synthetic user in `.md` (file path or pasted). Must have at least:
   functional role, explicit goal, behavior rules, abandonment rules, forbidden assumptions.
2. **URL** — the live web app to evaluate.
3. **Task** — the thing to accomplish, in natural language (e.g. *"check the alerts"*) — **or**
   `screen` to evaluate only the screen at the URL.

Ask only for what's missing, in one message. Do not invent any of them.

### Profile available? (HARD STOP)

A profile counts as available ONLY if: there is a `.md` in
`user-simulation-tests/simulation/profiles/`, **or** the user gave a path, **or** pasted one.
**Actually check** the folder. If none, STOP and show:

> **🛑 I need a synthetic user to act as.**
> Build one at **https://synthetic.tuggsy.com/**, download the `.md` and drop it in
> `user-simulation-tests/simulation/profiles/` (or paste it here).
> *No profile? Say "user simulation" and I'll infer the users for you.*

Never substitute a default, example, or invented profile.

---

## Step 1 — Prepare

- Read the full profile (if it's a path, use Read). Confirm it has the minimum sections.
- **Screen scope** (`scope=screen` or task `screen`/`infer`): set the task to *"Look at this screen
  and do what you came here to do"* framed by the profile's explicit goal, and set the step cap to
  **3**. The run also stops as soon as an action leaves the screen (a new URL / different page):
  record it as **Goal reached — would continue**.
- **Role-task fit check**: compare the task against the profile's "Suitable task types" and
  "Unsuitable task types". On a clear mismatch: in direct use, warn and ask (run anyway / change
  task or profile); in handoff mode, run anyway. Either way note the mismatch in the report header.
- Set the **step cap** (anti-loop safety net). Default: **15** (3 in screen scope).
- Initialize an empty `steps` list and `memory = null` (memory only exists from step 2 onward).

## Step 2 — Open the app

Pick the driver per `browser.md` §1 (once per session) and `OPEN(url)`. If the app can't be seen,
show `browser.md` §4 and STOP.

## Step 3 — Step-by-step loop

Repeat until a stop condition is met. Each iteration:

  **a. Perceive**: `READ()` — exact text, element roles, states (e.g. `[disabled]`, `[checked]`), and
     a `ref` per element. This IS the screen's visible content. Optionally `SHOT()` when visual
     layout matters as report evidence; the `READ` alone is enough to decide and act.

  **b. Decide**: spawn the `synthetic-screen-evaluator` subagent (Agent tool, `subagent_type: "synthetic-screen-evaluator"`)
     passing, as text:
       - the **full profile**
       - the **task** (the goal)
       - the **screen name/position** (e.g. "Current screen, step N")
       - the **visible content** of the screen (from step a, as readable text)
       - the **memory** from the previous step (omit on step 1)
     The subagent returns ONLY a JSON:
     `{ action, clarityLevel, doubtDetected, reason, abandoned, estimatedTimeSeconds, emotionalState, memory }`.

  **c. Record** the step (number, screen, and the full returned JSON).

  **d. Check stop conditions** (in this order):
       - `abandoned == true` → stop (the user gives up here).
       - the `action` indicates the **goal was reached** (arrived at the screen/state where
         the task would be resolved) → stop with success.
       - screen scope and the action leaves the screen → stop with *Goal reached — would continue*.
       - step count `>= cap` → stop as safety net (mark as "cap reached").
     If stopped, exit the loop.

  **e. Execute**: translate the declared `action` into `ACT` on the `ref` from the current `READ`
     (click / type with submit / press key). For anything that **animates or loads**, `WAIT` for the
     signal, never a guessed number of seconds. If the element the simulator named **doesn't
     exist**, do not fix it or look for alternatives — let the next step perceive it as real
     friction. In read-only mode, a non-executable action ends the run as **Read-only limit**
     (`browser.md` §3) — never as an abandonment.

  **f. Pass memory**: `memory = memory` returned. Goes to the next step.

## Step 4 — Synthesis

Spawn the `synthetic-flow-synthesizer` subagent (`subagent_type: "synthetic-flow-synthesizer"`) passing the complete
`steps` list (full JSON for each) + the profile + the task + the stop reason. Returns the report.

## Step 5 — Deliver

1. Save the report to `user-simulation-tests/simulation/results/<YYYY-MM-DD>-<profile-slug>.md`
   (create the folder if it doesn't exist).
2. **`deliver=none`** (handoff inside a combined run): print one line —
   `✔ Simulation done — <profile-slug>: <Result> (N steps)` — and return to the caller with
   `{ label: "U1", role, result, steps, reportPath, profilePath }`. Stop here.
3. Otherwise spawn the `summary-synthesizer` subagent (`subagent_type: "summary-synthesizer"`)
   passing as text: `LANGUAGE` (handoff `lang`, else the conversation's language), `META` (url,
   domain, date, test=simulation, users=own, scope + task, driver), `USERS` (the one user as U1),
   `REPORTS` (none besides the user report), `LINK_BASE: user-simulation-tests/` — paths relative
   to it. Save the reply to `user-simulation-tests/<YYYY-MM-DD>-<domain>-<profile-slug>-summary.md`
   and show exactly:

```
✅ Done — <domain>

<the summary, verbatim>

📄 Summary saved: user-simulation-tests/<filename>
```

Do **NOT** paste the full report in chat — the summary links to it.

---

## Orchestrator rules

- **Profile first, no exceptions.** Never open the app or spawn a subagent until a real profile is
  in hand. If the user says "just run it" without a profile, STOP and ask; never improvise one.
- **Don't steer the path.** You only execute the intent the simulator declared. Don't choose
  what to click "to move forward."
- **Act by `ref`, not by guessing.** Always use the `ref` from the latest `READ`, and re-`READ` after
  each action so refs are fresh.
- **Don't give the simulator extra context.** It only receives profile + current screen + its memory.
  None of your knowledge about the project, framework, or future screens.
- **Prototype-aware**: the simulator already distinguishes dummy data noise from flow friction — don't
  intervene in that.
- Always respect the **step cap** as a safety net, even if the flow seems not to end.
