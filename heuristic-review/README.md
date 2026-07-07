# Heuristic Review

A Claude Code plugin that runs a **Nielsen heuristic evaluation** over a live web app and returns a
**prioritized findings table** — detection by an expert agent under forced enumeration of the ten
usability heuristics, severity judged by three synthetic rater personas.

It is the inspection sibling of the [user-simulation](../README.md) plugin:

| | **user-simulation** | **heuristic-review** (this plugin) |
|---|---|---|
| Lens | Lived experience of one synthetic user | Expert inspection against principles |
| Finds | Friction, doubt, emotion, abandonment | Violations of Nielsen's 10 heuristics |
| Output | Step-by-step narrative + UX report | Prioritized findings table |

Simulation tells you where one user bleeds. Inspection tells you where the interface is sharp.
Run both.

## How it works

1. **Research** — the orchestrator reads the site and writes a business summary.
2. **Personas** — three fixed rater archetypes (power user · average user · low-digital-literacy
   user) are contextualized to the business and saved to `personas/` for reuse.
3. **Capture** — the scope is captured as accessibility snapshots + screenshots. Three modes:
   - `screen` — one screen
   - `flow` — a declared flow, walked deterministically (with optional error probing)
   - `site` — a same-origin crawl, capped (default 8 pages)
4. **Detection** — the `heuristic-expert-evaluator` agent sweeps every screen against each of the
   10 heuristics **in order** and must record an explicit result per heuristic (violations / clean /
   not observable). Every finding quotes verbatim evidence. It also scores **business impact**
   (1–3) from the business summary.
5. **Rating** — the three personas independently rate each finding's **usability impact** (1–3) and
   flag what is important *to them*. They never see the expert's scores (anti-anchoring).
6. **Priority** — computed deterministically by the orchestrator:
   `priority = businessImpact × avg(usabilityImpact)` (1.0–9.0), ties broken by rater convergence.
7. **Report** — an English Markdown report in `results/`: coverage ledger (all 10 heuristics),
   findings table, per-finding detail with rater divergence, "fix this first", method note.

The methodology is documented in [framework.md](framework.md) — grounded in Nielsen & Molich (1990),
Nielsen (1994), and Nielsen & Mack (1994).

## Requirements

- Claude Code with plugins enabled
- Chrome/Chromium
- The **Playwright MCP** server:
  ```
  claude mcp add playwright -- npx @playwright/mcp@latest
  npx playwright install chromium
  ```

## Install

```
/plugin marketplace add PabloManzoni/user-simulation
/plugin install heuristic-review@pablom-plugins
```

Then restart Claude Code.

## Use

```
/heuristic-review:evaluate https://your-app.com site
/heuristic-review:evaluate https://your-app.com flow "home → contact → submit the form"
/heuristic-review:evaluate https://your-app.com/pricing screen
```

Or just describe what you want in natural language — "run a heuristic evaluation on my site".

## What you get

- `results/<date>-heuristic-<domain>-<scope>.md` — the full report
- `personas/<domain>-*.md` — the three rater personas (reused on the next run)
- `personas/<domain>.research.md` — the business summary (reused together with the personas on the
  next run)
- A chat summary with the top findings by priority and the single "fix this first" item

## Credits & license

MIT — by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/), Product Designer at
[Kaizen Softworks](https://www.kzsoftworks.com/).
