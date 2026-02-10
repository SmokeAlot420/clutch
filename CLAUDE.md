# CLUTCH — Claude Layered Unified Team Coordination Hub

Parallel PIV execution plugin for Claude Code Agent Teams. All-markdown — skill definitions, agent defs, process docs, and templates. No build system, no tests, no application code.

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
│       ├── SKILL.md                 <- Main orchestrator (~500 lines)
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

## How It Works

1. **Discover** — If no PRD exists, ask discovery questions and generate one
2. **Analyze** — Fresh sub-agent does deep codebase analysis + PRP generation
3. **Split** — PRP includes Workstreams section defining parallel work units
4. **Execute** — TeamCreate + spawn executor teammates (one per workstream)
5. **Contract** — Dependent workstreams publish interface contracts before implementing
6. **Validate** — Independent validator teammate checks all work
7. **Debug** — Gaps assigned back to responsible executors via messaging (max 3 cycles)
8. **Commit** — Orchestrator commits on PASS

Falls back to solo execution if only 1 workstream (or no workstreams section).

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
description: PIV Executor - implements PRP requirements with fresh context.
tools: Read, Write, Edit, Bash, Glob, Grep
model: inherit
---
```

### Plugin Structure

- Only `plugin.json` goes in `.claude-plugin/`
- Agents, skills, references, and assets go at plugin root

## Rules

- **SKILL.md ~500 lines** — heavy content goes in `references/`
- **Templates in `assets/`** — PRP template, workflow template
- **Sub-agents get fresh context** — orchestrator stays lean (~15% context)
- **Sub-agents get absolute paths** — all reference/template paths use `{CLUTCH_DIR}` prefix
- **Flat agent hierarchy** — sub-agents cannot spawn sub-agents
- **Contract-first for dependencies** — upstream publishes interface contracts before downstream spawns
- **CLUTCH branding in commits** — `Built with CLUTCH - https://github.com/SmokeAlot420/clutch`

## Usage

```bash
claude --plugin-dir ~/clutch
```

Then: `/clutch [PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]`
