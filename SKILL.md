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

Run the full SDD cycle: clarifying questions → SPEC → PLAN → optional audit → task preview → implement.

## When to use

- Feature spans multiple files or domains
- Business logic has non-obvious rules (auth, idempotency, boundaries)
- Legacy codebase where "obvious" context isn't embedded in the code
- Multiple agents will work on the feature in parallel

## When NOT to use

- Small bug fix, config change, or single-file edit → use Plan mode instead
- For very large features, consider splitting into independent sub-features, each with its own SDD cycle

---

## Platform Adapters

This skill is platform-agnostic. Where specific tool calls are needed, use the adapter for your environment:

| Operation | Claude Code | Terminal (tmux) | Cursor / Codex / other |
|-----------|-------------|-----------------|------------------------|
| **Lattice: create task** | `lattice_create` MCP tool | `lattice create` CLI | `lattice create` CLI |
| **Lattice: link tasks** | `lattice_link` MCP tool | `lattice link` CLI | `lattice link` CLI |
| **Lattice: list/check** | `lattice_list` MCP tool | `lattice list` CLI | `lattice list` CLI |
| **Spawn worker** | `Agent` tool with `isolation: "worktree"` | `agentic-workspace spawn` (requires tmux) | run a new agent session in a fresh worktree |
| **ia-bridge audit** | `mcp__ia-bridge-mcp__single_opinion_run` | `ia-bridge` CLI | `ia-bridge` CLI |
| **Check Lattice init** | `lattice_list` MCP tool → inspect error | `lattice list` CLI → inspect error | `lattice list` CLI → inspect error |
| **Lattice: archive task** | `lattice_archive` MCP tool | `lattice archive` CLI | `lattice archive` CLI |
| **Lattice: stop dashboard** | n/a (MCP only) | `lattice dashboard --stop` | `lattice dashboard --stop` |

If none of the above worker-spawning options are available, fall back to sequential self-implementation (Phase 4, mode 2).

---

## Prerequisites check

**Before starting Phase 1**, run two checks:

### 1. Lattice availability

Check using the adapter for your platform (MCP tool or CLI — see Platform Adapters above).

- **If it succeeds** — Lattice is installed, proceed to check 2.
- **If it fails with "command not found" or similar** — Lattice is not installed. Tell the user:

  ```
  ⚠️  Lattice is not installed. It is required for Phase 4 (task tracking and worker spawning).

  To install:
    uv tool install --force 'lattice-tracker[mcp]'

  If uv is not installed:
    curl -LsSf https://astral.sh/uv/install.sh | sh
    source ~/.local/bin/env
    uv tool install --force 'lattice-tracker[mcp]'

  For Claude Code: also register the Lattice MCP server in your Claude Code settings
    (the MCP server is bundled with lattice-tracker[mcp] — see README for setup).
    Then restart your agent environment so the MCP is picked up.

  Options:
    [1] Stop here and install now (recommended)
    [2] Continue without Lattice — Phase 4 will run in sequential mode (no parallel workers)
  ```

  Wait for the user's choice before proceeding.

### 2. Lattice project initialization

If Lattice is available, verify it is initialized for this project.

- **If it returns a project** — all good, proceed to Phase 1.
- **If it fails with "no .lattice/ directory" or "project_code missing"** — the project has not been initialized. Tell the user:

  ```
  ⚠️  Lattice is installed but not initialized for this project.

  Run:
    lattice init

  Then set your project_code in .lattice/config (e.g. project_code = MYPROJ).
  Signal when ready.
  ```

  Wait for confirmation before proceeding to Phase 1.

---

## Phase 1: SPEC

**Goal:** Define *what* to build. Functional layer only — no implementation details yet.

### Steps

1. Read the feature description from `$ARGUMENTS` (or from the user's prompt if invoked without a slash command).
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
4. Once answers are in, generate `SDD/<feature-slug>/SPEC.md` using `templates/SPEC.md`.
   - Use `REQ-001`, `REQ-002`, ... for all functional requirements.
   - Write acceptance criteria in Given/When/Then format, referencing the REQ.
   - Keep it tech-agnostic — no implementation decisions here.

### Gate

**PAUSE.** Present the spec. Ask the user to approve or request changes.

**Revision loop:** If rejected, amend the spec in-place, show the diff, and re-present. Maximum 3 revision rounds. If still unresolved after 3 rounds, ask the user to either edit the file directly and signal when ready, or approve as-is and log the disagreement in the Decision Log.

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
3. Generate `SDD/<feature-slug>/PLAN.md` using `templates/PLAN.md`.
   - Every file that will be created or modified must appear in the File Change Manifest.
   - Map each change to its REQ refs.
   - Specify rollback procedure, failure modes, observability, and risks.
   - Reference existing patterns explicitly — file paths and line numbers.
   - List which tools or MCPs workers should use.

### Gate

**PAUSE.** Present the plan. Ask the user to approve or request changes.

**Revision loop:** Same as SPEC — amend in-place, show diff, max 3 rounds, then escalate (edit directly or approve as-is with disagreement in the Decision Log).

---

## Phase 2.5: AUDIT (optional)

**Goal:** Surface gaps or risks in the plan via a peer agent before committing to task decomposition.

After the PLAN is approved, offer an optional audit step:

```
PLAN approved. Before decomposing into tasks, would you like an ia-bridge peer audit?
ia-bridge runs a structured review by a second agent (Codex or another Claude instance)
and can surface issues the primary agent missed — particularly import/API contract gaps.
See: https://github.com/AlterMundi/ia-bridge-mcp

  [1] Yes — run ia-bridge audit (recommended for risky or complex changes)
  [2] No — proceed directly to Phase 3
```

If the user chooses [1]:

1. Run the audit using the ia-bridge adapter for your platform (see Platform Adapters above), passing the PLAN content as input.
2. Present the audit findings verbatim.
3. **Surface this caveat to the user:** The reviewing agent may not have direct file access. Any finding that references specific file contents should be verified against the actual code before acting on it.
4. For each finding, ask: "Does this change the plan?" If yes, amend `PLAN.md`, show the diff, and re-present for approval before proceeding.
5. Once findings are resolved (or none required changes), proceed to Phase 3.

---

## Phase 3: TASK PREVIEW

**Goal:** Break the plan into small, ordered, self-contained tasks ready for execution.

### Execution model decision — ask FIRST

Before decomposing tasks, ask the user which execution model Phase 4 will use — this changes how task descriptions must be written:

```
Before I decompose tasks: which execution model will Phase 4 use?
  [1] Parallel workers — tasks dispatched to separate agent sessions (worktrees)
  [2] Sequential self-implementation — I implement each task myself, one at a time

Parallel workers require fully self-contained task descriptions with repo context
embedded. Sequential tasks can rely on shared session context.
```

Wait for the answer before decomposing.

### Steps

1. Read `SDD/<feature-slug>/SPEC.md` and `SDD/<feature-slug>/PLAN.md`.
2. Decompose into tasks. Each task must:
   - Be completable in a single agent session
   - Produce a verifiable change with tests
   - Have no ambiguity — all context embedded
   - Reference its `spec_ref: REQ-###`
   - Declare dependencies on other tasks
3. **If execution model is parallel (mode 1):** each task description must include a "Cold Agent Context" section (see below). Sequential tasks (mode 2) may omit it.
4. Present the task list as a **transient preview** in chat — do not create a `TASKS.md` file. Format:

> **`review_steps`** — commands the orchestrator runs after a task completes to verify correctness (build, tests, lints). Must be fast, deterministic, and exit non-zero on failure.

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
  review_steps:
    - run: npm run build
    - run: npm test src/items/
    - grep: "import.*\.\./" src/items/create.ts  # check for alias leakage
  cold_agent_context: |    # only for parallel/worktree mode
    Repo layout: packages/api (Express), packages/web (Next.js), packages/shared (types).
    Files that EXIST and must be modified: src/items/create.ts (line 42: handler entry point).
    Files that must be CREATED: src/items/create.test.ts (does not exist yet).
    Critical pattern: use the @api alias (see tsconfig.json paths) — never relative imports
    across packages. See packages/api/src/auth/verify.ts:12 for the pattern to follow.
    Gotcha: Three.js is loaded externally via CDN; do not import it directly.

TASK-002: <title>
  spec_ref: REQ-002
  deps: TASK-001
  ...
```

5. Check: is the list overengineered? Can tasks be merged? Flag if yes. If a task touches more than 5 files, verify it is still atomic and independently reviewable — if not, split it.

### Cold Agent Context guidance

For parallel/worktree mode, every task description must give a cold agent enough to start without reading the whole repo. Include:

- **Repo structure summary** — top-level packages or directories relevant to this task
- **Files that exist vs. files to create** — be explicit; don't assume the agent can infer this
- **Alias/import conventions** — e.g. `@api`, `@engine` aliases, where they're defined
- **Critical gotchas** — external deps loaded via CDN, generated files not to edit, env vars required
- **Pattern reference** — one concrete `file:line` example of the pattern the task should follow

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

**Goal:** Materialize tasks and dispatch workers to implement them.

> **CRITICAL — You are the orchestrator, not the implementor.**
> Do NOT write code yourself in this phase. Your job is to create tasks and spawn
> workers that will do the implementation. The only exception is if the user explicitly asks
> you to implement directly (e.g. "just do it yourself, no workers").

The execution model was already decided at the start of Phase 3. Use that choice.

If the user chose **mode 2 (sequential)**, skip the worker steps below and implement each task
sequentially, pausing for checkpoint after each one.

### Steps (mode 1 — parallel workers)

1. Create Lattice tasks using the adapter for your platform. For each task from the approved preview:
   - MCP: `lattice_create(title="<title>", description="spec_ref: REQ-###\n\n<acceptance criteria>\n\nCold Agent Context:\n<cold_agent_context>\n\nReview steps:\n<review_steps>\n\nTest plan: <test plan>")`
   - CLI: `lattice create --title "<title>" --description "..."`

   Capture the returned task ID (e.g. `PROJ-1`) for each task.

2. Link dependencies using the returned IDs:
   - MCP: `lattice_link(source_id="PROJ-1", target_id="PROJ-2", link_type="blocks")`
   - CLI: `lattice link PROJ-1 blocks PROJ-2`

3. Spawn workers for independent (unblocked) tasks using the adapter for your platform:

   **Claude Code** — use the `Agent` tool with `isolation: "worktree"`. Spawn independent tasks in parallel (multiple Agent calls in one message):
   ```
   Agent(
     subagent_type="general-purpose",
     isolation="worktree",
     prompt="<full task description including cold agent context and review steps>"
   )
   ```

   **Interactive terminal with tmux** — use `agentic-workspace`:
   ```bash
   agentic-workspace collab-init --project . --agents "claude:implementor,codex:auditor"
   agentic-workspace start --project . --orchestrator claude --heartbeat-interval 30
   agentic-workspace spawn claude --task PROJ-1
   agentic-workspace spawn codex  --task PROJ-2 --worktree
   ```

   **Cursor / Codex / other** — open a new agent session in a fresh `git worktree` branch and paste the full task description (including cold agent context). Coordinate merges back to the working branch when tasks complete.

   **No worker spawning available** — fall back to sequential mode: implement each task yourself, one at a time.

### Execution checkpoints

Do NOT pause after every single task. Instead, checkpoint at:
- **Dependency boundaries** — when a blocking task completes before dependents start
- **Every 3 tasks** — if tasks are small and sequential

At each checkpoint: review completed work, confirm it satisfies acceptance criteria, then proceed.

### Worker Failure Protocol

If a worker reports a failure, do not retry indefinitely. Apply this decision tree:

- **Test failure** — retry once with a clean state (fresh worktree or cleared cache). If it persists, mark the Lattice task as `blocked`, attach the failure diff as a comment, and report to the user.
- **Merge conflict** — attempt rebase. If unresolved, mark `blocked` and escalate to the user.
- **Timeout / stall** — reduce the task scope or split it into smaller tasks. Flag the partial work to the user.
- Always log the failure and decision as a Lattice task comment (MCP: `lattice_comment` / CLI: `lattice comment`).

### Spec Amendment Protocol

If implementation reveals that a requirement must change mid-Phase 4:

1. Pause active workers on affected tasks.
2. Amend `SDD/<feature-slug>/SPEC.md` in-place — bump the `Version` field and append an entry to the `Amendment Log`.
3. Identify stale tasks by their `spec_ref` tag matching the changed REQ.
4. **PAUSE.** Present the amendment and affected tasks to the user. Get explicit approval.
5. Update or recreate the stale Lattice tasks to reflect the new requirements.
6. Resume workers.

> Do not restart from Phase 1. Amendments are surgical — only the affected REQs and tasks change.

### Completion

When all tasks are done:
1. Verify all acceptance criteria from SPEC are satisfied.
2. Commit `SDD/<feature-slug>/` to the repo alongside the code changes.
3. Report: tasks completed, any open risks, any deviations from the spec.
4. Clean up Lattice:
   - Archive all completed tasks — MCP: `lattice_archive(task_id="...", actor="...")` for each / CLI: `lattice archive <task-id>`
   - Stop the dashboard if it was started: `lattice dashboard --stop`

---

## Resume Protocol

If an SDD session is interrupted, resume by:
1. Detecting existing `SDD/<feature-slug>/` — skip completed phases.
2. Checking Lattice task states (via MCP or CLI) to determine where execution left off.
3. Continuing from the first incomplete phase or first pending task.

Never restart from scratch if artifacts already exist. Present what was found and ask the user to confirm before resuming.

---

## Artifact Reference

| File | Purpose | Committed? |
|------|---------|-----------|
| `SDD/<slug>/SPEC.md` | Functional spec | Yes |
| `SDD/<slug>/PLAN.md` | Technical plan | Yes |
| Task list | Transient preview only — source of truth is Lattice | No |

Templates: `templates/`
