<p align="center">
  <h1 align="center"><b>CLUTCH</b></h1>
  <p align="center"><i>Claude Layered Unified Team Coordination Hub</i></p>
  <p align="center">Parallel PIV execution for Claude Code Agent Teams. Multiple executor agents working simultaneously, coordinating via messaging, validated independently.</p>
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

## What is CLUTCH?

CLUTCH brings **parallel execution** to the [PIV (Plan-Implement-Validate)](https://github.com/SmokeAlot420/ftw) workflow using Claude Code's **Agent Teams**.

Instead of one executor working through tasks serially, CLUTCH splits work into **workstreams** and spawns multiple executor teammates that work **simultaneously** — then validates everything independently.

```
Solo PIV:     Analyze → Execute (1 agent) → Validate → Debug
CLUTCH:       Analyze → Execute (2-4 agents in parallel) → Validate → Debug
```

## How It Works

```
┌──────────────────────────────────────────────────────────┐
│                  CLUTCH ORCHESTRATOR v2                    │
├──────────────────────────────────────────────────────────┤
│  1. Generate PRP with ## Workstreams section              │
│  2. Evaluate: 1 workstream → solo, 2-4 → team mode      │
│  3. Check depends_on:                                     │
│     No deps  → spawn all executors in parallel            │
│     Has deps → contract-first staggered spawn:            │
│       a. Spawn upstream executors first                   │
│       b. Receive & verify interface contracts             │
│       c. Forward contracts to downstream executors        │
│       d. All executors build in parallel                  │
│  4. Pre-integration contract diff (if deps existed)       │
│  5. Spawn validator → PASS / GAPS_FOUND / HUMAN_NEEDED   │
│  6. Debug: assign gaps back to responsible executors      │
│  7. Shutdown team, commit, next phase                     │
└──────────────────────────────────────────────────────────┘
```

### The Key Insight: Workstreams

The PRP (Project Requirements Plan) includes a **Workstreams** section that defines how work splits across parallel executors:

```markdown
## Workstreams

### Workstream 1: frontend
- **Files owned**: src/components/, src/pages/
- **Depends on**: none
- **Tasks**: Build React components, page routing

### Workstream 2: api
- **Files owned**: src/api/, src/middleware/
- **Depends on**: none
- **Tasks**: REST endpoints, auth middleware

### Workstream 3: database
- **Files owned**: src/models/, migrations/
- **Depends on**: none
- **Tasks**: Schema, migrations, seed data
```

Each workstream becomes one executor teammate. No file conflicts — each executor owns exclusive files.

## Quick Start

```bash
# Clone
git clone https://github.com/SmokeAlot420/clutch.git

# Use as Claude Code plugin
claude --plugin-dir ./clutch

# Run on a project with a PRD
/piv-teams ~/my-project/PRDs/PRD.md 1

# Run on a project (auto-discover PRD)
/piv-teams ~/my-project

# Run specific phases
/piv-teams ~/my-project 2 3
```

## When to Use CLUTCH vs Solo PIV

| Scenario | Use |
|----------|-----|
| Single-focus phase (one component, one feature) | Solo `/piv` via [FTW](https://github.com/SmokeAlot420/ftw) |
| Phase with parallelizable work (frontend + backend + DB) | `/piv-teams` via CLUTCH |
| Unsure | Use CLUTCH — it auto-falls back to solo if only 1 workstream |

## Features

- **Parallel execution** — 2-4 executor teammates working simultaneously
- **Inter-agent messaging** — teammates coordinate on dependencies
- **Independent validation** — validator doesn't trust executors, verifies everything
- **Debug via messaging** — gaps assigned back to responsible executors (they already have context)
- **Smart fallback** — 1 workstream? Falls back to solo PIV automatically
- **Full PIV pipeline** — analyze, PRP gen, execute, validate, debug, commit

## Plugin Structure

```
clutch/
├── .claude-plugin/plugin.json
├── agents/
│   ├── piv-executor.md
│   ├── piv-validator.md
│   └── piv-debugger.md
└── skills/
    └── piv-teams/
        ├── SKILL.md              # Orchestrator (407 lines)
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

## v2: Contract-First Protocol

CLUTCH v2 adds **contract-first spawning** for workstreams with dependencies (inspired by [Cole Medin's agent team patterns](https://github.com/coleam00/context-engineering-intro)):

- **Staggered spawn**: Dependent workstreams spawn in order (upstream first, downstream after contract verified)
- **Lead as relay**: Upstream executors publish interface contracts before coding; lead verifies and forwards to downstream
- **Cross-cutting concerns**: Shared behaviors (URL conventions, error shapes) explicitly assigned to one owner
- **Pre-integration diff**: Contract comparison before validation catches mismatches early
- **Anti-pattern prevention**: No more "3 agents built 3 things that don't connect"

**Independent workstreams still spawn fully parallel** — zero overhead when there are no dependencies. The contract-first protocol only activates when the PRP has `depends_on` relationships.

## Related

- **[FTW (First Try Works)](https://github.com/SmokeAlot420/ftw)** — Solo PIV workflow for Claude Code + OpenClaw. CLUTCH extends FTW with parallel teams.

## License

MIT
