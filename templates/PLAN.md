---
Feature: <!-- feature name -->
Spec: SDD/<!-- feature-slug -->/SPEC.md
Author: <!-- your name -->
Date: <!-- YYYY-MM-DD -->
---

# Plan: <!-- Feature Name -->

## Architecture Overview

<!-- High-level description of the technical approach. Why this approach over alternatives? -->

## Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| <!-- e.g. Auth strategy --> | <!-- JWT --> | <!-- Matches existing pattern in `src/auth/jwt.ts:42` --> |
| <!-- ... --> | <!-- ... --> | <!-- ... --> |

## Existing Patterns to Follow

<!-- Point explicitly to file:line references the agent must replicate. -->
- `<!-- file:line -->` — <!-- what pattern to replicate -->
- `<!-- file:line -->` — <!-- what pattern to replicate -->

## File Change Manifest

<!-- Every file that will be created or modified. No surprises. -->

| File | Change | REQ refs |
|------|--------|----------|
| `<!-- path/to/file.ts -->` | <!-- create / modify --> | REQ-001 |
| `<!-- ... -->` | <!-- ... --> | <!-- ... --> |

## Data Models

```
<!-- Entity or schema changes. Show before/after if modifying existing. -->
```

## API Contracts

```
<!-- Endpoint definitions, request/response shapes, status codes. -->
```

## Dependencies & Migration

<!-- New packages, schema migrations, config changes. -->
- **New deps:** <!-- ... -->
- **Migrations:** <!-- ... -->
- **Back-compat:** <!-- Does this break existing callers? How is it handled? -->

## Failure Modes & Rollback

<!-- How does this fail? How do we roll it back if it does? -->
| Failure mode | Impact | Rollback step |
|--------------|--------|---------------|
| <!-- e.g. DB timeout on insert --> | <!-- ... --> | <!-- ... --> |
| <!-- ... --> | <!-- ... --> | <!-- ... --> |

**Rollback procedure:**
1. <!-- Step 1 -->
2. <!-- Step 2 -->

## Security & Authorization

<!-- Auth rules, input validation, data exposure risks. -->
- <!-- e.g. Only users with role=admin may call POST /api/items -->
- <!-- e.g. Validate and sanitize all user-supplied input before DB write -->

## Observability

<!-- What to log, measure, and trace. Don't leave this blank. -->
- **Logs:** <!-- e.g. Log item_id + actor on every create/update/delete -->
- **Metrics:** <!-- e.g. Counter: items_created_total{source="backoffice"} -->
- **Traces:** <!-- e.g. Span wrapping the DB write transaction -->
- **Alerts:** <!-- e.g. Alert if error rate > 1% over 5m window -->

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| <!-- e.g. Race condition on concurrent inserts --> | Medium | High | <!-- Use DB-level unique constraint + retry logic --> |
| <!-- ... --> | <!-- ... --> | <!-- ... --> | <!-- ... --> |

## Testing Strategy

- **Unit:** <!-- What to unit test. Coverage target. -->
- **Integration:** <!-- What to integration test. Real DB or service? -->
- **E2E:** <!-- If applicable. -->
- **Manual verification:** <!-- Steps to verify the feature works end-to-end before merge. -->

## Performance Constraints

- <!-- e.g. Primary endpoint must respond in < 100ms at p99 -->
- <!-- e.g. Batch operation must handle up to 1000 items without timeout -->

## MCPs / Tools

<!-- Which MCPs or tools should workers use, and for what. -->
- <!-- e.g. `mcp__db__query` for read-only DB inspection during implementation -->

## Out of Scope (Technical)

<!-- What the agent must NOT build or change in this plan. -->
- <!-- ... -->
