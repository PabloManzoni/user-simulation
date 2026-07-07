# Heuristic Review Framework

## Conceptual framework and operational protocol for AI-assisted heuristic evaluation with synthetic severity raters

## 1. The Methodological Problem

Heuristic evaluation is the classic usability inspection method: trained evaluators judge an interface against a small set of recognized principles and produce a list of violations (Nielsen & Molich, 1990). It is cheap, fast, and systematic — and it has always depended on having three to five trained evaluators available.

A language model can inspect an interface. But an unconstrained LLM review reproduces, for inspection, the exact failure the Synthetic Users Framework identified for simulation: the model reports what is *salient*, not what is *systematic*. Asked to "review this UI", it finds the three loudest problems, praises the rest, and calls it a review.

The failure is not laziness. It is the absence of protocol.

<aside>
🎯

The issue is not the absence of expert review. It is the absence of **controlled inspection**.

This document defines a framework for running heuristic evaluations with AI agents under explicit protocol: one expert agent detects violations under **forced enumeration** of the ten heuristics, and three **synthetic rater personas** independently judge the impact of every finding.

</aside>

## 2. Two Lenses: Inspection and Experience

This framework is the sibling of the Synthetic Users Framework (the `user-simulation` plugin). They are complementary instruments, not competitors.

| | Heuristic review (this framework) | User simulation |
|---|---|---|
| Lens | Inspection against principles | Lived experience of one profile |
| Who judges | An expert agent + rater personas | The synthetic user itself |
| Uses external knowledge | Required (the heuristics ARE external knowledge) | Forbidden |
| Finds | Violations of known usability principles | Friction a specific profile actually hits |
| Output | Prioritized findings table | Step-by-step narrative + emotional arc |
| Blind spot | Task-level failures, emotional accumulation | Problems off the simulated path |

Simulation tells you where one user bleeds. Inspection tells you where the interface is sharp. Run both.

## 3. Formal Definition

A **controlled heuristic review** is an inspection protocol in which one expert agent detects violations under forced enumeration of the ten usability heuristics, and three synthetic rater personas independently judge the usability impact of each finding from defined perspectives, producing a priority ranking on two explicit axes.

Each component is essential:

- **Forced enumeration.** The expert sweeps every screen against heuristic 1, then heuristic 2, and so on through 10 — and must record an explicit result for every heuristic, including "no violations found".
- **Verbatim evidence.** Every finding quotes the exact interface text or element it is about. A finding that cannot be quoted does not exist.
- **Fixed rater archetypes.** Always the same three: a power user, an average user, and a low-digital-literacy user — contextualized to the evaluated business, never replaced.
- **Two-axis scoring.** Business impact (judged by the expert, from a business summary) and usability impact (judged by each rater persona, from their own seat).
- **Deterministic priority.** Priority is computed, not opined: business impact × average usability impact, ties broken by rater convergence.

## 4. Why AI Changes the Method

The classic protocol was designed for humans. Three of its pillars transform when the evaluator is a model.

### No second pass

Nielsen's protocol has each evaluator traverse the interface at least twice: the first pass exists so a *human* can learn the flow before judging elements in context. An LLM receives the entire captured scope at once — the learning pass buys nothing. What must be preserved from the original protocol is not the two passes but the **element-by-heuristic systematicity**, which is exactly what an LLM loses by default.

### Salience bias → forced enumeration

A model asked to "review this UI" answers from salience. The countermeasure is structural, not exhortative: the protocol forces the expert to sweep every screen against each heuristic in order, and to record an explicit per-heuristic result. Coverage is enforced by the protocol, not requested of the model.

The findings the model would have produced anyway are not the point. The point is the sweeps it would have skipped.

### Why 1 expert + 3 raters replaces 3–5 evaluators

Nielsen aggregated multiple evaluators for two distinct reasons:

1. **Coverage** — no single evaluator finds enough problems (individual evaluators found only about 35% of known problems in Nielsen & Molich's data).
2. **Severity judgment** — severity estimates from a single evaluator are unreliable, so Nielsen recommends collecting severity ratings from several evaluators *after* detection.

In this framework, coverage is addressed by forced enumeration — and honestly bounded (see §8). Severity judgment is recovered by the three rater personas: they restore the multi-perspective severity panel without re-running detection. The personas are not decoration; **they are the replacement for Nielsen's severity-rating panel.**

### Detection and rating never mix

A rater who detects reintroduces salience bias through the back door. An expert who rates "for the user" collapses three perspectives into one guess. Strict role separation is the AI analogue of Nielsen's rule that evaluators inspect alone and severity is collected afterward.

<aside>
💡

Explicit role separation is the core of the method.

</aside>

## 5. The Scoring Model

Every finding is scored on two axes and one convergence signal.

### Business impact (1–3) — judged by the expert, from the business summary

- **3 — Blocks or damages a core business outcome.** The violation sits on a path the business lives on — signup, purchase, booking, contact, quote request — and can plausibly cause that outcome to fail, or exposes the business to loss of trust or money.
- **2 — Degrades a supporting outcome.** Adds real friction to a path adjacent to revenue or retention: longer funnels, avoidable support contacts, partial task failure with a workaround.
- **1 — Peripheral.** Annoying, but does not plausibly move any business outcome named in the business summary.

### Usability impact (1–3) — judged by each rater persona, in first person

- **3 — Severe.** Facing this, I would likely fail the task, abandon it, or lose enough trust to leave.
- **2 — Moderate.** It would slow me down, create doubt, or force a workaround — but I would recover and finish.
- **1 — Minor / cosmetic.** I might notice it; it would not meaningfully slow me down.

### Relation to Nielsen's severity framework

Nielsen rates severity as a combination of **frequency**, **impact**, and **persistence** on a single 0–4 scale. This framework adapts it in two moves:

1. Frequency, impact and persistence are collapsed into the 1–3 usability axis and judged **per persona** — because frequency and persistence are properties of a *usage pattern*, and the three personas have different ones. A weekly power user and a first-time visitor do not hit the same problem at the same frequency.
2. A second axis — business impact — is added, because a severity number alone cannot tell a founder what to fix first.

### Priority and convergence

```
priority    = businessImpact × avg(usabilityImpact)     → 1.0–9.0, one decimal
convergence = how many of the 3 raters marked the finding "important to me" → N/3
```

Findings are ranked by priority descending; ties broken by convergence, then business impact, then heuristic number. The computation is deterministic and performed by the orchestrator — never improvised by an agent.

**Divergence is not noise.** A finding rated 3 by the low-digital-literacy rater and 1 by the power user tells you precisely who the interface is silently built for. The report surfaces every such split.

## 6. Roles and Boundaries

- **The expert** detects violations and scores business impact. It inspects; it does not simulate, does not rate usability severity, and does not praise.
- **The rater personas** judge usability impact and declare importance, each strictly from their own seat. They are jurors, not inspectors: they never detect, never navigate, never propose fixes.
- **The persona generator** contextualizes the three *fixed* archetypes to the evaluated business. The trio never changes; only who they concretely are relative to this site does.
- **The synthesizer** computes nothing and re-judges nothing — it receives pre-computed numbers and presents them faithfully.

## 7. Three Implementation Modes

### A. Direct LLM

Paste screenshots or copied interface content into a chat along with the ten-heuristics block and the scoring rubrics from this document.

**Advantages:** fast, zero setup, good for a first look.
**Limitations:** no forced-enumeration guarantee, no rater panel, not reproducible.

### B. Interactive live review

An MCP browser captures the scope; you run the expert prompt over it screen by screen, iterating as the design changes.

**Advantages:** structured, repeatable per screen, good for design iteration.
**Limitations:** manual orchestration; the rater panel and priority computation are on you.

### C. Automated orchestration (this plugin)

The full protocol: research → persona generation → scope capture → forced-enumeration detection → independent rating → deterministic priority → report.

**Advantages:** reproducible, complete protocol, comparable across runs and versions.
**Limitations:** requires the Playwright MCP setup; bounded by what the captured scope contains.

The report's Method note describes the protocol that ran (mode C when produced by this plugin).

## 8. Limitations

- **The single-evaluator ceiling, adapted.** Nielsen & Molich found that individual evaluators detect only about 35% of usability problems; aggregating 3–5 evaluators is what pushes coverage up. Forced enumeration raises an LLM's systematicity, but this framework still runs **one** detector: treat the findings list as a **lower bound**, never as proof of absence. "No violations found" means "none observable in this scope by this protocol".
- **Snapshot blindness.** Accessibility snapshots carry text, roles and states — not whitespace, visual hierarchy, color or motion. Some heuristics (notably aesthetic and minimalist design) are only partially observable; the protocol requires the expert to declare confidence and to record "not observable in scope" rather than guess. *Clean* and *not observable* are different verdicts, and the report keeps them distinct.
- **Rated judgment, not empirical severity.** Persona ratings are calibrated perspectives, not user data. They order the backlog; they do not validate it.
- **Inspection finds violations of principles, not task failures.** A perfectly heuristic-clean interface can still fail its users. Pair this instrument with user simulation, and eventually with real research.
- **False positives are cheap by design.** Every finding carries a verbatim evidence quote, so a human can verify or discard it in seconds.

## 9. References

Foundational:

- Nielsen, J., & Molich, R. (1990). *Heuristic Evaluation of User Interfaces.* Proceedings of ACM CHI'90 (Seattle, WA), 249–256.
- Nielsen, J. (1994). *Enhancing the Explanatory Power of Usability Heuristics.* Proceedings of ACM CHI'94 (Boston, MA), 152–158.
- Nielsen, J., & Mack, R. L. (Eds.) (1994). *Usability Inspection Methods.* John Wiley & Sons, New York.

Operative (embedded in the expert agent's protocol):

- Nielsen, J. *10 Usability Heuristics for User Interface Design.* Nielsen Norman Group, nngroup.com/articles/ten-usability-heuristics/
- Nielsen, J. *How to Conduct a Heuristic Evaluation.* Nielsen Norman Group, nngroup.com/articles/how-to-conduct-a-heuristic-evaluation/
- Nielsen, J. *Severity Ratings for Usability Problems.* Nielsen Norman Group, nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/

## Closing

The power of a heuristic review does not lie in the model's taste. It lies in forcing the model to look where it was not going to look.

The stricter the enumeration, the stronger the finding.

<aside>
📌

Prepared by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/)

Product Designer at [Kaizen Softworks](https://www.kzsoftworks.com/)

Last updated: July 7th, 2026

</aside>
