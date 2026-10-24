# Browser — use whatever is already there

The single source of truth for **how every skill in this plugin drives the browser**. No skill
requires installing anything: you pick the first browser that is already available, in this order,
and fall back to read-only mode when there is none. **Never block the user with a setup wall.**

The skills describe browser work with five abstract operations — `OPEN`, `READ`, `ACT`, `WAIT`,
`SHOT` — and this file maps them onto the concrete tool you found.

---

## 1. Pick the driver (silently, once per session)

Run ONE ToolSearch with query `browser navigate read page click` (max_results 15) and look at the
tool names it returns. Choose the first match:

| # | Driver | Recognized by | Why this order |
|---|--------|---------------|----------------|
| 1 | **Playwright MCP** | `mcp__playwright__*` | Already set up by a power user: isolated, headless, most deterministic. Never install it. |
| 2 | **Built-in browser** (Claude desktop app) | `mcp__Claude_Browser__*` or `mcp__remote-devices__Claude_Browser__*` | Zero install — ships with the desktop app. |
| 3 | **Claude in Chrome** | `mcp__claude-in-chrome__*` | Zero install if the extension is on. Acts in the person's real Chrome: always work in a **new tab**. |
| 4 | **Read-only** | none of the above | Always available: WebFetch for public URLs, `curl -sL` via Bash for localhost / private hosts. |

- **Not found on the first try?** MCP servers connect asynchronously while a session starts — retry
  the same ToolSearch once before falling through.
- If only an **`enable__…`** tool shows up for driver 2 or 3 (e.g. `enable__mcp__claude-in-chrome`,
  `enable__mcp__remote-devices__Claude_Browser`), that IS the one-click way to turn it on: call it,
  then re-run the ToolSearch.
- Before the first call to driver 2 or 3, **load its skill** (`anthropic-skills:built-in-browser` or
  `anthropic-skills:chrome-browser`) when it is listed — it names the exact tools and their quirks.
- Say which driver you use in **one line**, e.g. `🌐 Browser: built-in (desktop app)`. Nothing else.

## 2. The five operations

| Op | Meaning | Playwright | Built-in / Chrome | Read-only |
|----|---------|-----------|-------------------|-----------|
| `OPEN(url)` | Go to a URL | `browser_navigate` | its navigate tool (Chrome: in a new tab) | WebFetch the URL (or `curl -sL <url>` for localhost/private) |
| `READ()` | Perceive the screen: exact text, roles, states, a `ref` per interactive element | `browser_snapshot` | its page-reading tool that returns the **accessibility tree with element refs** (e.g. `read_page`); page-text tool only as a supplement | The fetched page reduced to: title, headings, visible text, links (label → href), buttons, form fields with labels |
| `ACT(kind, ref, text?)` | click / type (+submit) / press key, on the `ref` from the LATEST `READ` | `browser_click` · `browser_type` · `browser_press_key` | its click / form-input / key tools, by ref | Only **click on a link** = `OPEN(href)`; a GET form = `OPEN(action?query)`. Everything else is **not executable** (see §3) |
| `WAIT(signal)` | Wait for something to appear/disappear — the signal, never a guessed number of seconds | `browser_wait_for` (text / textGone) | its wait tool, or re-`READ` up to 3 times until the signal shows | n/a |
| `SHOT(name)` | Screenshot as visual evidence | `browser_take_screenshot` (filename `<name>.png`) | its screenshot tool; if it can't save a file, keep no path | none — skip |

Rules shared by every driver:

- **Act by ref, not by guessing.** Always use refs from the latest `READ`; re-`READ` after each
  `ACT` so refs are fresh.
- **Context hygiene.** A `READ` result is big: extract what the step needs and drop it.
- `SHOT` is optional evidence. When a driver can't produce a screenshot file, the heuristic expert
  rates visually-dependent heuristics (#8) as *not observable* — that is the protocol, not a bug.

## 3. Read-only mode — honest limits

Read-only works well for **public, server-rendered pages** and **link-based navigation**. It cannot
type into forms, run JavaScript, or see client-rendered apps.

- **Empty shell detection:** if a fetched page has almost no visible text (< ~200 characters) and is
  mostly script bundles / an empty `<div id="root">`, it is a client-rendered app — read-only can't
  see it. Stop and show the *no browser* message below (one time, short).
- **Non-executable action** (the simulated user wants to type, or click a JS button): stop that run
  with result **`Read-only limit`** and record the intended action. It is NOT a UX abandonment and
  must never be reported as one.
- Tell the user once, at the start: `🌐 Browser: read-only — I can read pages and follow links, not
  fill forms.`

## 4. The only setup message (when nothing can see the app)

Show this — in the user's language — and STOP. No other setup text anywhere in the plugin.

> **🌐 I need a browser to walk `<url>`** (it's a JavaScript app, or it needs forms).
> Any of these works, nothing to install:
> - Open this conversation in the **Claude desktop app** — it has a built-in browser.
> - Or turn on **Claude in Chrome** (in Claude Code: `/chrome`).
>
> *Hardcore mode:* Playwright MCP also works — see the README.

## 5. Safety rails (every driver, every skill)

- Never complete payments, deletions, or irreversible submissions. Use obviously fake placeholder
  data in forms. Stop before a final real submission unless the user confirmed a test environment.
- With Claude in Chrome you are in the person's real browser with their real sessions: open your
  own tab, never touch their other tabs, never change account settings.
