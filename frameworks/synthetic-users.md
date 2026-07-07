# Synthetic Users Framework

## Conceptual framework and operational protocol for controlled synthetic user simulation

## 1. The Methodological Problem

UX validation traditionally depends on real users. Interviews, usability testing, and quantitative data allow us to observe authentic behaviors and make informed decisions. However, in early design stages or rapid iteration cycles, it is not always feasible to rely immediately on formal research.

In this context, a structural tension emerges. We need to stress-test design decisions before investing in formal validation, yet we lack an intermediate instrument that allows us to do so in a rigorous and reproducible way.

Traditional personas describe user profiles, but they do not execute decisions. Language models can simulate interaction, but they tend to optimize the flow, fill design gaps, or rely on undeclared knowledge.

The issue is not the absence of simulation. It is the absence of controlled simulation.

<aside>
🎯

This document defines a **framework for building and using synthetic users as decision agents** operating under explicit rules and clearly defined constraints, with the goal of revealing structural friction before exposing the system to real users.

</aside>

## 2. From Description to Execution

A UX persona is a descriptive artifact. It summarizes context, motivations, and characteristics of a typical user. Its purpose is to guide design discussions.

A synthetic user does not describe. It executes.

The difference is methodological. A descriptive artifact does not make decisions within a workflow. An executable agent does.

A **synthetic user does not attempt to predict human behavior** or represent a population. It does not generate empirical evidence. Its function is to simulate decision-making under controlled conditions in order to expose how a system behaves when interpreted by an agent with defined limits.

It does not simulate humanity. It simulates interpretation and cognitive friction.

## 3. Formal Definition

A **synthetic user is an artificial decision agent** constructed through an explicit objective, behavioral rules, defined constraints, and forbidden assumptions, designed to interact with a digital system under controlled conditions and without external compensation.

Each component is essential:

- It has a concrete objective.
- It operates within defined limits.
- It follows explicit behavioral rules.
- It cannot use knowledge external to the defined scenario.
- It does not optimize the flow unless its profile explicitly allows it.
- It does not compensate for design deficiencies through additional reasoning.

The quality of a synthetic user depends more on its constraints than on its profile. Without explicit limits, the model optimizes automatically and the simulation loses value.

## 4. Conceptual Architecture of a Synthetic User

Every synthetic user requires four structural layers.

### The first layer is the functional profile:

It defines the agent’s role, its level of domain expertise, its familiarity with digital tools, and its tolerance for friction. Demographic attributes are unnecessary unless directly relevant. What matters is the agent’s relationship to the task.

### The second layer is the explicit objective:

It must be clearly defined. Without an objective, there is no valid simulation. The agent must know what it needs to achieve and what level of certainty is considered sufficient.

### The third layer is the behavioral and emotional layer:

This layer establishes how the agent interprets information, what it prioritizes, what it avoids, how it reacts to ambiguity, and under what conditions it abandons the task.

It also defines the emotional state of the agent during the interaction. The synthetic user must register not only what it does, but also what it feels or internally experiences while using the system. This includes confusion, frustration, relief, boredom, hesitation, confidence, distrust, or fatigue.

The emotional layer does not attempt to simulate deep human psychology. Its role is to expose how cumulative friction may affect the user’s internal experience and final perception of the flow.

### The fourth layer is the constraints and forbidden assumptions layer:

It specifies what knowledge the agent does not possess, what it cannot assume, and which inferences are outside its scope. This section prevents the model from compensating for system gaps.

## 5. Principles of Controlled Simulation

The simulation must be sequential and contextual. The agent may only make decisions based on:

- The information visible in the current step.
- What is explicitly defined in its profile.
- What has been experienced within the evaluated flow.

Each step experienced by the synthetic user must be stored as accumulated interaction memory.

The agent may not inspect future screens, sections, windows, or steps before reaching them. It can only use information from the current step, its defined profile, and the memory accumulated from previous steps.

This accumulated memory must influence later interpretation. For example, repeated ambiguity, excessive effort, unclear feedback, or unnecessary repetition may increase frustration, fatigue, distrust, or impatience over time.

It cannot use external knowledge, backend logic, or undeclared organizational context.

The central principle is to prevent artificial cognitive compensation. If the system does not communicate something, the synthetic user cannot assume it. If an element is blocked without explanation, the agent must experience it as friction.

<aside>
💡

Explicit constraint is the core of the method.

</aside>

## 6. From Step by Step to Structural Insight

The interaction narrative is not the final output. **It is input.**

Value emerges when multiple simulations are compared and consistent patterns appear. This process makes it possible to distinguish between:

- Structural friction that affects different profiles
- Friction that depends on expertise level
- Friction driven by expectations of efficiency or the need for confirmation

The insight does not come from the model’s opinion, but from the consistent repetition of behaviors under defined rules.

## 7. Calibration With Real Behavior

Synthetic users can be calibrated using real human behavior signals, such as analytics, logs, heatmaps, anonymized recordings, or data from tools like MS Clarity or Pendo.

These inputs do not turn a synthetic user into a predictive model. Their role is to adjust behavioral rules so they reflect observable patterns, without abandoning the controlled nature of the method.

## 8. Practical Usage Protocol

Operationally, the framework follows a clear process:

1. First, define the synthetic user, including profile, objective, behavioral rules, and forbidden assumptions.
2. Then, prepare the flow to be evaluated, which can be provided as structured copy, screenshots, prototypes, or sequential descriptions.
3. Run the simulation step by step. The agent makes decisions aligned with its profile and records friction, doubt, and potential abandonment.
4. Finally, aggregate multiple simulations and extract patterns. The method does not end with a single narrative, but with comparative synthesis.

## 9. Minimal Operational Specification

Every simulation must produce a structured output that includes:

- Step by step evaluation with action taken, clarity level, doubt detection, emotional state, and accumulated memory
- Point of highest friction
- Point of highest emotional friction
- Final agent decision
- Final emotional summary
- Detected risks
- Derived structural insight
- Overall study analysis

<aside>
💡

The consistency of the format allows executions to be compared across versions and across profiles. Synthetic users do not replace research with real people. They do not capture deep emotional nuance or complex organizational context.

</aside>

## 10. Practical Implementation

From this point forward, the focus is no longer conceptual.

It is operational.

To use synthetic users effectively, you need four concrete components:

1. A clearly defined synthetic user
2. A clearly bounded flow to evaluate
3. An execution prompt
4. A structured output format

Without these four elements, the method does not work.

### 10.1 Mandatory Minimal Structure

Before running any simulation, the following structures must exist.

### A. Synthetic User

At minimum, it must include:

```jsx
Functional Role:
Domain Expertise Level:
Technical Proficiency Level:
Primary Motivation:
Explicit Objective:
Success Criteria:
Constraints:
Behavioral Rules:
Abandonment Rules:
Forbidden Assumptions:
Evaluated Flow:
```

If any of these are missing, the model will optimize or compensate.

### B. Flow to Evaluate

The flow must be clearly bounded.

It may be presented as:

- Ordered screenshots
- Structured copy
- Figma prototype
- MCP + Figma
- Sequential textual screens

It must be clear:

- Where the flow starts
- Where it ends
- Which part of the system is being evaluated

### C. Mandatory Output Format

Without a fixed format, there is no comparability.

Minimal example format:

```jsx
Step 1
Action taken:
Clarity level: High / Medium / Low
Doubt detected: Yes / No
Emotional state:
Accumulated memory:
Reason:

Step 2
Action taken:
Clarity level: High / Medium / Low
Doubt detected: Yes / No
Emotional state:
Accumulated memory:
Reason:

Point of Highest Friction:
Point of Highest Emotional Friction:
Final Decision: Continues / Abandons
Final Emotional Summary:
Describe what the synthetic user internally experienced across the whole flow. This must be written as a simulated internal reflection, not as an analytical UX summary.

Example:
“I completed the form, but it felt unnecessarily long. At first I was fine, then I started getting annoyed because I had to keep making decisions without being fully sure. I finished it, but I would not want to do this again unless I really had to.”

Detected Risks:
Structural Insight:
Overall Study Analysis:
Synthesize the full evaluation across all steps. Include functional friction, emotional progression, accumulated frustration or confidence, abandonment risk, and the likely perception of the experience after completion.
```

### 10.2 Three Implementation Modes

The method can be implemented in three different ways.

### A. Direct LLM

A fast, manual approach.

You manually provide:

- The synthetic user
- The flow
- The instructions

The result is generated directly in chat.

**Advantages:**

- Fast
- Flexible
- Ideal for early exploration

**Limitations:**

- Not automatically reproducible
- Does not guarantee consistency across iterations
- Not scalable

### B. MCP + Figma

An interactive, live design mode.

Configured once with:

- MCP connection to Figma
- The synthetic user
- A structured prompt

The agent:

- Reads the design directly
- Executes the simulation
- Responds to changes in real time
- Allows rapid instruction iteration

**Advantages:**

- More structured
- More consistent
- Ideal for iterative design work
- Supports reusable, stable profiles

**Limitations:**

- Not yet large-scale automation
- Does not automatically generate historical records unless configured

This is the recommended mode for serious iterative work.

### C. Automated Orchestration

A system-level implementation.

An agent is configured to:

- Retrieve the design automatically
- Run multiple synthetic users
- Normalize outputs
- Store structured results
- Enable comparison across versions

**Advantages:**

- Scalable
- Reproducible
- Supports longitudinal analysis

**Limitations:**

- Requires technical architecture
- Less exploratory

### 10.3 Base Simulation Prompt

Standard execution prompt:

```jsx
Act strictly as the synthetic user defined below.
You must follow its behavioral rules, constraints, and forbidden assumptions.
You may not add external knowledge.
You may not compensate for system gaps.
You may not optimize the flow unless the profile explicitly allows it.
Evaluate the flow in the order presented.
Produce the result exclusively in the specified structured format.
```

### 10.7 How to Validate Execution Quality

A simulation is considered valid when:

- Decisions are consistent with the defined profile
- No external knowledge appears
- The design is not mentally “fixed”
- Real hesitation is detected
- The agent can abandon when appropriate
- Emotional reactions align with the profile
- Emotional state evolves through accumulated memory
- Frustration, fatigue, trust, or confidence persist across steps
- The agent does not use future information
- The final emotional summary reflects the full experience
- If the synthetic user always completes the flow without friction, it is poorly calibrated.

## 11. Profile Contrast and Reusable Library

The method becomes significantly stronger when contrasting different profiles over the same flow.

Comparing a senior profile with an intermediate one allows you to distinguish structural friction from expertise-dependent friction. If both detect the same issue, the design likely has a structural weakness. If only one does, the friction may be contextual.

Synthetic users can be stored as reusable assets. Once properly defined, they can be reactivated in different evaluations. This enables comparison across system versions over time while maintaining methodological consistency.

Building a deliberate library of well-defined synthetic users allows you to “call” a specific profile when needed, ensuring evaluation stability and avoiding constant redefinition.

Intentional profile diversity expands your ability to stress-test the system without abandoning the controlled framework.

## Closing

The synthetic user is a methodological tool designed to expose structural friction before formal validation.

Its power does not lie in simulating real people. It lies in applying explicit constraints that prevent the model from compensating for system deficiencies.

The clearer the rules, the stronger the insight.

<aside>
📌

Prepared by [Pablo Manzoni](https://www.linkedin.com/in/pablomanzoni/)

Product Designer at [Kaizen Softworks](https://www.kzsoftworks.com/)

Last updated: February 20th, 2026

</aside>