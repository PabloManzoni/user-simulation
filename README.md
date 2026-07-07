# User Simulation

A Claude Code plugin for **synthetic UX testing** over your **live web app** — two complementary
lenses on the same interface:

| | **Experience** (simulation) | **Inspection** (heuristics) |
|---|---|---|
| The question | *How does it feel to use this?* | *Does it violate known usability principles?* |
| Who looks | A synthetic user, in character | An expert evaluator + rater personas |
| You get | A step-by-step run: friction, doubt, emotion, where they give up | A prioritized findings table against Nielsen's 10 heuristics |

Simulation tells you where one user bleeds. Inspection tells you where the interface is sharp.
Run both.

It's app-agnostic: it points at any web app by URL, and drives the browser through Playwright MCP
using accessibility snapshots (no pixel-guessing).

## The three commands

| Command | Lens | What defines the test |
|---|---|---|
| `/user-simulation:simulation-run` | Experience | **You** — you bring the profile, the URL, and the task |
| `/user-simulation:simulation-auto` | Experience | **The site** — from just a URL it infers who uses it, proposes users, and consolidates the runs |
| `/user-simulation:heuristic-test` | Inspection | **The heuristics** — an expert audits the UI; you bring the URL and a scope (screen / flow / site) |

## Requirements

- **Claude Code** — this plugin runs inside it.
- **Google Chrome** — Playwright MCP drives your installed Chrome by default.
- **Playwright MCP** — the browser driver. A one-time install (see below).
- **A synthetic user profile** (`.md`) — only for `simulation-run`. Build one at
  **https://synthetic.tuggsy.com/**. (`simulation-auto` and `heuristic-test` build their own.)

## Install

**Step 1 — In your terminal:** install the Playwright MCP server and its browser:

```
claude mcp add playwright -- npx @playwright/mcp@latest
npx playwright install chromium
```

**Step 2 — In Claude Code:** add the plugin source and install:

```
/plugin marketplace add PabloManzoni/user-simulation
/plugin install user-simulation@pablom-plugins
```

**Step 3 — In Claude Code:** reload plugins:

```
/reload-plugins
```

> To update later: refresh the catalog with `/plugin marketplace update pablom-plugins`, then
> `/plugin uninstall user-simulation` and reinstall. (In the desktop app you can also press
> **Update** on the plugin's settings page.)

---

## Experience — `simulation-run`

**You know which type of user you want to test.** You bring the profile and define the task.

```
/user-simulation:simulation-run
```

1. Build your profile at **https://synthetic.tuggsy.com/**, download the `.md`, and put it in a
   `user-simulation-tests/simulation/profiles/` folder in your project — or drop it into the chat.
2. Make sure your web app is running (e.g. `http://localhost:5173`).
3. Run the command and give it three things: **which profile**, the **app URL**, and the **task**
   (e.g. *"create a raffle and pick a winner"*).

It walks the app screen by screen as that user and saves a Markdown report to
`user-simulation-tests/simulation/results/`.

## Experience — `simulation-auto`

**From your site, it infers who likely uses it** and lets you choose — and edit — which users to
run. You bring just the URL.

```
/user-simulation:simulation-auto
```

1. It **researches** your site and **proposes** synthetic users, each with their own tasks.
2. **You edit the proposal** in plain language ("remove user 2", "add this task") — or say
   *"run as is"*.
3. It **generates the profiles** — a `.md` for simulating, plus a `.builder.json` you can reopen in
   the [profile builder](https://synthetic.tuggsy.com/).
4. It **runs one simulation per user** and writes a **consolidated report** ranking findings by how
   many users hit them — including contrasted user pairs that reveal who your design favors.

## Inspection — `heuristic-test`

**An expert audits the UI against Nielsen's 10 usability heuristics**, and three synthetic personas
(power user / average / low-digital-literacy) rate each finding's impact. You bring the URL and a
scope.

```
/user-simulation:heuristic-test https://your-app.com site
/user-simulation:heuristic-test https://your-app.com flow "home → contact → submit the form"
/user-simulation:heuristic-test https://your-app.com/pricing screen
```

The expert sweeps every screen against each heuristic under forced enumeration (recording an
explicit verdict per heuristic), quoting verbatim evidence. Priority is computed as
**business impact × average usability impact**, with rater convergence as tiebreaker. You get an
English findings table in `user-simulation-tests/heuristic/results/`, plus reusable personas in
`user-simulation-tests/heuristic/personas/`.

## What you get

Every command ends with a short chat summary — result, where it hurt most, and a single
**"Fix this first"** — and saves the full report under `user-simulation-tests/`:

- **simulation-run / simulation-auto**: per-screen walkthrough, emotional arc, detected risks,
  structural insight; auto adds a cross-user consolidated report.
- **heuristic-test**: a coverage ledger (all 10 heuristics), a prioritized findings table with
  per-finding evidence and per-persona ratings, and the divergences that reveal who each issue hurts.

Everything lands in one `user-simulation-tests/` folder in your project, split by lens:

```
user-simulation-tests/
├── simulation/
│   ├── profiles/    ← the synthetic users (navigators)
│   └── results/     ← simulation-run and simulation-auto reports
└── heuristic/
    ├── personas/    ← the three rater personas + business research
    └── results/     ← heuristic-test reports
```

## How it works

An **orchestrator** (the skill) drives the browser via Playwright MCP with accessibility snapshots
and delegates the judgment to focused subagents with clean context:

- **Experience**: a `synthetic-screen-evaluator` reacts to one screen at a time in character; a
  `synthetic-flow-synthesizer` writes the run report; in `simulation-auto`, a
  `synthetic-profile-generator` builds each user and a `synthetic-autopilot-synthesizer` consolidates.
- **Inspection**: a `heuristic-persona-generator` builds the three rater personas, a
  `heuristic-expert-evaluator` detects violations, three `heuristic-persona-rater` calls score them,
  and a `heuristic-report-synthesizer` writes the report.

The methodology behind each lens:
[frameworks/synthetic-users.md](frameworks/synthetic-users.md) (experience) and
[frameworks/heuristic-test.md](frameworks/heuristic-test.md) (inspection, grounded in Nielsen &
Molich 1990, Nielsen 1994, Nielsen & Mack 1994).

## Example profile

See [examples/profiles/diego-nakamura-b7k.md](examples/profiles/diego-nakamura-b7k.md) for a
ready-to-use simulation profile.

## Credits

Built by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/), Product Designer at
[Kaizen Softworks](https://www.kzsoftworks.com/).

Licensed under the MIT License — see [LICENSE](LICENSE).
