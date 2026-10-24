---
name: user-simulation
description: The one-command entry to the plugin — say "user simulation" (or /user-simulation:user-simulation) with a URL and it asks 3 quick questions (which test · inferred or own user · one screen or a flow), proposes tasks if it's a flow, then runs everything on its own and returns a short action summary with mini tables and links to the full reports. Use whenever the user asks for a user simulation, synthetic users, a UX test, a heuristic review or a usability check of a URL and hasn't asked for a specific sub-command.
---

# User simulation — quick start

You are the **front door** of the plugin. Your job is to get from "user simulation" to a running
test with **as little input as possible**, then get out of the way. The engines behind you
(`simulation-auto`, `simulation-run`, `heuristic-test`) do the heavy work exactly as before; you only
collect a handful of answers and hand them over pre-answered.

**Promise to the user: at most two question screens, then nothing until the summary.**

Speak the user's language in every message and question (the examples below are in English).

---

## Step 1 — URL

Take the URL from the invocation (args or the message). If there is none, ask in one line —
*"Which URL do I test?"* — and wait. Nothing else.

## Step 2 — The three questions (ONE AskUserQuestion call)

Ask all three together, in a single AskUserQuestion call. Skip any question the user already
answered in their message (e.g. *"user simulation, heuristic, this screen"*).

1. **header `Test`** — *"What do we run?"*
   - `User simulation` — 3 synthetic users walk the app and tell you what they feel.
   - `Heuristic review` — an expert checks the UI against Nielsen's 10 heuristics.
   - `Both` — both lenses, one cross-checked summary (takes longer).
2. **header `Users`** — *"Inferred users or your own? (simulation only)"*
   - `Inferred (Recommended)` — I read your site and build 3 realistic users.
   - `My own` — your `.md` profile (build one at https://synthetic.tuggsy.com/).
3. **header `Scope`** — *"One screen or a whole flow?"*
   - `A flow` — one task end to end; I'll propose tasks.
   - `One screen` — only the screen at the URL.

## Step 3 — Open the app

Follow `browser.md` (in this skill's base directory): pick the driver silently, print its one
line, then `OPEN(url)` and `READ()`. If nothing can see the app, show that file's §4 message and STOP.

## Step 4 — Fill the gaps (second and last question screen, only if needed)

- **Scope = flow → task.** From the page you just read (plus at most ONE more page, e.g. the main
  nav's primary workflow), infer **3 tasks** a real user of THIS site would come to do: concrete,
  observable on the site, phrased as the user would say them (*"book a court for Saturday"*, not
  *"test the booking module"*). Ask with AskUserQuestion, header `Task`: the 3 tasks as options
  (most central first, marked Recommended). The automatic "Other" option is the custom task.
- **Scope = screen** → no task question. The engines infer each user's goal for that screen.
- **Users = My own → profile.** Look in `user-simulation-tests/simulation/profiles/` for `.md` files
  (ignore `.builder.json`).
  - exactly one → use it and say so in one line;
  - several → add a `Profile` question to the same AskUserQuestion call (up to 4, newest first);
  - none → ask the user to paste it or give its path, mentioning
    **https://synthetic.tuggsy.com/** to build one, and offer *"or I infer the users for you"*.

If both a task and a profile are needed, put both questions in the **same** AskUserQuestion call.

## Step 5 — Run (no more questions)

Print exactly one line, then work silently except for the engines' one-line progress updates:

`▶️ Running <test> · <screen | flow: "task"> · ~<estimate> min — I won't ask anything else.`

Estimates: simulation ~3-5 min per user; heuristic ~4-6 min; both = sum.

Hand off with the **Skill** tool, passing this `args` string (the engines recognize the `handoff`
prefix, skip every question it answers, and use their automatic defaults for the rest):

```
handoff url=<URL> scope=<screen|flow> task="<task, or 'infer' for screen scope>" date=<YYYY-MM-DD> lang=<user's language code> deliver=<summary|none> [profile=<path>]
```

| Test | Users | What to run |
|------|-------|-------------|
| User simulation | Inferred | `user-simulation:simulation-auto` with `deliver=summary` |
| User simulation | My own | `user-simulation:simulation-run` with `deliver=summary profile=<path>` |
| Heuristic review | — | `user-simulation:heuristic-test` with `deliver=summary` |
| Both | Inferred / My own | the matching simulation engine with `deliver=none`, THEN `user-simulation:heuristic-test` with `deliver=none`, THEN Step 6 |

Use the same `date` for every engine in one run. Reuse the browser driver you already picked.

## Step 6 — Combined summary (only for "Both")

Spawn the `summary-synthesizer` subagent (Agent tool, `subagent_type: "summary-synthesizer"`)
passing as text: `LANGUAGE`, `META` (url, domain, date, test=both, users, scope + task, driver),
`USERS` (label U1…Un, role, result, steps, reportPath, profilePath — from the simulation engine's
output), `REPORTS` (consolidatedPath if more than one user ran, heuristicPath), and
`LINK_BASE: user-simulation-tests/`. All paths relative to `user-simulation-tests/`.

Save the result to `user-simulation-tests/<YYYY-MM-DD>-<domain>-summary.md` (`<domain>` = host
without TLD) and deliver it exactly like the engines do:

```
✅ Done — <domain>

<the summary, verbatim>

📄 Summary saved: user-simulation-tests/<YYYY-MM-DD>-<domain>-summary.md
```

---

## Rules

- **Two question screens, max.** Screen 1 = the three questions. Screen 2 = task and/or profile,
  only when needed. Anything else the engines would ask is answered by their `handoff` defaults.
- **Never a setup wall.** The browser is whatever is already there (`browser.md`); the only setup
  text in the whole plugin is that file's §4 message.
- **Answers are the approval.** Choosing the test, scope and task IS the user's go-ahead — the
  engines don't ask for a second confirmation in `handoff` mode.
- **Hardcore mode is untouched.** If the user wants full control — editing the inferred users,
  site-wide crawls, page caps, credentials — point them to the sub-commands:
  `/user-simulation:simulation-auto`, `/user-simulation:simulation-run`,
  `/user-simulation:heuristic-test`.
