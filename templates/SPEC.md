---
Feature: <!-- feature name -->
Author: <!-- your name -->
Date: <!-- YYYY-MM-DD -->
Version: 1.0
---

# Spec: <!-- Feature Name -->

## Goals

<!-- One paragraph: what this feature does and why it exists. -->

## Non-Goals

<!-- Explicitly what this feature does NOT do. This is as important as the goals. -->
- <!-- ... -->

## Actors

| Actor | Role | Permissions |
|-------|------|-------------|
| <!-- e.g. Admin --> | <!-- ... --> | <!-- Can do X, cannot do Y --> |
| <!-- e.g. End User --> | <!-- ... --> | <!-- ... --> |

## Boundaries & Interfaces

<!-- Which system, service, or API does this touch? Which "backoffice"? Which storage layer? -->
- **Entry point:** <!-- e.g. internal admin panel, not seller-facing portal -->
- **APIs touched:** <!-- ... -->
- **Data stores:** <!-- ... -->
- **External dependencies:** <!-- ... -->

## Assumptions

Work through this checklist before writing requirements. Fill what you know; flag the rest as open questions.

- [ ] **Idempotency** — Can the operation be safely repeated? What happens on duplicate calls?
- [ ] **Authorization** — Who is allowed to do this? What happens if an unauthorized actor tries?
- [ ] **Actors** — Are all user roles and their permissions clearly identified?
- [ ] **Boundaries** — Is the target system (API, UI, service) unambiguous?
- [ ] **Error states** — What are the failure modes? How should each be surfaced?
- [ ] **Data lifecycle** — What happens to data on create/update/delete? Any cascading effects?
- [ ] **Observability** — What needs to be logged, measured, or traced for this feature?
- [ ] **Back-compat** — Does this change break existing behavior for current users?

## Requirements

### Functional

- **REQ-001:** <!-- ... -->
- **REQ-002:** <!-- ... -->
- **REQ-003:** <!-- ... -->

### Non-Functional

- **REQ-NF-001:** <!-- e.g. Response time under 100ms for the primary endpoint -->
- **REQ-NF-002:** <!-- e.g. Operation must be idempotent -->

## Edge Cases

- **EDGE-001:** <!-- What happens when... -->
- **EDGE-002:** <!-- What if... -->

## Acceptance Criteria

```gherkin
# REQ-001
Given <context>
When <action>
Then <outcome>

# REQ-002
Given <context>
When <action>
Then <outcome>
```

## Success Metrics

<!-- How do we know this feature is working correctly in production? -->
- <!-- e.g. Error rate on /api/items/create < 0.1% -->
- <!-- e.g. Zero duplicate-insert incidents in the first 30 days -->

## Decision Log

<!-- Decisions made during spec review. Add entries as: [YYYY-MM-DD] Decision: ... Rationale: ... -->
- <!-- ... -->

## Open Questions

<!-- Questions that need answers before or during implementation. -->
| # | Question | Owner | ETA |
|---|----------|-------|-----|
| 1 | <!-- ... --> | <!-- ... --> | <!-- ... --> |

## Amendment Log

<!-- Record changes made to this spec after implementation began. Format: [YYYY-MM-DD] REQ-### changed: <what changed and why> -->
<!-- ... -->
