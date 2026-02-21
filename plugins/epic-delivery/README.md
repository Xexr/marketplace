# Epic Delivery Plugin

Orchestrate full delivery of a beads epic through polecats, refinery, and integration branches. The crew member acts as dispatcher and monitor; polecats do the implementation work.

## Where This Fits

Epic delivery is the final stage of the **design-to-delivery pipeline**. The earlier stages live in the [gt-toolkit formulas](https://github.com/Xexr/gt-toolkit/tree/main/formulas):

```
gt-toolkit formulas                          Marketplace plugins       Separate step
───────────────────────────────────          ───────────────────       ──────────────
Spec → Plan → Review → Beads → Beads Review → Epic Delivery           → Land to main
                                                                        ↑
                                                                        After QA
```

| Stage | Tool | What it does |
|-------|------|-------------|
| 1-4 | `spec-workflow` formula | Scope questions, brainstorm, interview, multimodal review → produces `spec.md` |
| 5 | `plan-writing` formula | Deep codebase analysis → produces `plan.md` |
| 6 | `plan-review-to-spec` formula | Verifies plan covers spec → produces `plan-review.md` |
| 7 | `beads-creation` formula | Converts plan into beads epic with validated dependencies |
| 8 | `beads-review-to-plan` formula | 3-direction review verifying beads match the plan → produces `beads-review.md` |
| **9** | **`epic-delivery` plugin** | **Delivers the epic through polecats and the refinery** |

Stages 1-8 handle *what to build* and verify it's captured correctly. This plugin handles *getting it built* through Gas Town's polecat infrastructure.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install epic-delivery@xexr-marketplace
```

## When to Use

- Epic exists with leaf tasks ready for implementation
- Want polecat-delegated delivery (not local crew execution)
- Need integration branch isolation before landing to main
- Want automated wave dispatch based on dependency unblocking

## When NOT to Use

- Single task (just `gt sling <id> <rig>` directly)
- No beads epic exists (create one first with `design-to-beads`)
- Want crew to implement directly (use `implementing-beads-epic`)

## The Process

### Phase 1: Setup

Creates or reuses an integration branch and convoy. Gathers all leaf beads (tasks, bugs, features, chores — not epics) and registers them with the convoy for tracking.

### Phase 2: Dispatch

Queries polecat capacity, finds unblocked leaves via `bd ready`, and slings them to the rig. Respects `max_polecats` limits. Dispatches in waves as dependencies resolve.

### Phase 3: Monitor

Hybrid monitoring loop: checks integration branch status (MR merges) and convoy progress at 2-minute intervals. Dispatches newly unblocked work when dependencies land. Escalates to user after 5 consecutive no-change cycles. Hands off to fresh session after 15 cycles.

**Critical:** MR merge to the integration branch is the readiness signal, not bead closure. Polecats close beads on `gt done`, but the code hasn't landed until the refinery merges the MR.

### Phase 4: Validation

When all leaves are closed or deferred:
1. Verify integration branch state
2. Run quality gates (setup → types → lint → build → test)
3. Handle failures via bug-fix sub-epics
4. Lightweight plan-vs-actual summary
5. Offer deep review via `review-implementation` skill
6. Report next steps (QA → land to main)

## Failure Handling

The skill handles refinery re-assignments (merge conflicts), orphaned polecat branches, stale beads, and stuck tasks. Escalation is always dependency-aware — tasks with downstream dependents cannot be skipped.

## Commands

| Command | Purpose |
|---------|---------|
| `/epic-delivery` | Deliver a beads epic through polecats and the refinery |

## License

MIT
