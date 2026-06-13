# Create PRD: Intent-Engineered Product Requirements Document

## Overview

Generate a PRD that captures **intent**, not a spec. Intent engineering means
defining the **WHY** for the builder: a concrete problem, a **falsifiable
hypothesis** (with a right AND a wrong condition), real success criteria, and how
the work is verified. Nail that and even a smaller model can carry the build,
because everything it needs to decide is already decided.

> **The PRD says why and what. The spec/PRP says how.** Data models, API shapes,
> schemas, libraries, file layouts — those are engineering decisions and do NOT
> belong in the PRD. A PRD full of them is the anti-pattern.

Gold-standard worked example to model: `prd-exemplar-slack-threads.md`
(co-located in this references folder — read it before writing).

## Output File

Write the PRD to: `$ARGUMENTS` (default: `PRD.md`).

## Discovery Phase (before writing — gather the WHY first)

If the user hasn't provided enough, ask only what's load-bearing:

1. **What's broken, for whom, with what numbers?** The observable problem and
   the cohort (not "users want X" — the measured pain).
2. **Why now?** The trigger that makes it urgent.
3. **The hypothesis + the wrong condition.** The change, the expected behavior
   shift, the outcome — AND what result would prove us wrong. If there's no wrong
   condition, there's no PRD yet.
4. **Success metrics.** What moves, by how much, for which cohort, in what
   window? What guardrail must NOT drop?
5. **Non-goals.** What are we deliberately NOT doing in v1?
6. **The PRD/spec line.** Which engineering decisions are we leaving to the spec?
7. **Model / context window** (Claude Opus/Sonnet ~200K/1M, Codex, Gemini,
   local). Record it — CLUTCH uses it for phase sizing and sub-agent prompts.

Do not invent load-bearing answers. Ask one tight question instead.

## PRD Structure (write these sections, in order)

Keep every section concrete; cut anything generic. The first ten sections are the
intent. The Implementation Phases section is the buildable breakdown CLUTCH
consumes — it stays at the outcome/definition-of-done level, never engineering HOW.

**1. Problem statement** — concrete, with observable behaviors and real numbers;
name the cohort and the measured pain.

**2. Why now** — the triggers that make it urgent (trend, customer signal,
competitive pressure, internal signal).

**3. Hypothesis** — the falsifiable heart, in this exact shape:
> We believe **[change]** will cause **[cohort]** to **[behavior change]**,
> resulting in **[measurable outcome]**.
> We'll know we're **right** if **[primary metric moves by X]** within **[window]**.
> We'll know we're **wrong** if **[guardrail drops]** or **[counter-signal exceeds a threshold]**.

**4. Target user** — Primary / Secondary / **Not the target** (exclusions matter).

**5. Non-goals** — explicitly out of scope for v1, each with a one-line reason.

**6. Risks and assumptions** — table: `# | Risk | Assumption it depends on | How we'll de-risk`.

**7. Open questions** — flagged here, resolved during build (not answered here).

**8. Success metrics** — table: `Metric | Target | Cohort | Window`. Include a
**guardrail** (must-not-drop) and a **leading indicator**.

**9. Experiments & discovery plan** — spikes / reverse-experiments to resolve
before full build commit (strongest signal: turn it off and measure complaints).

**10. What is deliberately NOT in this PRD** — the engineering decisions that
belong in the spec (data model, API design, schema, routing rules, libraries).
End with: "The PRD says *[what won't happen]*. The spec says *how*."

**11. Implementation Phases** — REQUIRED for CLUTCH. Break the work into 2-4
phases. **Emit each phase as a literal `## Phase N: <Title>` header** (CLUTCH
auto-detects phases by scanning for these headers and generates a PRP per phase).
Each phase block must contain:

```markdown
## Phase 1: <Short title>
- **Goal:** <the outcome this phase delivers — what, not how>
- **Deliverables:** <the concrete artifacts/surfaces produced>
- **Validation:** <how we know this phase is done — its definition of done and
  the check that proves it: tests pass, the metric moves, the surface renders>
```

Keep phases at the outcome + definition-of-done level. The engineering HOW for
each phase lives in that phase's generated PRP/spec, not here.

## Quality gate (do not ship the PRD until all true)

- [ ] **Falsifiable hypothesis with BOTH a right and a wrong condition.**
- [ ] Problem statement is concrete with real numbers and a named cohort.
- [ ] **Non-goals** listed (each with a reason).
- [ ] Success metrics have **cohort + window** and include a **guardrail**.
- [ ] **Zero engineering HOW** in sections 1-10 (data models / API shapes /
      schemas / libraries go to the spec).
- [ ] Implementation Phases present, each as `## Phase N: <title>` with Goal +
      Deliverables + Validation (so CLUTCH phase auto-detect + PRP generation work).
- [ ] Scoped to one bet — not boiling the ocean.

## Anti-patterns — do NOT do this

- The old kitchen-sink template (Executive Summary, Mission, Tech Stack, API
  Specification, Security & Configuration, Core Architecture…). It pushes HOW
  into the PRD and reads like a spec.
- A "goal" with no measurable outcome and no way to be wrong.
- No non-goals → guaranteed scope creep.
- Success metrics with no cohort or window ("improve engagement").
- Engineering decisions (data models, endpoints, libraries) in sections 1-10.
- Overplanning — deciding what the spec should own.

## Output Confirmation

After writing the PRD:
1. Confirm the file path.
2. Summarize the hypothesis and the phase breakdown.
3. Note any assumptions made for missing info.
4. Next: CLUTCH detects the `## Phase N:` headers and generates a PRP (the spec /
   HOW) per phase.
