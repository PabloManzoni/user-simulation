---
name: run
description: Runs a synthetic user over a live web app to evaluate UX step by step (action, clarity, doubt, emotion, memory) and produces a report. Use when the user wants to simulate a profile over an app/URL, test usability, or types /user-simulation:run.
---

# User simulation

You orchestrate a **synthetic user simulation** over a **live web app**: a profile receives
a task, navigates the app clicking toward that goal, and each step records what they do,
how clear it was, their doubts and emotions. At the end, it produces a UX report.

You (the main orchestrating agent) **manage the browser and run the loop**.
Each screen decision is made by a **simulator subagent** with clean context, and
the final report is built by a **synthesis subagent**. You never choose the path —
you only mechanically execute the intent the simulator already declared.

The browser is driven through the **Playwright MCP** server: you perceive each screen with an
accessibility **snapshot** (exact text, roles, states, and a `ref` per element) and act on elements
by their `ref` — not by pixel coordinates.

---

## On launch — run Step 0 checks silently first

Run the Step 0 checks immediately. Do NOT print the quick start unless a check fails.
If both checks pass and the user already provided the 3 required inputs, start the simulation directly.
If both checks pass but inputs are missing, ask for the missing ones only (profile / URL / task).
Only show the quick start block below when a preflight check fails and the user needs setup guidance.

> **🧪 User Simulation — quick start**
> Run a synthetic user through your live web app and get a UX report.
>
> **1. Set up the browser (one time):**
>    ```
>    claude mcp add playwright -- npx @playwright/mcp@latest
>    npx playwright install chromium
>    ```
>    Then restart Claude Code and run `/reload-plugins`.
> **2. Get a profile** — build your synthetic user at **https://synthetic.tuggsy.com/**, download the `.md`, and drop it into `profiles/`.
> **3. Tell me 3 things** — the profile, the app URL, and the task to test (e.g. *"create a raffle and pick a winner"*).
>
> Then I'll open the app, walk it screen by screen as that user, and save an English report in `results/`.

---

## Required inputs (all 3 are mandatory)

1. **Profile** — a synthetic user in `.md` (file path or pasted). Must have at least:
   functional role, explicit goal, behavior rules, abandonment rules, forbidden assumptions.
2. **URL** — the live web app to evaluate (already running).
3. **Task** — the thing to accomplish, in natural language (e.g. *"check the alerts"*).

If any of the three is missing, ask for it before continuing. Do not invent any.

---

## Step 0 — Preflight checks (HARD STOP — do these before ANYTHING else)

These checks gate the entire skill. Until BOTH pass, you must NOT open the app, navigate, capture
snapshots, spawn subagents, or evaluate any screen. **Running the simulation without a confirmed
profile is a failure.** Never start "reviewing" the app on your own initiative, and never substitute
a default, example, or invented profile.

Run the checks in order. If either fails, show its block and **STOP** — wait for the user to come
back. Do not continue past a failed check under any circumstances, even if the user just said "run".

### 0a. Playwright browser available?

The simulator drives the browser through the **Playwright MCP** server. Check it's available: try to
load its tools via ToolSearch `select:mcp__playwright__browser_navigate,mcp__playwright__browser_snapshot`.

If the tools are NOT found (the server isn't configured, or this session started before it was added):

> **⚠️ Setup needed — Playwright browser**
>
> The simulator drives the browser through the **Playwright MCP** server, which isn't available yet.
>
> 1. In your terminal, run: `claude mcp add playwright -- npx @playwright/mcp@latest`
> 2. **Restart Claude Code** — MCP tools only load when the session starts.
> 3. Tell me when it's back and I'll continue.

**STOP here.** Do not proceed without Playwright available.

### 0b. Profile available?

A synthetic user profile is **required** — it's the user you act as; with no profile there is
literally nothing to simulate. A profile counts as available ONLY if one of these is true:

- there is at least one `.md` file inside a `profiles/` folder in the current working directory, **or**
- the user gave you a file path to a profile `.md`, **or**
- the user pasted a full profile into the chat.

**Actually check** — list the `profiles/` folder; don't assume. If none of the three is true,
**STOP immediately** and show exactly this. Do not open the app, do not start any step, and do not
invent or reuse an example profile:

> **🛑 I can't run yet — there's no synthetic user profile.**
>
> A profile is the user I'm supposed to act as. Without one there's nothing to simulate, so I'm
> stopping here on purpose.
>
> **Do this first:**
> 1. Create a profile at **https://synthetic.tuggsy.com/**
> 2. Download the `.md` and drop it into a **`profiles/`** folder in this project
>    *(or paste it here in the chat, or give me the file path).*
> 3. Tell me when it's ready — then I'll ask for the app URL and the task, and start.

Wait for the user. Only continue once a real profile is in hand.

---

## Step 1 — Prepare

- Read the full profile (if it's a path, use Read). Confirm it has the minimum sections.
- **Role-task fit check**: compare the task against the profile's "Suitable task types" and
  "Unsuitable task types" sections. If the task clearly matches an unsuitable type (e.g. an admin
  configuration task given to an end-user profile), warn the user BEFORE running — a mismatched
  pair produces a report that looks valid but measures nothing real. Show which unsuitable type it
  matches and ask: run it anyway, or fix the pairing (different task or different profile). Proceed
  only with explicit confirmation; if they confirm, note the mismatch in the report header.
- Set the **step cap** (anti-loop safety net). Default: **15**. The user can change it.
- Initialize an empty `steps` list and `memory = null` (memory only exists from step 2 onward).

## Step 2 — Open the app

Load the Playwright tools you'll need via ToolSearch (select these by name):
`mcp__playwright__browser_navigate`, `browser_snapshot`, `browser_click`, `browser_type`,
`browser_press_key`, `browser_wait_for`, `browser_take_screenshot`.
Then `browser_navigate` to the URL.

## Step 3 — Step-by-step loop

Repeat until a stop condition is met. Each iteration:

  **a. Perceive**: capture a `browser_snapshot` — the accessibility tree with exact text, element
     roles, states (e.g. `[disabled]`, `[checked]`), and a `ref` for each element. This IS the
     screen's visible content. Optionally also `browser_take_screenshot` when visual layout matters
     as report evidence; the snapshot alone is enough to decide and act.

  **b. Decide**: spawn the `synthetic-screen-evaluator` subagent (Agent tool, `subagent_type: "synthetic-screen-evaluator"`)
     passing, as text:
       - the **full profile**
       - the **task** (the goal)
       - the **screen name/position** (e.g. "Current screen, step N")
       - the **visible content** of the screen (the snapshot from step a, as readable text)
       - the **memory** from the previous step (omit on step 1)
     The subagent returns ONLY a JSON:
     `{ action, clarityLevel, doubtDetected, reason, abandoned, estimatedTimeSeconds, emotionalState, memory }`.

  **c. Record** the step (number, screen, snapshot, and the full returned JSON).

  **d. Check stop conditions** (in this order):
       - `abandoned == true` → stop (the user gives up here).
       - the `action` indicates the **goal was reached** (arrived at the screen/state where
         the task would be resolved) → stop with success.
       - step count `>= cap` → stop as safety net (mark as "cap reached").
     If stopped, exit the loop.

  **e. Execute**: translate the declared `action` into a real Playwright action, using the `ref` from
     the current snapshot:
       - click → `browser_click` (`target` = the element's `ref`; `element` = a human-readable description)
       - type → `browser_type` (`target` = ref, `text`, and `submit: true` to press Enter after)
       - press a key → `browser_press_key`
     For anything that **animates or loads** (spinners, transitions, async results), use
     `browser_wait_for` (wait for the expected text to appear, or `textGone` for it to disappear) —
     wait for the *signal*, not a guessed number of seconds. If the element the simulator named
     **doesn't exist** in the snapshot, do not fix it or look for alternatives — let the next step
     perceive it as real friction.

  **f. Pass memory**: `memory = memory` returned. Goes to the next step.

## Step 4 — Synthesis

Spawn the `synthetic-flow-synthesizer` subagent (`subagent_type: "synthetic-flow-synthesizer"`) passing the complete
`steps` list (full JSON for each) + the profile + the task + the stop reason. Returns the report.

## Step 5 — Deliver

1. Save the report to `results/<YYYY-MM-DD>-<profile-slug>.md` (create `results/` if it doesn't exist).
2. Do **NOT** paste the full report in chat. Instead, show exactly this structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅  SIMULATION COMPLETE
📄  Report saved: results/<filename>.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Result:** <Goal reached / Abandoned at step N / Step cap reached>
**Steps:** N · **~Xs total**

**What happened:**
- <One sentence: where was friction highest and why>
- <One sentence: how trust/confidence evolved across the run>
- <One sentence: Fix this first — pulled verbatim from the report>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POTENTIAL IMPROVEMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<Pull the Potential improvements table from the report and show it here>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Orchestrator rules

- **Profile + Playwright first, no exceptions.** Never open the app, navigate, snapshot, spawn a
  subagent, or evaluate a screen until Step 0 has passed — a real profile is in hand AND Playwright
  is available. If the user says "just run it" without a profile, STOP and ask; do not improvise,
  default to, or reuse an example profile.
- **Don't steer the path.** You only execute the intent the simulator declared. Don't choose
  what to click "to move forward."
- **Act by `ref`, not by guessing.** Always click/type using the `ref` from the latest snapshot.
  Re-snapshot after each action so refs are fresh.
- **Don't give the simulator extra context.** It only receives profile + current screen + its memory.
  None of your knowledge about the project, framework, or future screens.
- **Prototype-aware**: the simulator already distinguishes dummy data noise from flow friction — don't
  intervene in that.
- Always respect the **step cap** as a safety net, even if the flow seems not to end.
