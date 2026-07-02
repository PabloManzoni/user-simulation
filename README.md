# User Simulation

A Claude Code plugin that runs a **synthetic user** over your **live web app** and gives you a
step-by-step UX report — what they do on each screen, how clear it is, where they hesitate, and
where they give up.

It's app-agnostic: it points at any web app by URL. The synthetic user only uses what's visible on
screen (no guessing, no filling gaps), so it surfaces *real* friction instead of a polished
best-case walkthrough.

## Requirements

- **Claude Code** — this plugin runs inside it.
- **Google Chrome** — required. Playwright MCP drives your installed Chrome by default, so Chrome
  must be present on the machine. (No Chrome extension needed.)
- **Playwright MCP** — the browser driver. A one-time install (see below).
- **A synthetic user profile** (`.md`) — build one at **https://synthetic.tuggsy.com/**, download
  it, and drop it into a `profiles/` folder in your project.

## Install

**Step 1 — In your terminal:** install the Playwright MCP server and its browser:

```
claude mcp add playwright -- npx @playwright/mcp@latest
npx playwright install chromium
```

**Step 2 — In Claude Code:** add the plugin source:

```
/plugin marketplace add PabloManzoni/user-simulation
```

**Step 3 — In Claude Code:** install the plugin:

```
/plugin install user-simulation@pablom-plugins
```

**Step 4 — In Claude Code:** reload plugins:

```
/reload-plugins
```

> To update later: run `/plugin uninstall user-simulation` in Claude Code, then repeat Steps 3–4.

## Use

1. Build your profile at **https://synthetic.tuggsy.com/**, download the `.md`, and put it in a `profiles/` folder in your project — or drag and drop it directly into the Claude Code chat.
2. Make sure your web app is running (e.g. `http://localhost:5173`).
3. Run:
   ```
   /user-simulation:run
   ```
4. When asked, give it three things: **which profile**, the **app URL**, and the **task** to test
   (e.g. *"create a raffle and pick a winner"*).

The plugin opens your app, walks through it screen by screen as that user, and saves a Markdown
report to `results/`.

## Automatic mode — autopilot

No profiles yet? Let the plugin build them:

```
/user-simulation:autopilot
```

Give it just your app's **URL** (plus an optional one-line description of the business). It
researches the site, **proposes** synthetic users with their tasks, and waits for you: edit the
proposal in plain language — or say *"run as is"*. Then it generates the profiles (the `.md` for
simulation plus a `.builder.json` you can re-open in the [profile builder](https://synthetic.tuggsy.com/)
to edit by hand), runs one simulation per user, and saves the individual reports plus a
**consolidated report** that ranks findings by how many users hit them — including contrasted
user pairs that reveal who your design favors.

## What you get

When the run finishes, you see a summary in chat: result, friction peak, and a table of actionable
improvements ordered by impact. The full report — per-screen walkthrough, emotional arc,
detected risks, structural insight, and a single "Fix this first" — is saved to `results/`.

## How it works

- An **orchestrator** (the skill) drives the browser via Playwright MCP and runs a
  *perceive → decide → execute* loop. It reads each screen as an accessibility snapshot
  (text + roles + states) and acts on elements by reference — no pixel-guessing.
- A **synthetic-screen-evaluator** subagent reacts to one screen at a time, in character, using only
  what's visible. This isolation is what keeps it from compensating for bad design.
- A **synthetic-flow-synthesizer** subagent reads the full run and writes the report.
- In autopilot mode, a **synthetic-profile-generator** subagent builds each approved user's profile,
  and a **synthetic-autopilot-synthesizer** subagent consolidates all the runs into one report.

The method is described in [framework.md](framework.md).

## Example profile

See [examples/profiles/diego-nakamura-b7k.md](examples/profiles/diego-nakamura-b7k.md) for a
ready-to-use profile.

## Credits

Built by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/), Product Designer at
[Kaizen Softworks](https://www.kzsoftworks.com/).

Licensed under the MIT License — see [LICENSE](LICENSE).
