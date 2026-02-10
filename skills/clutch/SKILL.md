---
name: clutch
description: "CLUTCH — Claude Layered Unified Team Coordination Hub. Parallel execution with Agent Teams. Analyze -> PRP -> execute -> validate -> debug pipeline with multiple executor teammates working simultaneously. Contract-first protocol for dependent workstreams. Triggers on: clutch, parallel execution, team execution, multi-agent, piv-teams."
disable-model-invocation: true
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList, TeamCreate, TeamDelete, SendMessage, Read, Write, Bash, Glob, Grep
argument-hint: "[PRD_PATH|PROJECT_PATH] [START_PHASE] [END_PHASE]"
---

# CLUTCH Orchestrator

> Claude Layered Unified Team Coordination Hub

## Arguments: $ARGUMENTS

Parse arguments using this logic:

### PRD Path Mode (first argument ends with `.md`)

If the first argument ends with `.md`, it's a direct path to a PRD file:
- `PRD_PATH` - Direct path to the PRD file
- `PROJECT_PATH` - Derived by going up from PRDs/ folder
- `START_PHASE` - Second argument (default: 1)
- `END_PHASE` - Third argument (default: auto-detect from PRD)

### Project Path Mode

If the first argument does NOT end with `.md`:
- `PROJECT_PATH` - Absolute path to project (default: current working directory)
- `START_PHASE` - Second argument (default: 1)
- `END_PHASE` - Third argument (default: 4)
- `PRD_PATH` - Auto-discover from `PROJECT_PATH/PRDs/` folder

### Detection Logic

```
If $ARGUMENTS[0] ends with ".md":
  PRD_PATH = $ARGUMENTS[0]
  PROJECT_PATH = dirname(dirname(PRD_PATH))
  START_PHASE = $ARGUMENTS[1] or 1
  END_PHASE = $ARGUMENTS[2] or auto-detect from PRD
  PRD_NAME = basename without extension
Else:
  PROJECT_PATH = $ARGUMENTS[0] or current working directory
  START_PHASE = $ARGUMENTS[1] or 1
  END_PHASE = $ARGUMENTS[2] or 4
  PRD_PATH = auto-discover from PROJECT_PATH/PRDs/
  PRD_NAME = discovered PRD basename
```

### Mode Detection

After parsing arguments:
- If PRD_PATH was provided or auto-discovered → **MODE = "execute"**
- If no PRD found → **MODE = "discovery"**

### Auto-Detect Phases from PRD

When PRD_PATH is specified, scan the PRD for phase sections:
1. Look for: `## Phase N:`, `### Phase N:`, `**Phase N:**`, `Phase N:`
2. Set END_PHASE to highest phase found (if not specified)

---

## Required Reading by Role

**CRITICAL: Each role MUST read their instruction files before acting.**

| Role | Instructions |
|------|-------------|
| Discovery (no PRD) | Read references/piv-discovery.md |
| PRD Creation | Read references/create-prd.md |
| PRP Generation | Read references/generate-prp.md |
| Team Orchestration | Read references/team-orchestration.md |
| Executor | Read references/execute-prp.md |

**DO NOT wing it. Follow the established processes.**

**Prerequisite:** A PRD must exist before entering the Phase Workflow. If no PRD exists, enter Discovery Mode.

---

## Discovery Mode (No PRD Found)

When MODE = "discovery":

1. Read references/piv-discovery.md for the discovery process
2. Present discovery questions to the user in a friendly, conversational tone
   - Target audience is vibe coders — keep it approachable
   - Skip questions the user already answered
3. Wait for user answers
4. Fill gaps with your expertise:
   - If user doesn't know tech stack → research and PROPOSE one
   - If user can't define phases → propose 3-4 phases based on scope
   - Always propose-and-confirm: "Here's what I'd suggest — does this sound right?"
5. Run project setup (create PRDs/, PRPs/templates/, PRPs/planning/)
6. Generate PRD: Read references/create-prd.md, write to PROJECT_PATH/PRDs/PRD-{project-name}.md
7. Set PRD_PATH to the generated PRD, auto-detect phases → continue to Phase Workflow

The orchestrator handles discovery and PRD generation directly (no sub-agent needed).

---

## Orchestrator Philosophy

> "Context budget: ~15% orchestrator, 100% fresh per teammate"

You are the **orchestrator**. You stay lean and manage the team. You DO NOT execute PRPs yourself — you spawn specialized teammates with fresh context.

**Before starting any phase**, read references/team-orchestration.md for team lifecycle details.

---

## Phase Workflow

For each phase from START_PHASE to END_PHASE:

### Step 1: Check/Generate PRP

#### Step 1a: Check for existing PRP
```bash
ls -la PROJECT_PATH/PRPs/ 2>/dev/null | grep -i "phase.*N\|pN\|p-N"
```
If a PRP already exists for this phase, skip to Step 2.

#### Step 1b: Spawn Fresh Research Agent for PRP Generation

**CRITICAL: Do NOT generate the PRP yourself. Spawn a FRESH sub-agent.**

Before spawning, the orchestrator must:
1. Read the PRD at PRD_PATH
2. Find the Phase N section
3. Extract the phase scope (title, deliverables, validation criteria)
4. Pass this extracted scope to the fresh agent

Spawn a `general-purpose` sub-agent with this prompt:

```
RESEARCH & PRP GENERATION MISSION - Phase {N}
==============================================

You are generating a PRP for Phase {N}. You have fresh context — use it wisely.

Project root: {PROJECT_PATH}
PRD Path: {PRD_PATH}

## Phase {N} Scope (from PRD)
{paste phase title, deliverables, and validation criteria}

## Step 1: Codebase Analysis
Read and follow the codebase analysis process doc.
Run deep codebase analysis for: {phase feature description}
Save analysis to: {PROJECT_PATH}/PRPs/planning/{PRD_NAME}-phase-{N}-analysis.md

## Step 2: Generate PRP (analysis context still loaded)
Read and follow the PRP generation process doc.
You already have the codebase analysis in your context — use it directly.
DO NOT spawn a sub-agent for this. You do it yourself.
Output PRP to: {PROJECT_PATH}/PRPs/PRP-{PRD_NAME}-phase-{N}.md

IMPORTANT: The PRP MUST include a ## Workstreams section that defines how
work can be split across parallel executor teammates. Each workstream should
own exclusive files. Aim for 2-4 workstreams. See the prp_base.md template.

## Critical Rules
- Do BOTH steps yourself in sequence
- Your analysis context feeds directly into PRP quality
- Follow the full generate-prp process (template, quality gates, info density)
- The PRP template is at: PRPs/templates/prp_base.md
- DO NOT spawn sub-agents for either step
- The Workstreams section is REQUIRED — this PRP will be used for team execution
```

**Wait for the research agent to complete** before proceeding.

### Step 2: Evaluate Workstreams and Choose Execution Mode

Read the generated PRP and find the `## Workstreams` section.

**Count workstreams:**
- **0 or 1 workstream** → **SOLO MODE**: Fall back to solo execution (Step 2a)
- **2-4 workstreams** → **TEAM MODE**: Create team and spawn teammates (Step 2b)
- **5+ workstreams** → Merge smallest workstreams to get 4, then **TEAM MODE**

#### Step 2a: Solo Fallback

Use the Task tool with `subagent_type: "piv-executor"`:

```
EXECUTOR MISSION - Phase {N}
============================

PRP Path: {PRP_PATH}
Project: {PROJECT_PATH}

Read the PRP execution process doc at references/execute-prp.md, then execute the PRP.
Follow: Load PRP → Plan Thoroughly → Execute → Validate → Verify
Output EXECUTION SUMMARY with Status, Files, Tests, Issues.
```

Then skip to Step 4 (spawn validator as solo sub-agent).

#### Step 2b: Team Execution

1. **Create team**: `TeamCreate` with name `{project}-phase-{N}`

2. **Read team-orchestration.md** for full lifecycle details.

3. **For each workstream**, spawn a teammate via Task:
   - `name`: `executor-{workstream-name}`
   - `subagent_type`: `general-purpose`
   - `team_name`: `{project}-phase-{N}`
   - Prompt:

```
EXECUTOR TEAMMATE MISSION - Phase {N}, Workstream: {workstream-name}
====================================================================

PRP Path: {PRP_PATH}
Project: {PROJECT_PATH}

## Your Workstream Scope
{paste the workstream section from the PRP: files owned, dependencies, tasks}

## Instructions
1. Read the PRP at the path above — absorb full context
2. Focus ONLY on your workstream's files and tasks
3. Do NOT touch files owned by other workstreams
4. If you need something from another workstream, message that teammate
5. Follow the execute-prp process: Load → Plan → Execute → Validate
6. When done, output an EXECUTION SUMMARY:
   - Status: COMPLETE / BLOCKED / PARTIAL
   - Files created/modified (list each)
   - Tests written and results
   - Issues encountered
   - Decisions made that affect other workstreams
```

4. **Create tasks** in the shared task list (one per workstream), assign to teammates.
5. **Set up blockedBy** if any workstreams have dependencies.

### Step 2c: Contract-First Protocol (TEAM MODE with dependencies)

If workstreams have `depends_on` relationships, use **staggered spawn** instead of fully parallel spawn:

1. **Map the contract chain** from the PRP's Workstreams section:
   - Workstreams with `depends_on: none` = **upstream** (spawn first)
   - Workstreams with `depends_on: [other]` = **downstream** (spawn after contracts received)

2. **Identify cross-cutting concerns** before spawning ANY executors:
   - URL/path conventions (trailing slashes, query params)
   - Response envelope format (flat vs nested)
   - Error shape (status codes, error body format)
   - Shared constants, config values, data storage semantics
   - Assign each concern to ONE upstream executor. Include in their spawn prompt:
     `"You own the cross-cutting concern: [X]. Define it in your contract."`

3. **Spawn upstream executors first.** Add to their prompt:
   ```
   ## Mandatory: Publish Interface Contract FIRST

   Before writing ANY implementation code, you MUST:
   1. Define your interface contract (exact function signatures, API URLs, response JSON shapes, error formats)
   2. Send it to the lead via SendMessage
   3. WAIT for lead confirmation before proceeding to implementation

   Your contract must include:
   - Exact function signatures or API endpoint URLs (with trailing slashes if applicable)
   - Exact request/response JSON shapes (field names, types, nesting)
   - All status codes for success and error cases
   - Error body format
   - Any streaming/event types or envelope wrappers
   ```

4. **Lead receives and verifies each contract:**
   - Are interfaces explicit? (exact URLs, exact JSON shapes — not "returns user data")
   - Are all status codes specified (200, 400, 404, 500)?
   - Is the error body format specified?
   - Any ambiguities that would cause downstream divergence?
   - If unclear → message executor for clarification before forwarding

5. **Forward verified contracts to downstream executors** in their spawn prompt:
   ```
   ## Contract You Must Conform To

   The following interface contract was published by executor-{upstream} and verified by the lead.
   Build to this contract EXACTLY. Do NOT deviate without asking the lead first.

   {paste contract verbatim}
   ```

6. **If ALL workstreams are independent (no `depends_on`):** skip this step entirely — spawn all in parallel as in Step 2b.

### Step 3: Monitor Team Execution

- Teammates send messages automatically when done or when they need help
- Answer teammate questions promptly
- If a teammate reports BLOCKED, provide guidance or reassign work
- Wait for all executor teammates to mark their tasks complete

### Step 3b: Pre-Integration Contract Diff (TEAM MODE with dependencies)

Before spawning the validator, run a contract diff if workstreams had `depends_on` relationships:

1. For each upstream-downstream pair, ask both executors via SendMessage:
   - Upstream: "What exact interface did you implement? Paste your final contract."
   - Downstream: "What exact interface are you consuming? Paste the contract you built against."
2. Compare the two responses:
   - URL mismatches (trailing slashes, path params, query string format)
   - Response shape mismatches (flat vs nested, missing fields, extra fields)
   - Status code disagreements
   - Error format divergence
3. If mismatches found: send correction to the wrong side, let them fix, then proceed to validator.
4. If no mismatches: proceed directly to validator.

**Skip this step entirely if all workstreams were independent (no `depends_on`).**

### Step 4: Spawn Validator

**Team mode**: Spawn validator as a teammate in the same team:
- `name`: `validator`
- `subagent_type`: `general-purpose`
- `team_name`: `{project}-phase-{N}`

**Solo mode**: Use Task tool with `subagent_type: "piv-validator"`.

Prompt (both modes):

```
VALIDATOR MISSION - Phase {N}
=============================

PRP Path: {PRP_PATH}
Project: {PROJECT_PATH}
Executor Summaries:
{concatenate all executor summaries here}

Verify ALL requirements independently. Don't trust executor claims.
Check every file, every test, every requirement in the PRP.
Output VERIFICATION REPORT with Grade (PASS/GAPS_FOUND/HUMAN_NEEDED), Checks, Gaps.
```

**Process validator result:**
- `PASS` → Proceed to Step 6 (commit)
- `GAPS_FOUND` → Proceed to Step 5 (debug)
- `HUMAN_NEEDED` → Ask user for guidance

### Step 5: Debug Loop (Max 3 iterations)

**Team mode**: Assign gaps to responsible executors via SendMessage:
1. Read the validator's gap list
2. For each gap, identify which executor owns the affected files
3. Send gaps to the responsible executor: "Validator found these issues in your files: {gaps}. Please fix and confirm when done."
4. Wait for executors to confirm fixes
5. Re-run validator (message or re-spawn)

**Solo mode**: Spawn debugger sub-agent using `subagent_type: "piv-debugger"`:

```
DEBUGGER MISSION - Phase {N} - Iteration {I}
============================================

Project: {PROJECT_PATH}
PRP Path: {PRP_PATH}
Gaps: {GAPS}
Errors: {ERRORS}

Fix root causes, not symptoms. Run tests after each fix.
Output FIX REPORT with Status, Fixes Applied, Test Results.
```

After fixes:
- Re-run validator
- If PASS → proceed to commit
- If GAPS_FOUND again → debug again (up to 3 total)
- After 3 iterations → escalate to user

### Step 6: Shutdown Team (Team mode only)

1. Send `shutdown_request` to each teammate via SendMessage
2. Wait for shutdown confirmations
3. Call TeamDelete to clean up

### Step 7: Smart Commit (Orchestrator does this)

After validation passes:
```bash
cd PROJECT_PATH
git status
git diff --stat
```

Create semantic commit:
- Format: `feat/fix/refactor(scope): description`
- Add: `Built with CLUTCH - https://github.com/SmokeAlot420/clutch`

### Step 8: Update Progress

Update `PROJECT_PATH/WORKFLOW.md`:
- Mark phase N as complete
- Note validation results and execution mode (solo/team)
- Record workstream summaries

### Step 9: Next Phase

Increment phase counter. If more phases remain, loop back to Step 1.

---

## Error Handling

### No PRD Found
Enter Discovery Mode (see above).

### Executor Teammate Returns BLOCKED
Message the teammate for details. If unresolvable, reassign work or escalate to user.

### Validator Returns HUMAN_NEEDED
Ask user: "Validator needs guidance on phase N. Question: [details]. Please advise."

### 3 Debug Cycles Exhausted
Ask user: "Phase N failed validation after 3 fix attempts. Persistent issues: [list]. Need your guidance."

### Teammate Timeout/Failure
1. Check for partial work (files created, tests written)
2. Reassign remaining tasks to another teammate or handle via solo fallback
3. If multiple teammates fail, escalate to user

---

## Completion

When all phases are complete, output:
```
## CLUTCH COMPLETE

Phases Completed: START to END
Execution Mode: Team (N workstreams) / Solo (fallback)
Total Commits: N
Validation Cycles: M

### Phase Summary:
- Phase 1: [feature] - [solo/team with N executors] - validated in N cycles
- Phase 2: [feature] - [solo/team with N executors] - validated in N cycles
...

All phases successfully implemented and validated.
```

---

## Visual Workflow

```
┌──────────────────────────────────────────────────────────────────┐
│                    CLUTCH ORCHESTRATOR                            │
│     Claude Layered Unified Team Coordination Hub                 │
├──────────────────────────────────────────────────────────────────┤
│ IF NO PRD FOUND:                                                 │
│   a. Ask discovery questions (piv-discovery.md)                  │
│   b. Generate PRD from answers (create-prd.md)                   │
│   c. Set PRD_PATH, auto-detect phases                            │
│                                                                  │
│ FOR EACH PHASE (START_PHASE to END_PHASE):                       │
│   a. Check if PRP exists                                         │
│   b. If not → spawn RESEARCH AGENT (analysis + PRP gen)          │
│      PRP MUST include ## Workstreams section                     │
│   c. Evaluate workstreams:                                       │
│      0-1 → Solo fallback (spawn 1 piv-executor)                 │
│      2-4 → TeamCreate + team mode                                │
│      5+  → Merge to 4, then team mode                           │
│   d. Check depends_on:                                           │
│      No deps  → spawn all executors in parallel                  │
│      Has deps → contract-first staggered spawn:                  │
│        i.  Identify cross-cutting concerns, assign owners        │
│        ii. Spawn upstream executors (publish contract first)     │
│        iii.Verify contracts, forward to downstream executors     │
│        iv. All executors build in parallel                       │
│   e. Monitor teammates / wait for completion                     │
│   f. Pre-integration contract diff (if deps existed)             │
│   g. Spawn VALIDATOR → PASS / GAPS_FOUND / HUMAN_NEEDED         │
│   h. If GAPS_FOUND → assign gaps to executors (max 3x)          │
│   i. Shutdown team + TeamDelete                                  │
│   j. Commit on PASS                                              │
│   k. Update WORKFLOW.md                                          │
│   l. Next phase                                                  │
└──────────────────────────────────────────────────────────────────┘
```
