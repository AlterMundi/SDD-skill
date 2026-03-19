# SDD — Spec-Driven Development

A methodology and agent skill for building features with AI coding agents without ambiguity.

## Why SDD?

When you give an agent a vague prompt, it guesses. It guesses which backoffice, which API contract, which auth model, which error handling strategy. Each guess is a silent decision. Some are right. Some are wrong. As complexity grows, so does the gap between what you meant and what the agent built.

SDD solves this by defining *what* to build and *how* to build it before a single line of code is written.

> "Vibe coding builds demos and MVPs. Spec-Driven Development builds production systems."
>
> — [@juliandeangeIis](https://x.com/juliandeangeIis/status/2033303156340240481)

## Prerequisites

SDD phases 1–3 (Spec, Plan, task preview) work with no setup beyond the skill itself.
Phase 4 (parallel worker execution) requires Lattice and `agentic-workspace`:

```bash
# If uv is not installed:
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env

# Install Lattice
uv tool install --force 'lattice-tracker[mcp]'

# Install agentic-workspace (registers Lattice MCP in Claude Code)
cd <path-to-Skills-repo>/mcps/agentic-workspace && ./install.sh
```

Then **restart Claude Code** so the Lattice MCP is picked up.

If Lattice is not installed, the skill will detect it at startup and offer to either stop for
installation or continue in sequential mode (no parallel workers).

## How It Works

```
/sdd "feature description"
   │
   ├─ Clarifying questions → surface hidden assumptions
   │
   ├─ SDD/<feature-slug>/SPEC.md    ← PAUSE: human approves
   │
   ├─ Codebase exploration
   ├─ SDD/<feature-slug>/PLAN.md    ← PAUSE: human approves
   │
   ├─ Transient task preview (shown in chat)
   │   └─ lattice create for each task with spec_ref backlinks
   │                                ← PAUSE: human approves
   │
   └─ agentic-workspace spawn workers → batched execution checkpoints
```

## The Four Phases

| Phase | Output | Human gate |
|-------|--------|------------|
| **Spec** | `SDD/<slug>/SPEC.md` — *what* to build, functional layer only | Required |
| **Plan** | `SDD/<slug>/PLAN.md` — *how* to build it, technical layer | Required |
| **Tasks** | Transient preview → Lattice tasks with `spec_ref` | Required |
| **Implement** | Workers via `agentic-workspace`, batched checkpoints | Per batch/boundary |

## Artifact Structure

```
SDD/
  <feature-slug>/
    SPEC.md    # Functional spec — REQ-numbered, assumptions, acceptance criteria
    PLAN.md    # Technical plan — architecture, rollback, observability, risks
               # No TASKS.md — Lattice is the task source of truth
```

Both files are committed to the repo (Spec-Anchored level). They travel with the code and evolve with it.

## Traceability

The SPEC uses numbered requirements (`REQ-001`, `REQ-002`). Every Lattice task carries a `spec_ref: REQ-###` tag. If the spec changes, you can identify which tasks are stale.

## When to Use SDD

**Use SDD when:**
- The feature spans multiple files or domains
- Business logic isn't obvious to an outside reader
- You're working in a legacy codebase
- Multiple agents will work on the feature in parallel

**Don't use SDD when:**
- It's a small bug fix or config change — use Plan mode instead
- Single-file, clearly-scoped change with no edge cases

## Maturity Levels

| Level | Description |
|-------|-------------|
| **Spec-First** | Write spec, discard after delivery. Eliminates ambiguity for that cycle. |
| **Spec-Anchored** | Spec lives in the repo alongside the code. ← *This skill targets this level* |
| **Spec-as-Source** | Code is regenerated from the spec. (Frontier — not fully here yet) |

## Integration with This Repo's Tooling

- **agentic-workspace** — executes tasks as parallel workers with Lattice tracking
- **ia-exchange-protocol (IAxP)** — governs multi-agent coordination during execution (activated *after* PLAN approval, not during spec writing)
- **ia-bridge-mcp** — use `/peer-opinion:forum` to audit your SPEC or PLAN before approving

## What is Lattice?

Lattice (`lattice-tracker`) is a **local task tracker** — a CLI + MCP + web dashboard. No external service, no account. It stores task state in a `.lattice/` folder at your project root (git-ignored).

Workers (Claude/Codex agents) interact with it through the **Lattice MCP**, which gets registered in Claude Code automatically during install. They never touch `.lattice/` files directly — they use MCP tools (`lattice_show`, `lattice_status`, `lattice_comment`, `lattice_complete`).

### Viewing the board

```bash
lattice dashboard         # opens browser UI
lattice list              # CLI table view
lattice list --status in_progress
lattice show PROJ-1 --full
```

### How it fits in the SDD workflow

```
SDD skill (you + Claude)        Lattice                  agentic-workspace
────────────────────────        ───────────────────      ─────────────────
SPEC + PLAN approved   ──►  lattice create TASK-001
                            lattice create TASK-002
                            lattice link T-001 blocks T-002

Write-scope approved   ──►                           agentic-workspace start
                                                     agentic-workspace spawn claude --task TASK-001
                                                     agentic-workspace spawn codex  --task TASK-002

Workers update state:       ◄── lattice_status in_progress
via Lattice MCP tools       ◄── lattice_comment "progress..."
                            ◄── lattice_complete

Checkpoint reached     ◄──  lattice list --status review
```

The SDD skill handles spec and plan. Lattice owns task lifecycle. `agentic-workspace` handles worker spawning and monitoring.

## Further Reading

- Original article: [@juliandeangeIis on X](https://x.com/juliandeangeIis/status/2033303156340240481)
- `mcps/agentic-workspace/SKILL.md` — task execution and worker spawning
- `ia-exchange-protocol/SKILL.md` — multi-agent governance protocol
