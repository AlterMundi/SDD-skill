# SDD — Spec-Driven Development

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A methodology and agent skill for building features with AI coding agents without ambiguity.

## Why SDD?

When you give an agent a vague prompt, it guesses. It guesses which backoffice, which API contract, which auth model, which error handling strategy. Each guess is a silent decision. Some are right. Some are wrong. As complexity grows, so does the gap between what you meant and what the agent built.

SDD solves this by defining *what* to build and *how* to build it before a single line of code is written.

> "Vibe coding builds demos and MVPs. Spec-Driven Development builds production systems."
>
> — [@juliandeangeIis](https://x.com/juliandeangeIis/status/2033303156340240481)

## Installation & Updates

### Claude Code

Install the skill as a symlink into Claude Code's skills directory:

```bash
ln -sfn /path/to/SDD-skill ~/.claude/skills/sdd
```

Because it's a symlink, **no reinstall is needed** — a `git pull` on this repo is all you need to get the latest version. The next `/sdd` run will use the updated skill automatically.

### Cursor, Codex, or any other agent

The skill is a plain markdown file. Point your agent at it directly:

- **Cursor**: add a reference to `SKILL.md` in your `.cursor/rules/` or paste the content into a project rule.
- **Codex**: pass it via `--instructions` or reference it in `.codex/instructions.md`.
- **Any agent that can read files**: instruct the agent to read `SKILL.md` and follow the SDD workflow.

No special installation needed — the methodology is self-contained in the file.

## Prerequisites

### Core (required for Phases 1–3)

No setup required. SPEC, PLAN, and task preview work with any agent that can read and write markdown files.

### Lattice (required for Phase 4 parallel execution)

Lattice is a local task tracker (CLI + MCP + web dashboard). No external service, no account.

```bash
# If uv is not installed:
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env

# Install Lattice
uv tool install --force 'lattice-tracker[mcp]'
```

**Claude Code only:** also register the Lattice MCP so the agent can call Lattice tools directly. The MCP server is bundled with `lattice-tracker[mcp]` — add it to your Claude Code MCP settings (e.g. in `.claude/settings.json`):
```json
{
  "mcpServers": {
    "lattice": {
      "command": "lattice",
      "args": ["mcp"]
    }
  }
}
```
Then restart your agent environment so the MCP is picked up.

If Lattice is not installed, the skill detects it at startup and offers to either stop for installation or continue in sequential mode (no parallel workers).

### ia-bridge (optional — Phase 2.5 peer audit)

[ia-bridge-mcp](https://github.com/AlterMundi/ia-bridge-mcp) enables a structured peer audit of your PLAN by a second agent (Codex or another Claude instance) before task decomposition. Recommended for risky or complex changes — it can surface import/API contract gaps and other issues the primary agent missed.

**Claude Code:** install the MCP server following the ia-bridge-mcp repo instructions.
**Other agents:** use the `ia-bridge` CLI directly.

The audit step (Phase 2.5) is optional and skippable — the skill will prompt you.

## How It Works

```
/sdd "feature description"          ← Claude Code invocation
# or: ask your agent to follow SKILL.md for other environments
   │
   ├─ Clarifying questions → surface hidden assumptions
   │
   ├─ SDD/<feature-slug>/SPEC.md    ← PAUSE: human approves
   │
   ├─ Codebase exploration
   ├─ SDD/<feature-slug>/PLAN.md    ← PAUSE: human approves
   │
   ├─ [optional] ia-bridge peer audit of PLAN
   │                                ← PAUSE: human approves any plan amendments
   │
   ├─ Execution model decision (parallel workers vs sequential)
   │
   ├─ Transient task preview (shown in chat)
   │   └─ lattice create for each task with spec_ref backlinks
   │                                ← PAUSE: human approves
   │
   └─ Workers dispatched → batched execution checkpoints
```

## The Five Phases

| Phase | Output | Human gate |
|-------|--------|------------|
| **1 · Spec** | `SDD/<slug>/SPEC.md` — *what* to build, functional layer only | Required |
| **2 · Plan** | `SDD/<slug>/PLAN.md` — *how* to build it, technical layer | Required |
| **2.5 · Audit** | ia-bridge peer review of PLAN, findings resolved | Optional |
| **3 · Tasks** | Transient preview → Lattice tasks with `spec_ref` and cold-agent context | Required |
| **4 · Implement** | Workers via worktree isolation, batched checkpoints | Per batch/boundary |

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

- **agentic-workspace** — executes tasks as parallel workers with Lattice tracking (requires tmux + interactive terminal)
- **ia-exchange-protocol (IAxP)** — governs multi-agent coordination during execution (activated *after* PLAN approval, not during spec writing)
- **ia-bridge-mcp** ([repo](https://github.com/AlterMundi/ia-bridge-mcp)) — peer audit of SPEC or PLAN by a second agent before task decomposition (Phase 2.5). Use `/peer-opinion:forum` in Claude Code or the `ia-bridge` CLI in other environments.

## What is Lattice?

Lattice (`lattice-tracker`) is a **local task tracker** — a CLI + MCP + web dashboard. No external service, no account. It stores task state in a `.lattice/` folder at your project root (git-ignored).

Workers (Claude/Codex agents) interact with it through the **Lattice MCP** (Claude Code) or directly via the **Lattice CLI** (any environment). They never touch `.lattice/` files directly — they use the API (`lattice show`, `lattice status`, `lattice comment`, `lattice complete`).

### Viewing the board

```bash
lattice dashboard         # opens browser UI
lattice list              # CLI table view
lattice list --status in_progress
lattice show PROJ-1 --full
```

### How it fits in the SDD workflow

```
SDD skill (you + agent)         Lattice                  Workers
────────────────────────        ───────────────────      ─────────────────────────────
SPEC + PLAN approved   ──►  lattice create TASK-001
                            lattice create TASK-002
                            lattice link T-001 blocks T-002

Write-scope approved   ──►                           Agent(isolation="worktree") [Claude Code]
                                                     agentic-workspace spawn     [tmux terminal]
                                                     new agent session + worktree [Cursor/Codex]

Workers update state:       ◄── lattice status in_progress
via CLI or MCP tools        ◄── lattice comment "progress..."
                            ◄── lattice complete

Checkpoint reached     ◄──  lattice list --status review
```

## Further Reading

- Original article: [@juliandeangeIis on X](https://x.com/juliandeangeIis/status/2033303156340240481)
- [AlterMundi/Skills](https://github.com/AlterMundi/Skills) — the agent harness this skill was built in, including `agentic-workspace` and `ia-exchange-protocol`
- [AlterMundi/ia-bridge-mcp](https://github.com/AlterMundi/ia-bridge-mcp) — peer audit tool used in Phase 2.5
