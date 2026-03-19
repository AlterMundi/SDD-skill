---
name: sdd
description: >
  Spec-Driven Development workflow. Guides through Spec → Plan → Tasks → Implement
  to eliminate agent ambiguity before writing code. Use for complex features,
  multi-file changes, or anything touching multiple domains. Not needed for simple
  bug fixes or single-file changes — use Plan mode for those.
argument-hint: "<feature description>"
---

# SDD — Spec-Driven Development

Run the full SDD cycle: clarifying questions → SPEC → PLAN → task preview → implement.

## When to use

- Feature spans multiple files or domains
- Business logic has non-obvious rules (auth, idempotency, boundaries)
- Legacy codebase where "obvious" context isn't embedded in the code
- Multiple agents will work on the feature in parallel

## When NOT to use

- Small bug fix, config change, or single-file edit → use Plan mode instead

---

## Prerequisites check

**Before starting Phase 1**, verify Lattice is available by calling `lattice_list` via MCP.

- **If it succeeds** — all good, proceed to Phase 1.
- **If it fails** — Lattice is not installed. Tell the user:

  ```
  ⚠️  Lattice is not installed. It is required for Phase 4 (task tracking and worker spawning).

  To install:
    uv tool install --force 'lattice-tracker[mcp]'
    cd <path-to-Skills-repo>/mcps/agentic-workspace && ./install.sh

  If uv is not installed:
    curl -LsSf https://astral.sh/uv/install.sh | sh
    source ~/.local/bin/env
    uv tool install --force 'lattice-tracker[mcp]'
    cd <path-to-Skills-repo>/mcps/agentic-workspace && ./install.sh

  After installing, restart Claude Code so the Lattice MCP is picked up.

  Options:
    [1] Stop here and install now (recommended)
    [2] Continue without Lattice — Phase 4 will run in sequential mode (no parallel workers)
  ```

  Wait for the user's choice before proceeding. If they choose [2], note that Phase 4 will
  use sequential mode regardless of task count, and skip all Lattice/agentic-workspace steps.

---

## Phase 1: SPEC

**Goal:** Define *what* to build. Functional layer only — no implementation details yet.

### Steps

1. Read the feature description from `$ARGUMENTS`.
2. Work through the assumptions checklist before writing anything. For each dimension, either fill it from existing codebase knowledge or ask the user:
   - **Actors** — who performs this action? What roles exist?
   - **Boundaries** — which system, service, or API? Which "backoffice"?
   - **Idempotency** — can the operation be safely repeated?
   - **Authorization** — who is allowed? What happens if unauthorized?
   - **Error states** — what are the failure modes?
   - **Data lifecycle** — create/update/delete effects, cascades?
   - **Observability** — what needs to be logged or measured?
   - **Back-compat** — does this break existing behavior?
3. Ask only about the dimensions you cannot confidently infer. Batch questions — do not ask one at a time.
4. Once answers are in, generate `SDD/<feature-slug>/SPEC.md` using `skills/sdd/templates/SPEC.md`.
   - Use `REQ-001`, `REQ-002`, ... for all functional requirements.
   - Write acceptance criteria in Given/When/Then format, referencing the REQ.
   - Keep it tech-agnostic — no implementation decisions here.

### Gate

**PAUSE.** Present the spec. Ask the user to approve or request changes.

**Revision loop:** If rejected, amend the spec in-place, show the diff, and re-present. Maximum 3 revision rounds. If still unresolved after 3 rounds, ask the user to edit the file directly and signal when ready.

---

## Phase 2: PLAN

**Goal:** Define *how* to build it. Technical layer — architecture, data, testing, rollback.

### Steps

1. Read the approved `SDD/<feature-slug>/SPEC.md`.
2. Explore the codebase. Produce an internal summary covering:
   - APIs and endpoints touched
   - Hotspots (files likely to change)
   - Existing patterns to follow (with file:line references)
   - Dependencies and migration surface
3. Generate `SDD/<feature-slug>/PLAN.md` using `skills/sdd/templates/PLAN.md`.
   - Every file that will be created or modified must appear in the File Change Manifest.
   - Map each change to its REQ refs.
   - Specify rollback procedure, failure modes, observability, and risks.
   - Reference existing patterns explicitly — file paths and line numbers.
   - List which MCPs or tools workers should use.

### Gate

**PAUSE.** Present the plan. Ask the user to approve or request changes.

**Revision loop:** Same as SPEC — amend in-place, show diff, max 3 rounds, then escalate.

---

## Phase 3: TASK PREVIEW

**Goal:** Break the plan into small, ordered, self-contained tasks ready for Lattice.

### Steps

1. Read `SDD/<feature-slug>/SPEC.md` and `SDD/<feature-slug>/PLAN.md`.
2. Decompose into tasks. Each task must:
   - Be completable in a single agent session
   - Produce a verifiable change with tests
   - Have no ambiguity — all context embedded
   - Reference its `spec_ref: REQ-###`
   - Declare dependencies on other tasks
3. Present the task list as a **transient preview** in chat — do not create a `TASKS.md` file. Format:

```
TASK-001: <title>
  spec_ref: REQ-001
  deps: none
  files: [src/items/create.ts, src/items/create.test.ts]
  acceptance:
    - [ ] Idempotent on duplicate calls
    - [ ] Returns 403 for non-admin actors
    - [ ] Unit tests cover error paths
  test_plan: Unit + integration with real DB

TASK-002: <title>
  spec_ref: REQ-002
  deps: TASK-001
  ...
```

4. Check: is the list overengineered? Can tasks be merged? Flag if yes.

### Write-scope confirmation

Before proceeding, present the full list of files and directories that will be modified. Get explicit user approval.

```
Files to be modified:
  src/items/create.ts       (create)
  src/items/create.test.ts  (create)
  src/routes/admin.ts       (modify)
  ...
Proceed?
```

### Gate

**PAUSE.** Ask the user to approve the task list and write-scope.

---

## Phase 4: IMPLEMENT

**Goal:** Materialize tasks in Lattice and dispatch workers to implement them.

> **CRITICAL — You are the orchestrator, not the implementor.**
> Do NOT write code yourself in this phase. Your job is to create Lattice tasks and spawn
> workers that will do the implementation. The only exception is if the user explicitly asks
> you to implement directly (e.g. "just do it yourself, no workers").

### Execution mode

Ask the user before proceeding:

```
Ready to create Lattice tasks and spawn workers.
Execution mode:
  [1] Full — parallel workers via agentic-workspace (recommended for 3+ tasks)
  [2] Sequential — I implement each task myself, one at a time (ok for 1-2 small tasks)
Which mode?
```

If the user chooses **mode 2**, skip the worker steps below and implement each task
sequentially, pausing for checkpoint after each one.

### Steps (mode 1 — parallel workers)

1. Create Lattice tasks using the MCP tools. For each task from the approved preview:
   ```
   lattice_create(title="<title>", description="spec_ref: REQ-###\n\n<acceptance criteria>\n\nTest plan: <test plan>")
   ```
   Capture the returned task ID (e.g. `PROJ-1`) for each task.

2. Link dependencies using the returned IDs:
   ```
   lattice_link(source_id="PROJ-1", target_id="PROJ-2", link_type="blocks")
   ```

3. Initialize IAxP governance and start the workspace:
   ```bash
   agentic-workspace collab-init --project . --agents "claude:implementor,codex:auditor"
   agentic-workspace start --project . --orchestrator claude --heartbeat-interval 30
   ```

4. Spawn workers for independent (unblocked) tasks:
   ```bash
   agentic-workspace spawn claude --task PROJ-1
   agentic-workspace spawn codex  --task PROJ-2 --worktree   # runs in parallel with PROJ-1
   ```

### Execution checkpoints

Do NOT pause after every single task. Instead, checkpoint at:
- **Dependency boundaries** — when a blocking task completes before dependents start
- **Every 3 tasks** — if tasks are small and sequential

At each checkpoint: review completed work, confirm it satisfies acceptance criteria, then proceed.

### Completion

When all tasks are done:
1. Verify all acceptance criteria from SPEC are satisfied.
2. Commit `SDD/<feature-slug>/` to the repo alongside the code changes.
3. Report: tasks completed, any open risks, any deviations from the spec.

---

## Resume Protocol

If an SDD session is interrupted, resume by:
1. Detecting existing `SDD/<feature-slug>/` — skip completed phases.
2. Checking Lattice task states to determine where execution left off.
3. Continuing from the first incomplete phase or first pending task.

Never restart from scratch if artifacts already exist. Present what was found and ask the user to confirm before resuming.

---

## Artifact Reference

| File | Purpose | Committed? |
|------|---------|-----------|
| `SDD/<slug>/SPEC.md` | Functional spec | Yes |
| `SDD/<slug>/PLAN.md` | Technical plan | Yes |
| Task list | Transient preview only — source of truth is Lattice | No |

Templates: `skills/sdd/templates/`
