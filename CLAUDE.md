# CLUTCH v2 — Claude Layered Unified Team Coordination Hub

Resilient multi-phase orchestration plugin for Claude Code. Fail-fast guards, context budgeting, structured checkpoints, workstream completion enforcement, and a decision model for team vs. sequential execution. All-markdown — skill definitions, agent defs, process docs, and templates. No build system, no tests, no application code.

## Architecture

```
clutch/
├── .claude-plugin/plugin.json       <- Plugin metadata
├── agents/                          <- Agent definitions
│   ├── piv-executor.md              <- Implements PRP requirements
│   ├── piv-validator.md             <- Verifies all PRP requirements met
│   └── piv-debugger.md              <- Fixes validator-identified gaps
├── skills/
│   └── clutch/                      <- The /clutch skill
│       ├── SKILL.md                 <- Main orchestrator (~600 lines, v2)
│       ├── references/              <- Process docs
│       │   ├── piv-discovery.md     <- Discovery questions for new projects
│       │   ├── create-prd.md        <- PRD generation process
│       │   ├── codebase-analysis.md <- Deep codebase analysis process
│       │   ├── generate-prp.md      <- PRP generation process
│       │   ├── execute-prp.md       <- PRP execution process
│       │   └── team-orchestration.md <- Team lifecycle management
│       └── assets/                  <- Templates
│           ├── prp_base.md          <- PRP template (copied to user project)
│           └── workflow-template.md <- WORKFLOW.md tracker template
├── CLAUDE.md                        <- This file
├── README.md                        <- Public docs
└── LICENSE                          <- MIT
```

## How It Works (v2)

1. **Discover** — If no PRD exists, ask discovery questions and generate one
2. **Analyze** — Fresh sub-agent does deep codebase analysis + PRP generation
3. **Budget Gate** — Check context budget before starting phase; checkpoint + stop if insufficient
4. **Decide** — Decision model chooses team, sequential, or solo based on 6 conditions
5. **Execute** — Team (with fail-fast guard) or sequential single-agents
6. **Fail-Fast** — After 2 turns, check git diff; kill team + go sequential if no output
7. **Contract** — Dependent workstreams publish interface contracts before implementing
8. **Validate** — Independent validator checks all work
9. **Enforce** — Verify EVERY workstream has output before committing (no silent drops)
10. **Checkpoint** — Write structured state to WORKFLOW.md enabling cold-start resumption
11. **Commit** — Orchestrator commits on PASS

Falls back to sequential if team fails, or to solo if only 1 workstream.

## Key Mechanism: CLUTCH_DIR Resolution

SKILL.md resolves the plugin's install path at runtime via `find`, then passes absolute paths to all sub-agents:
- `{CLUTCH_DIR}/skills/clutch/references/codebase-analysis.md`
- `{CLUTCH_DIR}/skills/clutch/references/generate-prp.md`
- `{CLUTCH_DIR}/skills/clutch/assets/prp_base.md`

This ensures fresh sub-agents (which have no inherited context) can locate process docs and templates.

## Claude Code Conventions

### Skill Frontmatter

```yaml
---
name: clutch
description: "CLUTCH — Claude Layered Unified Team Coordination Hub..."
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, TeamCreate, TeamDelete, SendMessage, Read, Write, Bash, Glob, Grep
argument-hint: "[PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]"
---
```

### Agent Frontmatter

```yaml
---
name: piv-executor
description: Executor - implements PRP requirements with fresh context.
tools: Read, Write, Edit, Bash, Glob, Grep
model: inherit
---
```

### Plugin Structure

- Only `plugin.json` goes in `.claude-plugin/`
- Agents, skills, references, and assets go at plugin root

## Rules

- **SKILL.md ~600 lines** — heavy content goes in `references/`
- **Templates in `assets/`** — PRP template, workflow checkpoint template
- **Sub-agents get fresh context** — orchestrator stays lean (~15% context)
- **Sub-agents get absolute paths** — all reference/template paths use `{CLUTCH_DIR}` prefix
- **Flat agent hierarchy** — sub-agents cannot spawn sub-agents
- **Contract-first for dependencies** — upstream publishes interface contracts before downstream spawns
- **Fail fast, recover cheap** — detect non-productive teams in 2 turns, kill and go sequential
- **Budget before you spend** — estimate context cost per phase, checkpoint if insufficient
- **Done means done** — every workstream verified before commit, no silent drops
- **Checkpoint everything** — structured WORKFLOW.md enables cold-start resumption
- **CLUTCH branding in commits** — `Built with CLUTCH v2 - https://github.com/SmokeAlot420/clutch`

## Usage

```bash
claude --plugin-dir ~/clutch
```

Then: `/clutch [PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]`
