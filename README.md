# User Simulation

A Claude Code plugin that runs a **synthetic user** over your **live web app** and gives you a
step-by-step UX report — what they do on each screen, how clear it is, where they hesitate, and where
they give up.

It's app-agnostic: it points at any web app by URL. The synthetic user only uses what's visible on
screen (no guessing, no filling gaps), so it surfaces *real* friction instead of a polished
best-case walkthrough.

## What you need

- **Claude Code** — this plugin runs inside it.
- **Playwright MCP** — so the simulator can drive the browser (no Chrome extension needed). One-time
  setup, see Install below.
- **A synthetic user profile** (`.md`) — build one in a couple of minutes at
  **https://synthetic.tuggsy.com/**, download it, and drop it into a `profiles/` folder.

## Install

**1. Add the Playwright MCP server** (one time), then restart Claude Code:
```
claude mcp add playwright -- npx @playwright/mcp@latest
```

**2. Install the plugin:**
```
/plugin marketplace add PabloManzoni/user-simulation
/plugin install user-simulation@pablom-plugins
```

## Use

1. Put your profile `.md` in a `profiles/` folder in your project.
2. Make sure your web app is running (e.g. `http://localhost:5173`).
3. Run:
   ```
   /user-simulation:run
   ```
4. When asked, give it three things: **which profile**, the **app URL**, and the **task** to test
   (e.g. *"create a raffle and pick a winner"*).

The plugin opens your app, walks through it screen by screen as that user, and saves a report to
`results/`.

## What you get

A Markdown report with: a step-by-step table (action · clarity · doubt), the highest friction points,
a first-person emotional summary, detected risks, and a structural design insight.

## How it works

- An **orchestrator** (the skill) drives the browser via **Playwright MCP** and runs a
  *perceive → decide → execute* loop. It perceives each screen as an accessibility **snapshot**
  (text + roles + states) and acts on elements by reference — robust, no pixel-guessing.
- A **screen-evaluator** subagent reacts to ONE screen at a time, in character, using only what's
  visible — this isolation is what keeps it from "compensating" for bad design.
- A **flow-analysis** subagent reads the whole run and writes the report.

The method (synthetic users as constrained decision agents) is described in [framework.md](framework.md).

## Example

See [examples/profiles/diego-nakamura-b7k.md](examples/profiles/diego-nakamura-b7k.md) for a real,
ready-to-use profile.

## Credits

Built by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/), Product Designer at
[Kaizen Softworks](https://www.kzsoftworks.com/).

Licensed under the MIT License — see [LICENSE](LICENSE).
