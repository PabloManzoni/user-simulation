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

---

## On launch — show this to the user FIRST

The moment this skill is invoked, print the quick start below **verbatim** (don't expand it),
then run the Step 0 checks to see how far along the user already is and guide them from there.

> **🧪 User Simulation — quick start**
> Run a synthetic user through your live web app and get a UX report.
>
> **1. Connect the browser** — install **Claude in Chrome** from the Chrome Web Store and connect it.
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

These checks gate the entire skill. Until BOTH pass, you must NOT open the app, navigate, take
screenshots, spawn subagents, or evaluate any screen. **Running the simulation without a confirmed
profile is a failure.** Never start "reviewing" the app on your own initiative, and never substitute
a default, example, or invented profile.

Run the checks in order. If either fails, show its block and **STOP** — wait for the user to come
back. Do not continue past a failed check under any circumstances, even if the user just said "run".

### 0a. Browser connected?

Load and call `mcp__Claude_in_Chrome__list_connected_browsers` (via ToolSearch
`select:mcp__Claude_in_Chrome__list_connected_browsers`).

If it returns `[]`:

> **⚠️ Setup needed — Chrome extension**
>
> The simulator needs the **Claude in Chrome** extension to see and interact with the browser.
>
> 1. Open Chrome and go to the Chrome Web Store
> 2. Search for **"Claude in Chrome"** and install it
> 3. Click the extension icon and connect it to this session
> 4. Once connected, tell me and I'll continue.

**STOP here.** Do not proceed without a connected browser.

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
- Set the **step cap** (anti-loop safety net). Default: **15**. The user can change it.
- Initialize an empty `steps` list and `memory = null` (memory only exists from step 2 onward).

## Step 2 — Open the app

Load the Claude in Chrome tools you'll need (navigate, screenshot,
get_page_text or read_page, find, computer/click) via ToolSearch. Navigate to the URL.

## Step 3 — Step-by-step loop

Repeat until a stop condition is met. Each iteration:

  **a. Perceive**: take a screenshot (stays in the step record) and extract the **visible content**
     of the current screen (text and interactive elements) with get_page_text/read_page.

  **b. Decide**: spawn the `screen-evaluator` subagent (Agent tool, `subagent_type: "screen-evaluator"`)
     passing, as text:
       - the **full profile**
       - the **task** (the goal)
       - the **screen name/position** (e.g. "Current screen, step N")
       - the **visible content** of the screen (from step a)
       - the **memory** from the previous step (omit on step 1)
     The subagent returns ONLY a JSON:
     `{ action, clarityLevel, doubtDetected, reason, abandoned, estimatedTimeSeconds, emotionalState, memory }`.

  **c. Record** the step (number, screen, screenshot, and the full returned JSON).

  **d. Check stop conditions** (in this order):
       - `abandoned == true` → stop (the user gives up here).
       - the `action` indicates the **goal was reached** (arrived at the screen/state where
         the task would be resolved) → stop with success.
       - step count `>= cap` → stop as safety net (mark as "cap reached").
     If stopped, exit the loop.

  **e. Execute**: translate the declared `action` into a real Chrome action (click the element
     the simulator named, type in a field, etc.) as faithfully as possible. If the element it
     named **doesn't exist**, do not fix it or look for alternatives — let the next step perceive
     it as real friction.

  **f. Pass memory**: `memory = memory` returned. Goes to the next step.

## Step 4 — Synthesis

Spawn the `flow-analysis` subagent (`subagent_type: "flow-analysis"`) passing the complete
`steps` list (full JSON for each) + the profile + the task + the stop reason. Returns the report.

## Step 5 — Deliver

- Show the report to the user in chat.
- Save it to `results/<date>-<profile-slug>.md` (create the `results/` folder if it doesn't exist).

---

## Orchestrator rules

- **Profile + browser first, no exceptions.** Never open the app, navigate, screenshot, spawn a
  subagent, or evaluate a screen until Step 0 has passed — a real profile is in hand AND the browser
  is connected. If the user says "just run it" without a profile, STOP and ask; do not improvise,
  default to, or reuse an example profile.
- **Don't steer the path.** You only execute the intent the simulator declared. Don't choose
  what to click "to move forward."
- **Don't give the simulator extra context.** It only receives profile + current screen + its memory.
  None of your knowledge about the project, framework, or future screens.
- **Prototype-aware**: the simulator already distinguishes dummy data noise from flow friction — don't
  intervene in that.
- Always respect the **step cap** as a safety net, even if the flow seems not to end.
