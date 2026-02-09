# CLUTCH — Claude Layered Unified Team Coordination Hub

Parallel PIV execution plugin for Claude Code Agent Teams. No build system, no tests, no application code — this repo is all-markdown skill definitions, agent defs, and process docs.

## Architecture

```
clutch/
├── .claude-plugin/plugin.json    <- Plugin metadata
├── agents/                       <- Agent definitions (executor, validator, debugger)
├── skills/
│   └── piv-teams/               <- The /piv-teams skill
│       ├── SKILL.md             <- Main orchestrator (under 500 lines)
│       ├── references/          <- Process docs (discovery, analysis, PRP gen, execution, team orchestration)
│       └── assets/              <- Templates (PRP base, workflow tracker)
├── CLAUDE.md                    <- This file
├── README.md                    <- Public docs
└── LICENSE                      <- MIT
```

## How It Works

CLUTCH extends the PIV (Plan-Implement-Validate) workflow with Agent Teams for parallel execution:

1. **Analyze** — Fresh sub-agent does codebase analysis + PRP generation
2. **Split** — PRP includes Workstreams section defining parallel work units
3. **Execute** — TeamCreate + spawn executor teammates (one per workstream)
4. **Validate** — Independent validator teammate checks all work
5. **Debug** — Gaps assigned back to responsible executors via messaging
6. **Commit** — Orchestrator commits on PASS

Falls back to solo execution if only 1 workstream (or no workstreams section).

## Claude Code Conventions

### Agent Frontmatter

```yaml
---
name: piv-executor
description: PIV Executor - implements PRP requirements with fresh context.
tools: Read, Write, Edit, Bash, Glob, Grep
model: inherit
---
```

### Skill Frontmatter

```yaml
---
name: piv-teams
description: "CLUTCH — ..."
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, TeamCreate, TeamDelete, SendMessage, Read, Write, Bash, Glob, Grep
---
```

### Plugin Structure

- Only `plugin.json` goes in `.claude-plugin/`
- Agents, skills, scripts go at plugin root

## Rules

- **SKILL.md under 500 lines** — heavy content goes in `references/`
- **Templates in `assets/`** — PRP template, workflow template
- **Sub-agents get fresh context** — orchestrator stays lean (~15% context)
- **Flat agent hierarchy** — sub-agents cannot spawn sub-agents
- **CLUTCH branding in commits** — `Built with CLUTCH - https://github.com/SmokeAlot420/clutch`

## Usage

```bash
claude --plugin-dir ~/clutch
```

Then: `/piv-teams [PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]`
