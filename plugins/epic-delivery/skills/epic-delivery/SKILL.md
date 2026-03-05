---
name: epic-delivery
description: Use when delivering a beads epic through polecats and the refinery - sets up integration branch, creates convoy, dispatches waves of leaf tasks, monitors progress, runs quality gates, and validates completeness against the original plan
---

# Epic Delivery

Orchestrate full delivery of a beads epic through polecats, refinery, and integration branches. The crew member acts as dispatcher and monitor; polecats do the implementation work.

**Coexists with `implementing-beads-epic`** (that skill = crew does work locally via subagents; this skill = polecats do work via Gas Town infrastructure).

## When to Use

- Epic exists with leaf tasks ready for implementation
- Want polecat-delegated delivery (not local crew execution)
- Need integration branch isolation before landing to main
- Want automated wave dispatch based on dependency unblocking

## When NOT to Use

- Single task (just `gt sling <id> <rig>` directly)
- No beads epic exists (create one first with `design-to-beads`)
- Want crew to implement directly (use `implementing-beads-epic`)

## Input

**Required:** Epic ID (e.g., `gt-abc`)

The target rig is auto-detected from your current workspace.

### Cross-Rig Projects

Some epics span multiple rigs (e.g., petals + node0 + lora_forge). Leaf beads live in different rig databases with different prefixes (e.g., `pe-`, `no-`, `lf-`). Cross-rig mode is **auto-detected** — if leaf beads have mixed prefixes, the skill switches to cross-rig handling:

- **Local epics:** a local epic is created in each secondary rig so `gt mq integration create` works natively
- **Integration branches:** one per rig via `gt mq integration create` (never manual git branches)
- **Dispatch:** each leaf is slung to the rig that owns it with `--epic <local-epic-id>`
- **Monitoring:** all rigs' integration statuses are checked each cycle
- **Validation:** quality gates run per rig on each integration branch

Single-rig epics (all leaves share one prefix) work exactly as before — no changes needed
---

## Process Overview

```dot
digraph epic_delivery {
    rankdir=TB;

    start [label="Epic ID provided" shape=doublecircle];
    check_existing [label="Existing convoy for this epic?" shape=diamond];
    reuse_convoy [label="Reuse existing convoy" shape=box];
    check_type [label="Verify bead type = epic\n(fix with bd update -t epic)" shape=box];
    setup_branch [label="Create/verify integration branch" shape=box];
    gather_leaves [label="Gather leaf beads (exclude epics)" shape=box];
    create_convoy [label="Create convoy with leaves" shape=box];

    ready_check [label="bd ready --parent <epic>" shape=plaintext];
    any_ready [label="Any unblocked leaves?" shape=diamond];
    sling_wave [label="Sling ready leaves to rig" shape=box];

    monitor [label="Monitor: check integration status\n+ convoy (2-min interval)" shape=box];
    progress [label="Any MRs merged to\nintegration since last check?" shape=diamond];
    check_done [label="All MRs merged\n(none pending)?" shape=diamond];
    stuck [label="5+ no-change cycles?" shape=diamond];
    escalate [label="Escalate to user\n(dependency-aware options)" shape=box];

    validate_branch [label="Verify integration branch\n(git checkout + pull)" shape=box];
    run_gates [label="Run MQ quality gates\n(setup→types→lint→build→test)" shape=box];
    gates_pass [label="All gates pass?" shape=diamond];

    create_bugfix [label="Create bug-fix sub-epic\nFile fixes, add to convoy" shape=box];
    sense_check [label="Lightweight plan-vs-actual\nsummary" shape=box];
    offer_review [label="Offer review-implementation\nskill?" shape=diamond];
    run_review [label="Run review-implementation\n(multi-LLM)" shape=box];
    report [label="Report to user" shape=doublecircle];

    start -> check_existing;
    check_existing -> reuse_convoy [label="yes"];
    check_existing -> check_type [label="no"];
    check_type -> setup_branch;
    reuse_convoy -> ready_check;
    setup_branch -> gather_leaves;
    gather_leaves -> create_convoy;
    create_convoy -> ready_check;

    ready_check -> any_ready;
    any_ready -> sling_wave [label="yes"];
    any_ready -> monitor [label="no (waiting)"];
    sling_wave -> monitor;

    monitor -> progress;
    progress -> check_done [label="yes (new merges)"];
    progress -> stuck [label="no change"];
    check_done -> validate_branch [label="yes, all done"];
    check_done -> ready_check [label="no, check for\nnewly unblocked"];
    stuck -> escalate [label="yes"];
    stuck -> monitor [label="no, keep waiting"];
    escalate -> monitor [label="user says wait"];

    validate_branch -> run_gates;
    run_gates -> gates_pass;
    gates_pass -> sense_check [label="yes"];
    gates_pass -> create_bugfix [label="no"];
    create_bugfix -> ready_check [label="resume loop"];
    sense_check -> offer_review;
    offer_review -> run_review [label="yes"];
    offer_review -> report [label="no, skip"];
    run_review -> report;
}
```

---

## Phase 1: Setup

### 1a. Check for Existing Convoy

If resuming a previous delivery attempt (e.g., after a crash or manual restart), a convoy may already exist:

```bash
# Check for existing convoys tracking this epic's children
gt convoy list --json
```

If a convoy already exists for this epic, **reuse it** — skip to Phase 2. The convoy is the persistent state machine; creating a duplicate would split tracking.

### 1b. Verify Epic Type

`gt mq integration create` requires the bead to be typed as `epic`. Beads may be typed as `task` even if they have children. Check and fix before proceeding:

```bash
# Check the bead's type
bd show <epic-id> --json | python3 -c "import json,sys; d=json.load(sys.stdin)[0]; print(d['issue_type'])"

# If type is NOT 'epic', fix it:
bd update <epic-id> -t epic
```

### 1c. Gather Leaf Beads

Collect all implementable children — exclude epics and sub-epics.

```bash
# List all children of the epic (recursive across all nesting depths)
bd list --parent <epic-id> --all --limit 0 --json
```

**Type-based filter:** Include `task`, `bug`, `feature`, `chore`. Exclude `epic`. The `--parent` flag is recursive, so this works regardless of nesting depth — a task three levels deep under nested sub-epics will appear in the flat result set.

**Cross-sub-epic dependencies are bead-to-bead, not epic-scoped.** If `gt-b2` depends on `gt-a1` (from a different sub-epic), `bd ready` correctly resolves this because dependencies reference bead IDs directly, not their parent epics.

### 1d. Detect Cross-Rig Mode

Extract the bead prefix from each leaf ID (the characters before the first `-` and number, e.g., `pe-` from `pe-k0e.3.1`, `no-` from `no-r2l`). If all leaves share the same prefix, this is a **single-rig project** — proceed normally. If leaves have **mixed prefixes**, this is a **cross-rig project**.

```bash
# Build a rig map from bead prefixes using routes.jsonl
# Each line in routes.jsonl maps: {"prefix": "pe", "path": "/home/ubuntu/gt/petals/mayor/rig"}
cat ~/gt/.beads/routes.jsonl
```

Build a **rig map** grouping leaves by their owning rig:

```
Rig Map:
  petals (pe-): pe-k0e.1.1, pe-k0e.1.2, pe-k0e.1.3, pe-k0e.2.1, ...
  node0 (no-):  no-r2l, no-2yg, no-kwx
  lora_forge (lf-): lf-mx5rb, lf-ouk9x
```

Record this map — it drives all subsequent phases.

**Single-rig shortcut:** If only one rig appears, skip all cross-rig handling. The rest of the skill works unchanged.

### 1e. Create or Verify Integration Branches

**Single-rig:** Create one integration branch as before:

```bash
gt mq integration status <epic-id>
# If none exists:
gt mq integration create <epic-id>
```

**Cross-rig:** Each rig needs its own local epic and its own integration branch. Do NOT create manual git branches — use `gt mq integration create` in every rig.

1. **Create a local epic in each secondary rig:**

```bash
# For each secondary rig (i.e., every rig except the primary where the root epic lives):
cd <secondary-rig-path>
bd create "<root-epic-title>: <rig-name> component" -t epic -p <priority> \
  -d "Local epic for <rig-name> components of <root-epic-title>. Cross-rig root: <root-epic-id> in <primary-rig>."
# Returns: <local-epic-id>
```

2. **Re-parent the secondary rig's tasks under the local epic:**

```bash
# For each task in this rig (identified by prefix in the rig map):
bd dep add <task-id> <local-epic-id> --type parent-child
```

3. **Create integration branches using `gt mq integration create` in every rig:**

```bash
# Primary rig (where the root epic lives):
gt mq integration create <root-epic-id>

# Each secondary rig:
cd <secondary-rig-path>
gt mq integration create <local-epic-id>
```

Record the integration branch name and local epic ID per rig:

```
Integration Branches:
  petals: integration/pe-k0e (primary — root epic pe-k0e, gt mq managed)
  node0:  integration/no-xyz (local epic no-xyz, gt mq managed)
  lora_forge: integration/lf-abc (local epic lf-abc, gt mq managed)
```

The branch name follows each rig's `merge_queue.integration_branch_template` setting (default: `integration/{epic}`).

### 1f. Create Convoy

```bash
# Create convoy with ALL leaf bead IDs across all rigs (NOT epic/sub-epic IDs)
gt convoy create "<epic-title> delivery" <leaf-id-1> <leaf-id-2> ... <leaf-id-n>
```

Record the convoy ID (e.g., `hq-cv-xyz`) — this is your persistent state tracker for the rest of the process.

**Only add leaf beads to the convoy.** Never add epic or sub-epic beads. The convoy tracks implementable work items; epics are organizational containers.

**Exclude already-closed leaves.** Some leaves may already be closed from prior work. Filter these out — adding closed beads to the convoy inflates the total count and confuses progress tracking.

**Cross-rig note:** The convoy lives in HQ and tracks bead IDs regardless of which rig database they live in. Cross-rig bead IDs (e.g., `no-r2l`, `lf-mx5rb`) work natively in convoys.

## Phase 2: Dispatch

### 2a. Query Polecat Capacity

```bash
# Single-rig: check one rig
gt rig config show <rig>

# Cross-rig: check each rig in the rig map
gt rig config show petals    # max_polecats for petals leaves
gt rig config show node0     # max_polecats for node0 leaves
gt rig config show lora_forge # max_polecats for lora_forge leaves
```

**Note:** Each rig enforces `max_polecats` independently. This limit covers ALL polecats in that rig — including ones you slung AND ones the refinery spawns for conflict resolution. Respect each rig's limit.

### 2b. Ensure Daemons Are Running

The daemon manages polecat lifecycle and drives the refinery. Without it, polecats won't be monitored and MRs won't be processed.

```bash
gt daemon status
# If not running:
gt daemon start
```

**Cross-rig note:** There is one daemon for the entire Gas Town workspace. It manages all rigs. You only need to start it once.

### 2c. Find Ready Work

```bash
# Find unblocked leaf tasks under the epic
bd ready --parent <epic-id> --json
```

`bd ready` already excludes hooked, in_progress, blocked, and deferred beads. Only truly claimable work appears.

**Filter results carefully:**

- Exclude epics/sub-epics by type — only sling task, bug, feature, chore
- Verify each result's ID is actually under the target epic — `bd ready --parent` may return broader results than expected
- For Wave 2+: verify each task's dependencies have MRs merged to the integration branch (check `gt mq integration status`), not just that the dependency beads are closed

**Cross-rig note:** `bd ready` resolves cross-rig dependencies natively. A `no-r2l` task blocked by `pe-k0e.1.3` will only appear in ready results after `pe-k0e.1.3` is closed.

### 2d. Sling Ready Leaves

**Batch sling is currently broken.** Sling each leaf individually with `--no-convoy` (the convoy was already created in Phase 1):

**Single-rig:**

```bash
gt sling <leaf-1> <rig> --no-convoy
gt sling <leaf-2> <rig> --no-convoy
```

**Cross-rig:** Sling each leaf to the rig that owns it (from the rig map in Phase 1d). Use `--epic <local-epic-id>` so the sling targets the correct rig's integration branch:

```bash
# Leaves are slung to their owning rig with that rig's local epic
gt sling pe-k0e.1.1 petals --no-convoy --epic <petals-root-epic-id>       # primary rig uses root epic
gt sling pe-k0e.1.2 petals --no-convoy --epic <petals-root-epic-id>
gt sling no-r2l node0 --no-convoy --epic <node0-local-epic-id>             # secondary rig uses LOCAL epic
gt sling lf-mx5rb lora_forge --no-convoy --epic <lora-forge-local-epic-id>  # secondary rig uses LOCAL epic
```

**Critical: match bead prefix to rig AND use the correct epic.** A `no-` bead MUST be slung to node0 with `--epic <node0-local-epic-id>` (not the root epic). Slinging to the wrong rig causes the polecat to not find the bead. Using the wrong epic causes the MR to target the wrong integration branch.

`gt sling` auto-targets the integration branch when the parent epic has one. Each leaf gets its own polecat. Respect each rig's `max_polecats` — don't sling more than a rig allows.

**Report to user:**

- Single-rig: "Slung N tasks to <rig>: <list>"
- Cross-rig: "Slung N tasks across K rigs: <rig1>: <list>, <rig2>: <list>, ..."

## Phase 3: Monitor Loop

**Hybrid pattern:** 15 check cycles at 2-minute intervals, then handoff to fresh session.

### Critical: MR Merge Is the Readiness Signal

**Do NOT use bead closure as the signal to dispatch dependent tasks.** Polecats close their beads when they submit via `gt done`, but the code hasn't landed on the integration branch yet. The refinery merges MRs to the integration branch — only after that merge is the code available to dependents.

**Source of truth for "work has landed":** `gt mq integration status <epic-id>` → count of merged MRs and commits ahead of main.

**Commits ahead ≠ MRs merged.** The commit count is always higher because: polecats may make multiple commits per task, the refinery creates merge commits, and the refinery may add its own fix commits. Track the **merged MR count**, not the commit count.

Sequence of events for each task:

1. Polecat completes work → `gt done` → MR bead created, task bead may close
2. Refinery picks up MR → reviews → merges to integration branch
3. **Only now** is it safe to sling tasks that depend on this work

### The Check Cycle

Each cycle:

1. **Check integration branch status (primary signal):**

   **Single-rig:**

   ```bash
   gt mq integration status <epic-id>
   ```

   **Cross-rig:** Check each rig's integration branch using its local epic:

   ```bash
   # For each rig in the rig map:
   cd <rig-path> && gt mq integration status <local-epic-id>
   ```

   Aggregate merged MR counts across all rigs. Track per-rig progress.

2. **Check convoy status (secondary, for overall progress):**

   ```bash
   gt convoy status <convoy-id>
   ```

3. **If new MRs merged since last check:**
   - Report: "Landed on integration branch: <task-title> on <rig> (N/total MRs merged)"
   - Check completion (see below)
   - If not all done: run `bd ready --parent <epic-id>` to find newly unblocked work
   - Cross-check: for each newly ready task, verify its dependencies' MRs have merged (check integration status merged list)
   - If ready AND dependencies landed, sling it to the **correct rig** (Phase 2d)
   - Reset no-change counter to 0

4. **If no new MRs merged (across all rigs):**
   - Increment no-change counter
   - If counter exceeds patience threshold (5 consecutive no-change cycles = ~10 min): escalate to user

5. **Check for refinery re-assignments** (see Failure Handling below)

### Completion Check

**"All done" means:** every leaf task's MR has been merged to its rig's integration branch. Do NOT rely on bead status alone.

**Single-rig:**

```bash
# Primary: check integration branch for all expected MRs merged
gt mq integration status <epic-id>

# Secondary: check for any leaves still open/in_progress (exclude epics by type)
bd list --parent <epic-id> --status open --limit 0 --json
bd list --parent <epic-id> --status in_progress --limit 0 --json
```

**Cross-rig:** Check each rig's integration status using its local epic:

```bash
# For each rig in the rig map:
cd <rig-path> && gt mq integration status <local-epic-id>
```

**Both conditions must be true:** all rigs' integration statuses show their MRs merged (no pending), AND no open/in_progress leaf tasks remain. If pending MRs still exist in any rig, wait — that rig's refinery hasn't finished processing.

If deferred tasks exist, note them — they'll appear in the completion report. Use `gt convoy close <convoy-id> --force` at the end since the convoy won't auto-close with deferred items.

### Timing

```
Cycle 1  → check → [sleep 2 min]
Cycle 2  → check → [sleep 2 min]
...
Cycle 15 → check → handoff (if work still in flight)
```

**Sleep implementation:**

```bash
sleep 120  # 2 minutes between checks
```

### Handoff During Monitoring

When cycling after 10 checks:

```bash
gt handoff -s "Epic delivery: <epic-id> monitoring" -m "
IMPORTANT: Run Skill('epic-delivery') FIRST before doing anything else.

Epic: <epic-id>
Convoy: <convoy-id>
Cross-rig: <yes/no>
Rig map: <prefix>:<rig-name> (e.g., pe:petals, no:node0, lf:lora_forge)
Local epics: <rig-name>: <local-epic-id> (e.g., node0: no-xyz, lora_forge: lf-abc)
Integration branches:
  <rig1>: <branch-name> (N merged, M pending)
  <rig2>: <branch-name> (N merged, M pending)
Progress: N/total MRs merged across all rigs, M deferred, K in-flight
Pending MRs: <list of pending MR bead IDs with rig>
Open leaves: <list of open leaf bead IDs with rig>
Active polecats: <count per rig>
Refinery re-assignments: <any noted>
Last slung: <timestamp>
No-change counter: <value>
Phase: MONITORING
Next action: reload skill, then resume check cycle
"
```

### Resume After Handoff

**MANDATORY FIRST STEP: Reload this skill.** Run `Skill("epic-delivery")` before doing ANYTHING else. The skill defines the full process including phases you may not remember from the handoff message. Skipping this step causes phases to be missed (e.g., the deep review offer in Phase 4e, the landing boundary in Phase 4f).

Then detect phase from state:

1. Read convoy status: `gt convoy status <convoy-id>` (convoy ID is in the handoff mail)
2. Check epic progress: `bd list --parent <epic-id> --tree`
3. If all leaves closed/deferred → go to Phase 4 (Validation)
4. If leaves still open → resume Phase 3 (Monitor Loop)
5. Run `bd ready --parent <epic-id>` — sling anything newly unblocked

**The convoy IS the persistent state machine.** No separate state file needed.

---

## Failure Handling

### Refinery Re-assigns Work (Normal)

The refinery may encounter a merge conflict and spawn a fresh polecat to re-implement. This is normal.

**How to detect:** Check `gt convoy status <convoy-id>` — the convoy status shows tracked issues and their current state. Also check merge request beads:

```bash
# Look for MR beads related to the task
bd list --parent <epic-id> --type merge-request --json
```

A re-assigned task will have an MR bead with conflict status, and a new polecat session will appear. The original task bead remains `in_progress` — it will be closed when the fresh polecat succeeds.

**Action:** Inform the user and wait.
> "Refinery detected conflict on <task-title>. Re-assigned to fresh polecat. Waiting for re-implementation."

The refinery-spawned polecat counts against the rig's `max_polecats` limit. If the rig is at capacity, `gt sling` will wait for a slot. You do not need to manage this — the infrastructure handles it.

### Orphaned Polecat Work (Code Exists, No MR)

A polecat may exit without calling `gt done` — code is pushed to a branch but no MR was submitted. The deacon may respawn the task to a fresh polecat, wasting the completed work.

**How to detect:** Polecat is gone (`gt polecat list <rig>` shows no polecat for the task), but the polecat branch exists with commits:

```bash
# Check if branch exists with work
git branch -r | grep "<task-id>"
git log origin/polecat/<name>/<task-id>@<session> --oneline -5

# Check diff against integration branch
git diff origin/<integration-branch>...origin/polecat/<name>/<task-id>@<session> --stat
```

**Recovery:** If the code looks complete, manually submit the MR:

```bash
gt mq submit --branch polecat/<name>/<task-id>@<session> --issue <task-id> --epic <epic-id> --no-cleanup
```

This puts the branch in the merge queue for the refinery to process. The `--no-cleanup` flag preserves the branch.

### Stale Beads After MR Merge

MRs may merge to the integration branch but the task bead remains open — often because molecule wisps block closure. This prevents dependent tasks from becoming ready.

**How to detect:** `gt mq integration status` shows merged MR, but `bd show <task-id>` shows open/in_progress.

**Recovery:** Force-close the bead:

```bash
bd close <task-id> --force
```

The `--force` flag overrides open molecule wisp blockers. Only use this when you've confirmed the MR has merged.

### Truly Stuck (Escalate)

**Detection:** No progress on a task for an extended period. Signs:

- Polecat session is gone (crashed/killed) but bead still open
- Bead in `in_progress` but no active polecat working on it
- MR submitted but refinery not processing it

**Action:** Escalate immediately. Do NOT re-sling or kill polecats. Check dependencies before presenting options:

```bash
# Check what depends on the stuck task (direction=up shows dependents)
bd dep tree <bead-id> --direction=up
```

**If the stuck task HAS dependents** (other tasks are blocked by it):

```
AskUserQuestion:
  "Task <task-title> (<bead-id>) appears stuck:
   - <description of symptoms>
   - WARNING: <N> downstream tasks depend on this. Skipping is not an option.
   Should I:
   1. Wait longer (give it more time)
   2. Re-sling to a fresh polecat
   3. I'll fix this manually — pause the delivery until I say to continue
   4. Abort the delivery entirely"
```

**If the stuck task has NO dependents** (nothing downstream depends on it):

```
AskUserQuestion:
  "Task <task-title> (<bead-id>) appears stuck:
   - <description of symptoms>
   - No downstream dependencies — safe to skip.
   Should I:
   1. Wait longer (give it more time)
   2. Re-sling to a fresh polecat
   3. Skip this task (defer it, continue with others)
   4. I'll fix this manually — pause until I say to continue"
```

**If user says "skip" (only offered for tasks with no dependents):**

```bash
bd update <bead-id> --status deferred
```

**If user says "I'll fix manually":** Pause the monitor loop. Wait for the user to say "continue" or "resume". Then re-enter the check cycle — the user's manual fix should have closed or unblocked the task.

### Never Do These

- **Never kill a polecat** — the witness manages polecat lifecycle
- **Never re-sling a hooked/in_progress bead** — `bd ready` already excludes these
- **Never retry without user approval** when something has genuinely failed

---

## Phase 4: Validation

Triggered when all convoy leaves are closed or deferred.

### 4a. Verify Integration Branches

**Single-rig:**

```bash
gt mq integration status <epic-id>
```

**Cross-rig:** Verify each rig's integration branch using its local epic:

```bash
# For each rig in the rig map:
cd <rig-path> && gt mq integration status <local-epic-id>
```

Confirm all expected work has landed on every rig. If any MRs are still pending in any rig's refinery queue, wait for them before proceeding.

### 4b. Run Quality Gates

**Always check out and pull the integration branch before running gates.**

**Single-rig:**

```bash
git checkout <integration-branch>
git pull
gt rig settings show <rig>
```

**Cross-rig:** Run quality gates **per rig** on each rig's integration branch:

```bash
# For each rig in the rig map:
cd <rig-path>
git checkout integration/<local-epic-id>
git pull
gt rig settings show <rig-name>
# Run configured gates for this rig
```

Read each rig's MQ settings to determine which gates are configured:

Run each configured gate **in order** per rig. Skip any that are empty/unconfigured:

| Order | Setting | Run if |
|-------|---------|--------|
| 1 | `setup_command` | Non-empty |
| 2 | `typecheck_command` | Non-empty |
| 3 | `lint_command` | Non-empty |
| 4 | `build_command` | Non-empty |
| 5 | `test_command` | Non-empty |

**Fail fast:** If any gate fails on any rig, stop and proceed to 4c. Report which rig failed.

### 4c. Handle Quality Gate Failures

If any gate fails:

1. **Report failures to user** with full output
2. **Recommend** creating a bug-fix sub-epic:

```
AskUserQuestion:
  "Quality gate '<gate-name>' failed:
   <error summary>

   Recommended: Create bug-fix sub-epic with individual fix tasks, add to convoy, and resume delivery.

   Options:
   1. Create bug-fix sub-epic (Recommended)
   2. Let me fix manually first
   3. Skip and report as-is"
```

1. If user approves bug-fix sub-epic:

```bash
# Create sub-epic under the main epic
bd create "<epic-title>: bug fixes" -t epic --parent <epic-id>

# File individual fix beads (one per issue identified)
bd create "Fix: <description of issue 1>" -t bug --parent <bugfix-epic-id>
bd create "Fix: <description of issue 2>" -t bug --parent <bugfix-epic-id>

# Add new leaves to existing convoy
gt convoy add <convoy-id> <fix-1-id> <fix-2-id> ...
```

1. **Resume from Phase 2.** The fix beads target the same integration branch. After fix polecats complete and the refinery merges their work, **re-run ALL quality gates from scratch** (not just the failed one) — earlier gates may have been affected by the fixes.

**Remember to `git checkout <integration-branch> && git pull`** before re-running gates to pick up the new commits from fix polecats.

### 4d. Lightweight Plan-vs-Actual Summary

When all quality gates pass:

1. **Find the plan document.** Check for a plan file first — this is the authoritative source of requirements:

   ```bash
   # Check for plan.md in the plans folder (common locations)
   ls plans/ .beads/plans/ docs/plans/ 2>/dev/null
   # Look for files matching the epic name/ID
   ```

   If a plan.md exists, read it and use it as the primary reference for acceptance criteria.

2. **Read the integration branch diff** vs main:

   **Single-rig:**

   ```bash
   git diff main...<integration-branch> --stat
   ```

   **Cross-rig:** Read diff per rig:

   ```bash
   # For each rig:
   cd <rig-path> && git diff main...integration/<local-epic-id> --stat
   ```

3. **Produce a summary mapping** each acceptance criterion to the task that delivered it:

```
Epic <epic-id> delivery complete.

Convoy: <convoy-id> — N leaves closed, M deferred (skipped)
Rigs involved: <list of rigs> (or "single-rig" if not cross-rig)
Integration branches: <branch per rig> — all quality gates pass

## Plan vs Actual
- [criteria 1]: Met (implemented in <task-id> on <rig>)
- [criteria 2]: Met (implemented in <task-id> on <rig>)
- [criteria 3]: Partial — <explanation of gap>

## Per-Rig Summary (cross-rig only)
- petals: N tasks, M MRs merged, K files changed
- node0: N tasks, M MRs merged, K files changed
- lora_forge: N tasks, M MRs merged, K files changed

## Skipped Tasks (if any)
- <bead-id>: <title> — deferred (reason: <why it was skipped>)

## Notes
- <any important observations, e.g., refinery re-assignments, bug-fix rounds, cross-rig dependency issues>
```

### 4e. Offer Deep Review

After the lightweight summary, ask the user if they want a comprehensive multi-LLM review:

```
AskUserQuestion:
  "Lightweight plan-vs-actual summary is above. Would you like a deeper review
   using the review-implementation skill? This runs a multi-LLM analysis
   (Opus, GPT, Gemini) comparing the implementation against the original spec."

   Options:
   1. Run review-implementation (thorough multi-LLM review)
   2. Skip — the summary is sufficient
```

If user selects option 1:

```
Skill(
  skill="review-implementation",
  args="<epic-id>"
)
```

### 4f. Report and Next Steps

```
## Next Steps
The integration branch(es) are validated. Typical next steps:

Single-rig:
1. QA run (separate skill/process — test the integration branch before landing)
2. Land to main: gt mq integration land <epic-id> (after QA passes)

Cross-rig:
1. QA run per rig (test each rig's integration branch)
2. Land to main per rig: cd <rig-path> && gt mq integration land <epic-id>
3. Coordinate landing order if rigs depend on each other at runtime
```

**This skill ends here.** Landing to main happens after QA, not before. The integration branch is the staging area; landing is a separate deliberate act.

## Quick Reference

| Phase | Key Command | What Happens |
|-------|-------------|--------------|
| Setup | `bd show <epic> --json` | Verify epic type before branch creation |
| Setup | `bd list --parent <epic> --all --limit 0 --json` | Gather leaves, detect cross-rig from prefixes |
| Setup | `cat ~/gt/.beads/routes.jsonl` | Map bead prefixes to rig paths |
| Setup | `gt mq integration create <epic>` | Creates integration branch (per rig for cross-rig) |
| Setup | `gt convoy create "<name>" <ids...>` | Creates convoy tracker |
| Dispatch | `bd ready --parent <epic>` | Finds unblocked leaves (cross-rig deps resolved natively) |
| Dispatch | `gt sling <id> <rig> --no-convoy` | Sling leaf to its **owning rig** (match prefix to rig) |
| Monitor | `gt mq integration status <epic>` | **Primary signal:** checks merged MRs (per rig for cross-rig) |
| Monitor | `gt convoy status <convoy-id>` | Secondary: overall progress across all rigs |
| Failure | `bd list --type merge-request --parent <epic>` | Check for conflict re-assignments |
| Failure | `gt mq submit --branch <branch> --issue <id> --epic <epic> --no-cleanup` | Rescue orphaned polecat branch |
| Failure | `bd close <id> --force` | Force-close stale bead (after MR merge confirmed) |
| Skip | `bd update <id> --status deferred` | Skip a stuck task |
| Validate | `gt mq integration status <epic>` | Verifies branch state (per rig for cross-rig) |
| Validate | `gt rig settings show <rig>` | Gets quality gate commands (per rig) |
| Review | `Skill("review-implementation", "<epic>")` | Multi-LLM deep review |
| Complete | `gt mq integration land <epic>` | Merges integration to main (per rig for cross-rig) |
| Close | `gt convoy close <convoy-id> --force` | Close convoy (if deferred tasks exist) |

## Common Mistakes

| Mistake | Correction |
|---------|------------|
| Slinging dependents on bead closure | **Most critical mistake.** Bead closure ≠ code landed. Wait for MR merge to integration branch (`gt mq integration status`) |
| Not checking epic type before branch creation | `gt mq integration create` requires type=epic. Check with `bd show --json`, fix with `bd update -t epic` |
| Slinging epics/sub-epics | Filter by type — only sling task, bug, feature, chore |
| Re-slinging hooked beads | `bd ready` already excludes these — trust it |
| Killing polecats when slow | Never kill polecats — escalate to user instead |
| Polling too aggressively | 2-minute intervals minimum between checks |
| No convoy created | Always create convoy — it's your state machine |
| Duplicate convoy on resume | Check for existing convoy before creating a new one |
| Forgetting max_polecats | Query `gt rig config show <rig>` and respect the limit |
| Skipping quality gates | Run ALL configured gates before declaring success |
| Running gates on stale code | Always `git checkout <branch> && git pull` before gates |
| Only re-running failed gate | After bug fixes, re-run ALL gates from scratch |
| Bug fixes on new branch | Bug fixes go to same integration branch via same convoy |
| Skipping task with dependents | NEVER skip tasks that have downstream dependents — offer re-sling or manual fix instead |
| Skipped task blocks convoy close | Use `bd update --status deferred` + `gt convoy close --force` (only for no-dependent tasks) |
| Not comparing plan vs actual | Always do lightweight sense check before reporting |
| Not loading skill on resume | After handoff, first action must be `Skill("epic-delivery")` to reload the process |
| Confusing commits ahead with MRs merged | Commits ahead is always higher (merge commits, multi-commit branches, refinery fixes). Track merged MR count |
| Ignoring orphaned polecat branches | If polecat exits without `gt done`, check for branch with code. Manually submit via `gt mq submit --branch ... --issue ... --epic ... --no-cleanup` |
| Stale beads blocking dependents | MR merged but bead still open? Force-close with `bd close <id> --force` |
| Creating manual git branches for cross-rig | NEVER create manual branches. Create a local epic per secondary rig and use `gt mq integration create` in every rig |
| Slinging cross-rig bead to wrong rig | Match bead prefix to rig via routes.jsonl. A `no-` bead MUST go to node0, not petals |
| Using root epic ID in secondary rigs | Secondary rigs need their own local epic. Use `--epic <local-epic-id>` when slinging to secondary rigs |
| Single integration branch for cross-rig | Each rig needs its own local epic + integration branch — each rig's refinery merges independently |
| Ignoring cross-rig mode detection | Always check leaf prefixes after gathering. Mixed prefixes = cross-rig = different handling |
| Running gates on only one rig | Cross-rig: run quality gates per rig on each rig's integration branch |
| Landing one rig without coordinating | If rigs depend on each other at runtime, coordinate landing order |
