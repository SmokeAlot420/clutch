<p align="center">
  <h1 align="center"><b>CLUTCH</b></h1>
  <p align="center"><i>Claude Layered Unified Team Coordination Hub</i></p>
  <p align="center">Resilient multi-phase orchestration for Claude Code. Fail-fast guards, context budgeting, structured checkpoints, and workstream completion enforcement.</p>
</p>

<p align="center">
  <a href="https://github.com/SmokeAlot420/clutch/stargazers"><img src="https://img.shields.io/github/stars/SmokeAlot420/clutch?style=flat-square" alt="GitHub Stars"></a>
  <a href="https://github.com/SmokeAlot420/clutch/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/badge/Claude_Code-plugin-orange?style=flat-square" alt="Claude Code Plugin">
  <img src="https://img.shields.io/badge/Agent_Teams-parallel-green?style=flat-square" alt="Agent Teams">
</p>

<p align="center">
  <code>claude --plugin-dir ./clutch</code>
</p>

---

## Why I Built This

I was using [Ralph PIV](https://github.com/SmokeAlot420/ralph-piv) (Plan-Implement-Validate) for everything — and it's great for solo execution. Deep analysis, context-rich plans, independent validation. But some phases have work that's clearly parallelizable: frontend + backend + database, or three independent API endpoints, or UI components that don't touch the same files.

Running those serially felt wrong. If the work is file-independent, why not run 2-4 executors at the same time?

That's CLUTCH. Same Ralph PIV quality pipeline, but the execution phase runs in parallel using Claude Code's Agent Teams. The PRP defines workstreams (file-exclusive work units), and CLUTCH spawns one executor per workstream. They work simultaneously, then validation catches anything that slipped through.

## Who This Is For

- Devs already using [`/piv`](https://github.com/SmokeAlot420/ralph-piv) via Ralph PIV who hit phases with parallelizable work
- Anyone building features that span frontend + backend + DB
- Teams that want faster execution without sacrificing validation

## Quick Start

```bash
# Clone
git clone https://github.com/SmokeAlot420/clutch.git

# Use as Claude Code plugin
claude --plugin-dir ./clutch

# Run on a project with a PRD
/clutch ~/my-project/PRDs/PRD.md 1

# Auto-discover PRD
/clutch ~/my-project

# Run specific phases
/clutch ~/my-project 2 3
```

## How It Works (v2)

```
┌──────────────────────────────────────────────────────────────┐
│                  CLUTCH v2 ORCHESTRATOR                        │
├──────────────────────────────────────────────────────────────┤
│  1. Generate PRP with ## Workstreams section                  │
│  2. BUDGET GATE: enough context? NO → checkpoint + stop       │
│  3. DECISION MODEL: team / sequential / solo                  │
│  4. EXECUTE with FAIL-FAST (team: check git diff @ 2 turns)  │
│  5. Validate → Debug (max 3x) → Verify workstreams           │
│  6. CHECKPOINT to WORKFLOW.md → commit → next phase           │
└──────────────────────────────────────────────────────────────┘
```

1. **Budget Gate** — Estimate phase cost before starting; checkpoint + stop if context is insufficient
2. **Decision Model** — 6-condition matrix chooses team, sequential, or solo execution
3. **Fail-Fast Guard** — After 2 turns, check `git diff`; kill team + go sequential if no output
4. **Contract-First** — Dependent workstreams publish interface contracts before coding
5. **Workstream Enforcement** — Verify EVERY workstream has output before committing
6. **Structured Checkpoints** — WORKFLOW.md enables cold-start resumption from any session

## Why It Works

**Workstream file ownership.** The PRP defines which files each workstream owns. No two executors touch the same file. This is the primary coordination mechanism — it eliminates merge conflicts and stepping-on-toes failures.

**Contract-first protocol.** When workstreams have dependencies (frontend needs the API contract), upstream executors publish their interface contracts *before* implementing. The lead verifies them, then forwards to downstream. No more "3 agents built 3 things that don't connect."

**Fresh context per agent.** Each executor spawns with clean context, reads only their workstream scope + the full PRP. No context drift from long sessions.

## When to Use

| Scenario | Use |
|----------|-----|
| Single-focus phase (one component, one feature) | Solo [`/piv`](https://github.com/SmokeAlot420/ralph-piv) via Ralph PIV |
| Phase with parallelizable work (frontend + backend + DB) | `/clutch` |
| Quick single feature, no PRD | [`/mini-piv`](https://github.com/SmokeAlot420/ralph-piv) |
| Unsure | Use `/clutch` — it auto-falls back to solo if only 1 workstream |

## Features

- **Fail-fast execution guard** — detect non-productive teams in 2 turns, kill and go sequential
- **Context budget gate** — estimate phase cost, checkpoint + stop if context is insufficient
- **Structured checkpoints** — WORKFLOW.md enables cold-start resumption from any session
- **Workstream completion enforcement** — verify every workstream has output before commit
- **Decision model** — 6-condition matrix for team vs. sequential vs. solo
- **Parallel execution** — 2-4 executor teammates working simultaneously (when appropriate)
- **Contract-first protocol** — dependent workstreams publish interfaces before implementing
- **Independent validation** — validator doesn't trust executors, verifies everything
- **Debug via messaging** — gaps assigned back to responsible executors
- **Smart fallback** — 1 workstream → solo, team failure → sequential

## Plugin Structure

```
clutch/
├── .claude-plugin/plugin.json
├── agents/
│   ├── piv-executor.md
│   ├── piv-validator.md
│   └── piv-debugger.md
└── skills/
    └── clutch/
        ├── SKILL.md
        ├── references/
        │   ├── team-orchestration.md
        │   ├── generate-prp.md
        │   ├── execute-prp.md
        │   ├── codebase-analysis.md
        │   ├── create-prd.md
        │   └── piv-discovery.md
        └── assets/
            ├── prp_base.md
            └── workflow-template.md
```

## v2: Resilient Multi-Phase Orchestration

CLUTCH v2 addresses five structural gaps found during real multi-phase PRD execution (a team failure that burned ~40% of context with zero output, no context budgeting, no cross-session checkpointing, a phase committed with a missing workstream, and underestimated coordination overhead):

- **Fail-fast guard**: Check `git diff` after 2 turns; kill non-productive teams immediately
- **Context budget gate**: Estimate phase cost before starting; checkpoint + stop if insufficient
- **Structured checkpoints**: WORKFLOW.md with full state — enables cold-start resumption from any session
- **Workstream enforcement**: Verify every workstream has output before committing — no silent drops
- **Decision model**: 6-condition matrix (workstream count, dependencies, team failures, context remaining) determines team vs. sequential vs. solo
- **Contract-first protocol**: Dependent workstreams publish interface contracts before implementing
- **No team retry rule**: If a team fails in a session, all remaining phases use sequential

The result: every phase of every PRD executes to verified completion, regardless of session length or agent failures.

## Credits

The **PIV Loop** (Plan-Implement-Validate) was created by [Cole Medin](https://github.com/coleam00) and [Rasmus Widing](https://github.com/rasmuswiding) as part of the [Dynamous Community](https://community.dynamous.ai) [Agentic Coding Course](https://github.com/dynamous-community/agentic-coding-course). Rasmus also created the **PRP framework** that both Ralph PIV and CLUTCH build on.

CLUTCH extends their methodology with parallel Agent Teams — multiple executors working simultaneously on file-exclusive workstreams, coordinated via contract-first protocol.

## Related

- **[Ralph PIV](https://github.com/SmokeAlot420/ralph-piv)** — Solo PIV workflow for Claude Code. Deep analysis, context-rich plans, independent validation. CLUTCH extends Ralph PIV with parallel teams.
- **[TeamBox](https://github.com/SmokeAlot420/teambox)** — Real-time dashboard for monitoring your CLUTCH team sessions live.
- **[Agentic Coding Course](https://github.com/dynamous-community/agentic-coding-course)** — The course where the PIV Loop and PRP framework originated. By Cole Medin & Rasmus Widing.

## License

MIT
