<p align="center">
  <h1 align="center"><b>CLUTCH</b></h1>
  <p align="center"><i>Claude Layered Unified Team Coordination Hub</i></p>
  <p align="center">Parallel execution for Claude Code Agent Teams. Multiple executor agents working simultaneously, coordinating via contracts, validated independently.</p>
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

I was using [PIV](https://github.com/SmokeAlot420/piv) (Plan-Implement-Validate) for everything — and it's great for solo execution. Deep analysis, context-rich plans, independent validation. But some phases have work that's clearly parallelizable: frontend + backend + database, or three independent API endpoints, or UI components that don't touch the same files.

Running those serially felt wrong. If the work is file-independent, why not run 2-4 executors at the same time?

That's CLUTCH. Same PIV quality pipeline, but the execution phase runs in parallel using Claude Code's Agent Teams. The PRP defines workstreams (file-exclusive work units), and CLUTCH spawns one executor per workstream. They work simultaneously, then validation catches anything that slipped through.

## Who This Is For

- Devs already using [`/piv`](https://github.com/SmokeAlot420/piv) who hit phases with parallelizable work
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

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                    CLUTCH ORCHESTRATOR                         │
├──────────────────────────────────────────────────────────────┤
│  1. Generate PRP with ## Workstreams section                  │
│  2. Evaluate: 1 workstream → solo, 2-4 → team mode           │
│  3. Check dependencies:                                       │
│     No deps  → spawn all executors in parallel                │
│     Has deps → contract-first staggered spawn                 │
│  4. Spawn validator → PASS / GAPS_FOUND / HUMAN_NEEDED        │
│  5. Debug: assign gaps to responsible executors                │
│  6. Shutdown team, commit, next phase                         │
└──────────────────────────────────────────────────────────────┘
```

1. **PRP with Workstreams** — The PRP includes a Workstreams section defining file-exclusive work units
2. **Team or Solo** — 2-4 workstreams get a team, 0-1 falls back to solo automatically
3. **Contract-First** — Dependent workstreams publish interface contracts before coding. Lead verifies, then forwards to downstream
4. **Parallel Execution** — Independent workstreams spawn all at once. No file conflicts — each executor owns exclusive files
5. **Validate & Debug** — Same independent validation as solo PIV, but gaps get assigned back to the responsible executor (they already have context)

## Why It Works

**Workstream file ownership.** The PRP defines which files each workstream owns. No two executors touch the same file. This is the primary coordination mechanism — it eliminates merge conflicts and stepping-on-toes failures.

**Contract-first protocol.** When workstreams have dependencies (frontend needs the API contract), upstream executors publish their interface contracts *before* implementing. The lead verifies them, then forwards to downstream. No more "3 agents built 3 things that don't connect."

**Fresh context per agent.** Each executor spawns with clean context, reads only their workstream scope + the full PRP. No context drift from long sessions.

## When to Use

| Scenario | Use |
|----------|-----|
| Single-focus phase (one component, one feature) | Solo [`/piv`](https://github.com/SmokeAlot420/piv) |
| Phase with parallelizable work (frontend + backend + DB) | `/clutch` |
| Quick single feature, no PRD | [`/mini-piv`](https://github.com/SmokeAlot420/piv) |
| Unsure | Use `/clutch` — it auto-falls back to solo if only 1 workstream |

## Features

- **Parallel execution** — 2-4 executor teammates working simultaneously
- **Contract-first protocol** — dependent workstreams publish interfaces before implementing
- **Independent validation** — validator doesn't trust executors, verifies everything
- **Debug via messaging** — gaps assigned back to responsible executors (they already have context)
- **Smart fallback** — 1 workstream? Falls back to solo execution automatically
- **Full pipeline** — analyze, PRP gen, execute, validate, debug, commit

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

## v2: Contract-First Protocol

CLUTCH v2 adds **contract-first spawning** for workstreams with dependencies (inspired by [Cole Medin's agent team patterns](https://github.com/coleam00/context-engineering-intro)):

- **Staggered spawn**: Dependent workstreams spawn in order (upstream first, downstream after contract verified)
- **Lead as relay**: Upstream executors publish interface contracts before coding; lead verifies and forwards to downstream
- **Cross-cutting concerns**: Shared behaviors (URL conventions, error shapes) explicitly assigned to one owner
- **Pre-integration diff**: Contract comparison before validation catches mismatches early
- **Anti-pattern prevention**: No more "3 agents built 3 things that don't connect"

**Independent workstreams still spawn fully parallel** — zero overhead when there are no dependencies. The contract-first protocol only activates when the PRP has `depends_on` relationships.

## Related

- **[PIV](https://github.com/SmokeAlot420/piv)** — Solo PIV workflow for Claude Code. Deep analysis, context-rich plans, independent validation. CLUTCH extends PIV with parallel teams.

## License

MIT
