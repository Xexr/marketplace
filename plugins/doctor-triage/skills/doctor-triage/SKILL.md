---
name: doctor-triage
description: Use when diagnosing workspace or tooling health issues, fixing doctor warnings, or when gt/bd tools feel misconfigured. Triggers on "run doctor", "fix doctor", "doctor triage", "health check", "workspace broken".
---

# Doctor Triage

## Overview

Run diagnostic checks on a Gas Town or beads workspace, interpret each finding, fix what's fixable, and surface unfixable issues that may be upstream bugs worth filing as beads.

## When to Use

- User asks to run or fix doctor warnings
- Something in the workspace feels broken or misconfigured
- After upgrades, upstream syncs, or rig additions
- Periodic health checks

## When NOT to Use

- Setting up a new rig from scratch (use `gastown-rig-setup`)
- Debugging a specific test failure (use `systematic-debugging`)

## Step 0: Pre-flight Checks

Verify the required tools are available before proceeding:

```bash
gt --version 2>/dev/null
bd --version 2>/dev/null
```

- If only `gt` is available → skip beads diagnostics, only offer "Gas Town (gt doctor)"
- If only `bd` is available → skip Gas Town diagnostics, only offer "Beads (bd doctor)"
- If neither is available → inform the user and exit

## Step 1: Ask Which Doctor

Use AskUserQuestion:

```
"Which workspace do you want to diagnose?"
Options:
  - "Gas Town (gt doctor)" — Town-wide: clones, daemon, Dolt, patrols, sessions
  - "Beads (bd doctor)" — Repo-level: database, schema, hooks, dependencies
  - "Both" — Run gt doctor first, then bd doctor
```

## Step 2: Run Diagnostics

### Gas Town

```bash
gt doctor -v
```

Key flags if issues found:
- `gt doctor --fix` — auto-fix fixable issues
- `gt doctor --rig <name>` — check specific rig only
- `gt doctor --fix --restart-sessions` — also restart stale patrol sessions

### Beads

Run from the relevant repo root (or `mayor/rig/` for Gas Town rigs):

```bash
bd doctor -v
```

Key flags if issues found:
- `bd doctor --fix` — auto-fix with confirmation
- `bd doctor --deep` — full graph integrity validation
- `bd doctor --server` — Dolt server connectivity checks
- `bd doctor --check=validate --fix` — data-integrity repairs

## Step 3: Interpret and Triage

Parse each warning/error into one of three categories:

| Category | Action |
|----------|--------|
| **Auto-fixable** | Run `--fix` and verify resolution |
| **Manually fixable** | Investigate root cause, apply targeted fix, verify |
| **Unfixable / upstream bug** | Explain to user, collect for Step 5 |

### Triage Guidelines

**Do NOT blindly loop `--fix`.** If `--fix` didn't resolve an issue on the first pass, investigate manually. Looping `--fix` can cycle without progress.

**Do NOT restart the daemon** unless the user asks or the diagnostic specifically recommends it.

**Known false positives to skip:**
- `bd doctor` reports "No dolt database found" in server mode — this checks for local `.beads/dolt/` which doesn't exist with a centralized Dolt server. Verify with `bd list` instead.

## Step 4: Fix What You Can

For each auto-fixable or manually fixable issue:

1. Explain what the issue is and what the fix does
2. Apply the fix
3. Re-run the relevant doctor command to verify resolution
4. Report result to user

**After all fixes, run a final clean check:**

```bash
# Gas Town
gt doctor -v

# Beads
bd doctor -v
```

## Step 5: Reflect and Surface Upstream Bugs

**This step is MANDATORY after every doctor run, even if all checks pass or all issues were fixed.** Do not skip it.

### 5a. Reflect on the Full Run

After all fixes are applied and the final clean check is done, review the entire session:

1. **Issues that `--fix` resolved** — Were any of these symptoms of deeper bugs? (e.g., fix worked but required workarounds, fix needed multiple passes, fix hint was wrong)
2. **Issues fixed manually** — Did the manual fix reveal a gap in `--fix` automation?
3. **Issues that `--fix` failed to resolve** — Why? Silent failure? Missing capability?
4. **Workarounds applied** — Any workaround suggests the tool should handle it natively
5. **Misleading or incorrect doctor output** — Wrong hints, stale data sources, missing error messages

Build a list of candidate upstream bugs. Include anything that is NOT a configuration problem — i.e., something that would affect any user of gt/bd, not just this workspace.

### 5b. Offer to Capture Upstream Issues

If NO candidates were identified, briefly state: "No upstream bugs identified — all issues were configuration-related." and skip to the end.

If candidates were identified, explain what was found and offer an escape hatch:

Use AskUserQuestion:

```
"I identified [N] potential upstream gt/bd issues during this doctor run — things that would affect any user, not just this workspace. Would you like to capture beads documenting these?"
Options:
  - "Yes — show me what you found" — Review and selectively file beads
  - "No — just fix my workspace" — Skip bug filing entirely
```

If the user declines, stop here. Do not proceed to 5c/5d.

If the user accepts, present the specific candidates:

Use AskUserQuestion with multiSelect:

```
"Which issues should we file?"
Options (multiSelect):
  - "#1 [short title]" — [what happened, why it's a bug, not a config issue]
  - "#2 [short title]" — [what happened, why it's a bug, not a config issue]
  - ...
```

If no issues are selected, stop here.

### 5c. Ask Where to File

The user may have different rigs set up for tracking bugs. Do NOT assume a specific rig.

Use AskUserQuestion:

```
"Which rig should these bugs be filed in?"
Options:
  - List each rig from `gt rig list` output
  - Include a description hint: for a rig named "gastown", say "Gas Town codebase (gt bugs)"
  - For a rig named "beads", say "Beads codebase (bd bugs)"
```

If bugs span both gt and bd codebases, ask once and let the user choose — or offer "Split by codebase" to file gt bugs in one rig and bd bugs in another.

### 5d. Create Beads

For each selected bug, create a bead using `bd q` (quick capture):

```bash
bd q -p P2 -l bug -l gastown -l "gt-doctor" "<title>"
# or for beads bugs:
# bd q -p P2 -l bug -l beads -l "bd-doctor" "<title>"
```

Then add a detailed description with `bd update`:

```bash
bd update <id> --description "<description>"
```

**Description template:**

```
## Bug

[What went wrong — observable behavior]

## Expected Behavior

[What should have happened]

## Actual Behavior

[What actually happened, including exact error output or silent failure]

## Reproduction

1. [Step-by-step to reproduce]

## Workaround

[If one was applied during this session]

## Environment

- gt version: [from `gt --version` or stale-binary check output]
- bd version: [from `bd --version` or beads-binary check output]
- Platform: [OS]
```

**Labels:**
- `bug` — always
- `gastown` or `beads` — based on which codebase the bug is in
- `gt-doctor` or `bd-doctor` — to group doctor-specific bugs

**Priority:**
- P1 — blocks normal operation (e.g., data loss, can't start daemon)
- P2 — wrong behavior but has workaround (most doctor bugs)
- P3 — cosmetic or hint text issues

## Quick Reference

| Tool | Basic | Verbose | Fix | Deep |
|------|-------|---------|-----|------|
| `gt doctor` | `gt doctor` | `gt doctor -v` | `gt doctor --fix` | N/A |
| `bd doctor` | `bd doctor` | `bd doctor -v` | `bd doctor --fix` | `bd doctor --deep` |

| Scenario | Command |
|----------|---------|
| After upstream sync | `gt doctor -v` then `bd doctor -v` |
| After rig addition | `gt doctor --rig <name>` then `bd doctor -v` (from rig) |
| Dolt connectivity | `bd doctor --server` |
| Data integrity | `bd doctor --check=validate --fix` |
| Stale sessions | `gt doctor --fix --restart-sessions` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Looping `--fix` when it doesn't resolve | Investigate manually after first `--fix` attempt |
| Running `bd doctor` from wrong directory | Must be in repo root or `mayor/rig/` for Gas Town rigs |
| Treating server-mode "No dolt database" as real | Use `bd list` to verify — it's a false positive |
| Restarting daemon for every issue | Only restart when diagnostic says to or user asks |
| Using `gt up --restart` for stale settings | `gt up --restart` doesn't exist. Use `gt doctor --fix --restart-sessions` to delete stale Claude settings and recreate them properly |
