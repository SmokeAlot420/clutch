# Team Orchestration for PIV Teams

## Overview

This document covers team-specific orchestration patterns for parallel PIV execution. The orchestrator creates a team of executor teammates that work simultaneously on different workstreams, coordinated through a shared task list and inter-agent messaging.

## Team Lifecycle

```
TeamCreate → Spawn Teammates → Create & Assign Tasks → Monitor → Shutdown → TeamDelete
```

### 1. Create Team

Create one team per phase execution:
- **Team name**: `{project-name}-phase-{N}` (e.g., `teambox-phase-2`)
- Use TeamCreate with the team name

### 2. Spawn Executor Teammates

For each workstream in the PRP, spawn one executor teammate:
- **Teammate name**: `executor-{workstream-name}` (e.g., `executor-frontend`, `executor-api`)
- **Subagent type**: `general-purpose` (needs file editing, bash, search tools)
- **Team name**: Pass the team name so the teammate joins the team
- **Prompt**: Include PRP path, project path, and the teammate's specific workstream scope

Each executor teammate receives:
- The full PRP path (they read it themselves for context)
- Their specific workstream assignment (which files they own, which tasks they handle)
- Instructions to follow the execute-prp.md process for their scope
- Instructions to message other teammates if they need coordination

### 3. Create and Assign Tasks

Use the shared task list for tracking:
- Create one task per workstream using TaskCreate
- Assign each task to the corresponding executor teammate using TaskUpdate
- Set up blockedBy relationships if workstreams have dependencies

### 4. Monitor Execution

The orchestrator monitors teammates:
- Teammates send messages when they complete work or encounter issues
- Messages are delivered automatically — no polling needed
- Answer teammate questions promptly to avoid blocking
- If a teammate gets stuck, provide guidance via SendMessage

### 5. Collect Execution Summaries

Each executor teammate should output an EXECUTION SUMMARY when done:
- Status (COMPLETE / BLOCKED / PARTIAL)
- Files created/modified
- Tests written and their results
- Issues encountered
- Decisions made

The orchestrator collects all summaries for the validator.

### 6. Shutdown and Cleanup

After validation passes (or escalation):
- Send shutdown_request to each teammate via SendMessage
- Wait for shutdown confirmations
- Call TeamDelete to clean up the team

## Teammate Coordination Patterns

### File Ownership

Each workstream exclusively owns its files. This is the primary coordination mechanism:
- **No two teammates should edit the same file**
- If a teammate needs something from another teammate's file, they message and wait
- The PRP's workstream section defines file ownership

### When to Message Teammates

Teammates should message each other when:
- They've created an interface or type that another teammate depends on
- They've discovered something that affects another workstream
- They need a dependency from another workstream to be completed first
- They've made a design decision that impacts shared contracts

### When NOT to Message

Don't message for:
- Status updates (the task list handles this)
- General progress reports (waste of context)
- Questions the PRP already answers

### Dependency Handling

If Workstream B depends on Workstream A:
1. The orchestrator sets up blockedBy in the task list
2. Executor A messages Executor B when the dependency is ready
3. Executor B can start on non-dependent work while waiting

## Validator as Teammate

The validator joins the same team after all executors complete:
- **Teammate name**: `validator`
- **Subagent type**: `general-purpose`
- **Input**: PRP path, project path, all executor summaries concatenated
- The validator can message executor teammates to ask about specific decisions
- Executors should still be alive (not shut down) during validation

### Validator Flow
1. Spawn validator as teammate in the existing team
2. Validator reads PRP and checks ALL requirements independently
3. Validator can message executors: "Why did you implement X this way?"
4. Validator outputs VERIFICATION REPORT with grade

## Debug via Messaging

When the validator finds gaps (GAPS_FOUND):
1. The orchestrator reads the gap list from the validator's report
2. For each gap, identify which workstream/executor owns the affected files
3. Send the specific gaps to the responsible executor via SendMessage
4. The executor fixes the issues and messages back when done
5. Re-spawn or message the validator to re-check

This is more efficient than spawning new debugger agents because:
- Executors already have context about their workstream
- No context re-loading needed
- Executors can coordinate fixes if gaps span workstreams

### Debug Loop Rules
- Maximum 3 debug iterations (same as solo PIV)
- Each iteration: assign gaps → executors fix → validator re-checks
- After 3 failures: escalate to user with all context

## Team Sizing Guidelines

| Workstreams in PRP | Executor Teammates | Notes |
|---|---|---|
| 0-1 | Don't use teams | Fall back to solo /piv behavior |
| 2 | 2 | Minimum for teams to be worthwhile |
| 3-4 | 3-4 | Sweet spot for most projects |
| 5+ | Cap at 4 | Merge smallest workstreams; more = more overhead |

### When to Merge Workstreams

If the PRP defines 5+ workstreams:
- Combine the two smallest workstreams
- Combine workstreams with heavy dependencies on each other
- Target 4 total executors maximum

## Graceful Degradation

### Teammate Gets Stuck
If a teammate stops responding or reports BLOCKED:
1. Check what they accomplished (read their files, check task status)
2. Try sending a clarifying message
3. If still stuck after 1 retry: take their remaining tasks and assign to another teammate or handle as orchestrator

### Team Creation Fails
Fall back to solo /piv behavior — spawn one piv-executor sub-agent.

### Partial Completion
If some workstreams complete but others fail:
1. Commit the successful workstreams (if they pass validation independently)
2. Escalate the failed workstreams to the user
3. Don't block successful work on failed work

## Team Naming Convention

```
Team:       {project}-phase-{N}
Executors:  executor-{workstream-name}
Validator:  validator
Tasks:      {workstream-name}: {brief description}
```

Examples:
```
Team:       teambox-phase-2
Executors:  executor-frontend, executor-api, executor-database
Validator:  validator
Tasks:      "frontend: Build React components for team dashboard"
            "api: Implement REST endpoints for team CRUD"
            "database: Create migration and seed data"
```
