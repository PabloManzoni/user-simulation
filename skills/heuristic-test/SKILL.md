---
name: heuristic-test
description: Runs a Nielsen heuristic evaluation over a live web app — one expert agent inspects every screen in scope against the 10 usability heuristics (forced enumeration), three synthetic rater personas judge each finding, and you get a prioritized findings table. Three scope modes: single screen, specific flow, or whole-site crawl. Use when the user wants a heuristic evaluation / expert UX review / usability audit of a URL, or types /user-simulation:heuristic-test.
---

# Heuristic test

You orchestrate a **controlled heuristic evaluation** of a **live web app**: one expert agent
detects violations of Nielsen's ten usability heuristics across the captured scope, three synthetic
rater personas independently judge each finding's usability impact, and the run produces a
**prioritized findings table** (business impact × usability impact, with rater convergence).

You (the main orchestrating agent) **manage the browser, capture the scope, and compute the
numbers**. Detection belongs to the `heuristic-expert-evaluator` subagent, rating to three
`heuristic-persona-rater` subagents, and the report to `heuristic-report-synthesizer`. You never
detect findings yourself and you never adjust a score — you capture, validate, compute, and deliver.

**Browser:** follow `../user-simulation/browser.md` (relative to this skill's base directory) — it
picks whatever browser is already available, nothing to install, and defines `OPEN`, `READ`, `ACT`,
`WAIT`, `SHOT`. You perceive each screen with a `READ` (the accessibility **snapshot**) plus one
`SHOT` per screen when the driver can take one (evidence for visually-dependent heuristics).

This is the **inspection** lens of the `user-simulation` plugin — the sibling of the
`simulation-run` / `simulation-auto` skills: those simulate how a user *feels*; this one audits
what the interface *violates*. Reports are always written in **English**; the action summary in
the user's language.

---

## On launch — run Step 0 checks silently first

Run the Step 0 checks immediately. If the user already provided URL + mode (or it's a handoff),
start directly. If inputs are missing, show the quick start below and ask only for the missing ones.

> **🔍 Heuristic Test — quick start**
> Tell me the app **URL** and the **mode**:
> · `screen` — one screen (its URL)
> · `flow` — a specific flow (*"home → contact → submit the form"*)
> · `site` — crawl the whole site (default cap: 8 pages)
>
> I'll build three rater personas, run the expert evaluation, and give you a short action summary
> with a link to the full report in `user-simulation-tests/heuristic/results/`.

## Handoff mode (called from `/user-simulation`)

If the args start with `handoff`, the user already answered everything: take `url`, `scope`
(`screen` | `flow`), `task` (for a flow: the task in natural language — normalize it into the flow
script), `date`, `lang`, `deliver`. **Ask nothing.** The defaults that replace every question in
this skill:

| Question in direct use | Handoff default |
|---|---|
| Persona files exist — reuse or regenerate? | Reuse |
| Confirm the flow script | Don't ask — walk it |
| Trigger validation errors? | Yes (safety rails still apply) |
| Flow element not found | Re-`READ` once; still missing → end the capture there and note it in the report meta |
| Persona gates fail after 2 retries | Use the last attempt and note it in the Method note |

---

## Required inputs

1. **URL** — the live web app (already running / deployed).
2. **Mode** — `screen` | `flow` | `site`. If `flow`, also the flow description in natural language.

Optional: page cap override for site mode (3–12, default 8) · a short business description ·
credentials (site mode, to evaluate behind a login) · error-probing preference for flow/screen
modes (asked later if not given).

If URL or mode is missing, ask before continuing. Do not invent either.

---

## Step 0 — Preflight (HARD STOP — before ANYTHING else)

### 0a. Browser

Pick the driver per `browser.md` §1 (silently, one line). Never ask the user to install anything.

### 0b. Reachability

`OPEN(url)` and `READ()` once. If navigation fails (DNS, timeout, blank page): show the error and
**STOP** — a technical failure is never a finding. If the app is client-rendered and the driver is
read-only, show `browser.md` §4 and STOP. In read-only mode, flows that need forms are captured up to
the first form (note it in the report meta).

---

## Step 1 — Business research (skippable on persona reuse)

If `user-simulation-tests/heuristic/personas/<domain>-power.md`,
`user-simulation-tests/heuristic/personas/<domain>-average.md` and
`user-simulation-tests/heuristic/personas/<domain>-low-literacy.md` already exist (domain = host
without TLD), ask: **reuse or regenerate?** (handoff: reuse) On reuse, load the business summary from
`user-simulation-tests/heuristic/personas/<domain>.research.md` and skip to Step 3.

Otherwise: navigate the home page plus up to 2 more pages (pricing / about / a primary workflow
page), snapshot each, and extract a **business summary**: what the product is, who it serves, key
workflows, and what a failed conversion costs. Discard each snapshot immediately after extracting
its signal. Fold in the user-provided business description if any. Save the summary to
`user-simulation-tests/heuristic/personas/<domain>.research.md` (create the folder if needed).

## Step 2 — Personas (one call, fixed trio)

Spawn the `heuristic-persona-generator` subagent (Agent tool,
`subagent_type: "heuristic-persona-generator"`) **once** — a single call generates all 3 for better
cross-persona differentiation — passing as text: `DOMAIN: <host without TLD>` plus the business
summary. Parse the JSON reply, **enforce the slugs yourself** (they must be exactly
`<domain>-power`, `<domain>-average`, `<domain>-low-literacy` — override any deviation before
writing files), and **verify the gates yourself** (never trust selfCheck alone):

- exactly 3 personas with archetypes `power` · `average` · `low-literacy`;
- the three "Sensitive to" lists overlap by at most 1 item;
- calibration anchors are site-specific (name real finding types for THIS site);
- no navigation machinery (abandonment rules, behavior axes, emotional progression) in any profile.

On a failed gate: up to **2 retries** passing the concrete issues back. Then write the three
profiles to `user-simulation-tests/heuristic/personas/<slug>.md`. Never overwrite existing files without asking.

## Step 3 — Capture the scope

Every captured state becomes an entry in the **screen registry**:
`{ screenId, url, stateLabel, oneLiner, screenshotPath }`, plus a **trimmed snapshot** held in a
buffer. Also capture one screenshot per state via `SHOT(<screenId>)` and record the returned path
(no path when the driver can't save screenshots — heuristic 8 then follows its no-screenshot rule).

**Trimming rules** (apply to every snapshot before buffering, in this order):

1. Strip all `ref=` tokens (the expert never acts on elements — refs are dead weight).
2. Collapse repeated sibling list items beyond the first 5 into `(+N more similar items)`.
3. Hard cap 15,000 characters per snapshot; truncate with `[SNAPSHOT TRUNCATED]`.
4. Write a one-line `oneLiner` describing the screen ("Pricing page: three plan cards + comparison
   table") into the registry.

In **site mode**, extract in-scope links into the crawl queue BEFORE trimming.

### 3A. Screen mode

Navigate, snapshot, screenshot, trim, register. One screen. If the screen hosts a form, offer
error probing (same rules as 3B) — the error state registers as a second entry
(`screenId: <id>--error`).

### 3B. Flow mode

The user declared the flow in natural language. Normalize it into a **flow script** — an ordered
list of steps, each `{ goal, action }` where action is one of `navigate <url>` | `click "<label>"` |
`fill form and submit` — and **show it back for confirmation** before walking it.

Ask once at confirmation (if not already answered): *"Also trigger validation errors deliberately?
(I submit forms empty/invalid first to evaluate error handling — heuristic 9.)"* Recommended
default: yes.

Then walk the script deterministically: snapshot → locate the target element by its label in the
snapshot → `ACT` on it (click / type / key, using the element's `ref` from the current `READ`) →
`WAIT` for the expected signal (text appearing or disappearing — never a guessed sleep) → `READ` +
`SHOT` the resulting state → trim + register.

- If a declared element does not exist in the snapshot: **STOP and ask** — in an inspection this is
  a script error to fix, not friction to record (the inverse of the simulation rule). In handoff
  mode, use the handoff default instead.
- **Error probing** (when opted in): at each form, first submit empty/invalid and register the
  error state as an extra entry (`stateLabel: "error"`), then fill correctly and continue.
- **Safety rails:** never probe or complete forms with destructive, payment, or irreversible
  semantics; use obviously fake placeholder data; before any final real submission, capture the
  filled state and stop the flow there unless the user explicitly confirmed a test environment.

### 3C. Site mode (crawl)

You (the orchestrator) crawl BFS from the home page:

- **Page cap: 8** by default (user override 3–12).
- **Scope:** same origin only (scheme + host, `www.` normalized). Subdomains excluded.
- **Dedup:** keep a `visited` set of normalized URLs (strip fragments, trailing slashes, tracking
  params). **Template dedup:** never enqueue more than 2 URLs matching the same path pattern
  (`/product/*` → evaluate one representative, not the catalog).
- **Traversal:** max depth 2, prioritized for template diversity — main-nav links first, then
  footer/utility links, then one representative per repeated card/list pattern.
- **Login walls:** if a page redirects to auth or renders a login form, capture the login screen
  ONCE (it is itself evaluable), mark everything behind it as `notEvaluated`. If the user supplied
  credentials, perform the login as a scripted pre-step (its states also get captured) and continue
  crawling inside.
- **Termination:** cap reached or queue empty. If the cap was hit, set `capHit: true` and record
  the leftover queue in `notEvaluated` — never silently drop it.

## Step 4 — Expert evaluation

**Budget gate:** if screens ≤ 8 AND total trimmed snapshot size ≤ 100,000 characters → single call.
Otherwise → chunked fallback (below).

**Single call.** Spawn `heuristic-expert-evaluator` (Agent tool,
`subagent_type: "heuristic-expert-evaluator"`) passing as text:

```
BUSINESS SUMMARY: <text>
SCOPE: <site|flow|screen> — <N> screens
--- SCREEN <screenId> | <url> | state: <default|error|filled> | screenshot: <path> ---
<trimmed accessibility snapshot>
--- END SCREEN ---
(repeat per screen)
```

**Chunked fallback** (announce it in one progress line): split screens into chunks of 4 in capture
order; one expert call per chunk with `PASS: chunk N/M` (evaluates heuristics 1–3 and 5–10 on its
screens and returns a `screenDigest` per screen; **validate that chunk's evidence quotes against
its snapshots, then discard those snapshots**). Then one final call with `PASS: consistency`
passing ALL digests — it emits only heuristic-4 findings, whose evidence is verified against the
retained digests (note in the report meta that heuristic-4 evidence is digest-based). Merge: chunk
findings + consistency findings; merge coverage per heuristic (heuristic 4 comes from the
consistency pass; for 1–3 and 5–10: `violations` in any chunk → `violations`; else `clean` in at
least one chunk → `clean` (the channel was observable somewhere); else `not-observable`,
concatenating the notes).

**Validate each reply** (per the failure table below). The coverage ledger of a single reply must
have **exactly one entry per heuristic assigned to that pass** — 10 in a full pass, 9 in a
`chunk N/M` pass, 1 in a `consistency` pass; the **merged** ledger must have all 10. Then re-key
finding ids to `F01…Fnn` in capture order. **Then discard every remaining snapshot from working
context** — from this point you keep only the screen registry, the coverage ledger, and the
findings JSON.

## Step 5 — Persona rating (3 parallel calls)

Spawn **three** `heuristic-persona-rater` subagents **in the same message** (they are independent
and never touch the browser). Each receives, as text:

- the full content of ITS persona profile (`user-simulation-tests/heuristic/personas/<slug>.md`),
- the business summary,
- the screen context: one `screenId: <oneLiner>` line per screen referenced by any finding,
- the findings **projection**: for each finding, ONLY `{ id, title, heuristicName, screens,
  description, evidence quotes }`. Do NOT include `businessImpact`, `businessImpactReason`, or
  `suggestedFix` — the raters' judgment must not be anchored by the expert's.

Validate each reply: every findingId rated exactly once, `usabilityImpact ∈ {1,2,3}`,
`importantToMe` boolean, non-empty first-person reason.

## Step 6 — Deterministic computation (YOU compute — never an agent)

For each finding:

- `avgUsability` = mean of the available usabilityImpact ratings, **one decimal**;
- `priority` = `businessImpact × avgUsability`, one decimal (1.0–9.0);
- `convergence` = count of raters with `importantToMe: true`, displayed as N/3 (or N/2 if a rater
  was dropped).

Sort: priority desc → convergence desc → businessImpact desc → heuristic number asc. Build the
complete computed findings list **before** invoking the synthesizer.

## Step 7 — Synthesis

Spawn `heuristic-report-synthesizer` (Agent tool, `subagent_type: "heuristic-report-synthesizer"`)
passing as text the full pre-computed bundle:

- meta: `{ url, domain, date, scopeMode, screensEvaluated (registry without snapshots), pageCap,
  capHit, notEvaluated, raterCoverage, errorProbing }`;
- the business summary;
- the personas: `{ slug, archetype, one-line role }` each;
- the coverage ledger (10 entries);
- the findings, complete and **pre-sorted**, each with its ratings and computed numbers.

Its instructions require it to copy every number verbatim. Save the returned Markdown to
`user-simulation-tests/heuristic/results/<YYYY-MM-DD>-heuristic-<domain>-<scope>.md` where
`<scope>` = `site` | `flow-<slug>` | `screen-<slug>` (create the folder if needed).

## Step 8 — Deliver

- **`deliver=none`** (handoff inside a combined run): print one line —
  `✔ Heuristic review done — M findings` — and return the report path to the caller. Stop here.
- Otherwise spawn the `summary-synthesizer` subagent (Agent tool,
  `subagent_type: "summary-synthesizer"`) passing as text: `LANGUAGE` (handoff `lang`, else the
  conversation's language), `META` (url, domain, date, test=heuristic, scope + task, driver),
  `USERS` (none), `REPORTS` (`heuristicPath`), `LINK_BASE: user-simulation-tests/` — paths relative
  to it. Save the reply to `user-simulation-tests/<YYYY-MM-DD>-<domain>-heuristic-summary.md` and
  show exactly:

```
✅ Done — <domain>

<the summary, verbatim>

📄 Summary saved: user-simulation-tests/<filename>
```

Do **NOT** paste the full report or its findings table in chat — the summary's mini tables are the
quick read and they link to the full `.md`. Then drop everything except the paths from working
context.

---

## Failure policies

| Failure | Policy |
|---|---|
| No browser can see the app | `browser.md` §4 message + STOP |
| Site unreachable | Step 0b: show error, STOP; never a finding |
| Expert returns malformed JSON | 1 retry appending the parse error + "return ONLY valid JSON". 2nd failure → STOP with a technical-failure message |
| Expert entries invalid (heuristic ∉ 1–10, businessImpact ∉ {1,2,3}, unknown screenId, empty evidence) | 1 corrective retry naming the offending ids; still invalid → drop those findings, note it in the report meta |
| Expert returns 0 findings | Accept only if the reply's coverage ledger is complete for its pass (clean / not-observable entries); otherwise 1 retry demanding the enumeration |
| Coverage ledger incomplete for its pass (10 full / 9 chunk / 1 consistency) | 1 corrective retry; the ledger is the enumeration proof — non-negotiable |
| A rater malformed/incomplete | 1 retry with the parse/coverage error; still failing → drop that rater wholesale: average over 2, convergence N/2, `raterCoverage: "2/3"` + note in the Method note. If 2 of 3 raters fail → STOP (a single rating is not an average) |
| Persona gates fail after 2 retries | Show the profiles with a warning and ask: use as-is / regenerate |
| Page cap hit mid-crawl | Normal termination: `capHit: true`, leftover queue → `notEvaluated`, surfaced in the report |
| Flow element not found | STOP and ask (script error, not friction) |
| Expert bundle over budget | Automatic chunked fallback, announced in one progress line |
| Persona files exist | Ask reuse vs regenerate; never overwrite silently |

## Orchestrator rules

- **You never detect and never score.** Detection is the expert's; usability judgment is the
  raters'; you compute arithmetic and enforce contracts.
- **Anti-anchoring is structural.** Raters never see the expert's businessImpact or suggestedFix;
  the two axes meet for the first time in YOUR Step 6 computation.
- **Evidence or it didn't happen.** Reject any expert finding whose evidence quote you cannot find
  in the corresponding trimmed snapshot (allow whitespace differences). In chunked mode, verify
  each chunk's findings against its snapshots BEFORE discarding them; consistency-pass (heuristic
  4) findings verify against the retained digests instead.
- **"Clean" ≠ "not observable".** Preserve the distinction through every hop — it is the
  framework's honesty mechanism.
- **Context hygiene:** research snapshots die at Step 1; scope snapshots die at Step 4 (after the
  expert call); rater raw replies die at Step 6; after Step 8 only the report paths survive.
- **Never a silent truncation.** Cap hits, login walls, dropped raters, dropped findings — all of
  it surfaces in the report meta.
- Reports and personas are always written in **English**, regardless of the conversation language.
  The action summary is written in the user's language.
