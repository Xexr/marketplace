---
name: feature-branch-pr
description: Use when creating a fix or feature branch PR for a Gas Town fork (beads or gastown). Triggers on phrases like "fix PR", "feature branch", "create a PR", "work on issue", "submit upstream". Expects a bead ID or will help select one.
---

# Feature Branch PR

End-to-end workflow for creating a fix/feature branch, implementing changes, submitting a PR to upstream, and cherry-picking onto your fork's main.

## When to Use

- You have a bead describing work to contribute upstream
- You want to create a clean PR from `upstream/main`
- Applies to both beads (steveyegge/beads) and gastown (steveyegge/gastown) forks

## When NOT to Use

- Upstream sync (use /beads-upstream-sync or /gastown-upstream-sync)
- Work that stays on your fork only (just commit to main)
- Finishing a branch already created (use finishing-branch-rebase)

## Handoff Protocol

**If resuming mid-skill from a handoff, ALWAYS re-read this skill file before continuing.** The handoff mail provides context, but THIS FILE defines the steps.

## Arguments

`$ARGUMENTS` — optional bead ID (e.g. `bd-abc` or `gt-xyz`)

## Process

### Phase 1: Setup

#### 1.1 Detect Environment

```bash
# Detect fork from origin remote
git remote get-url origin
# Parse FORK_OWNER and REPO_NAME

# Detect upstream
git remote get-url upstream
# Parse UPSTREAM_OWNER and UPSTREAM_REPO (e.g. steveyegge/beads)

# Detect project type from upstream URL
# "beads" in URL → Go project, binary = bd, cmd = cmd/bd
# "gastown" in URL → Go project, binary = gt, cmd = cmd/gt

# Working directory
pwd
# Parse CREW_NAME from path: */crew/<CREW_NAME>
```

**Present to user for confirmation:**

> Detected environment:
> - **Fork:** `FORK_OWNER/REPO_NAME` (origin) forked from `UPSTREAM_OWNER/UPSTREAM_REPO` (upstream)
> - **Project:** beads/gastown
> - **Crew:** CREW_NAME
> - **Binary:** bd/gt
>
> Please confirm or correct.

#### 1.2 Select Work Item

**If bead ID provided in `$ARGUMENTS`:**

```bash
bd show <BEAD_ID>
```

Read the bead to understand the issue/feature. Extract: title, description, any linked GitHub issues.

**If NO bead ID provided:**

```bash
bd ready -n 20
```

Present the ready issues and ask:

> Which issue would you like to work on? Pick from the list or provide a bead ID.

After selection, read the full bead with `bd show`.

#### 1.3 Determine Branch Name

Based on the bead type and title, propose a branch name:

- Bug/fix: `fix/<short-kebab-description>`
- Feature: `feat/<short-kebab-description>`
- Chore/refactor: `chore/<short-kebab-description>`

Ask user to confirm or adjust the branch name.

#### 1.4 Check for Linked GitHub Issue

Search for an existing upstream GitHub issue:

```bash
# If bead mentions a GH issue number
gh issue view <NUMBER> --repo UPSTREAM_OWNER/UPSTREAM_REPO --json title,state,body 2>/dev/null

# Otherwise search by keywords from bead title
gh search issues "<keywords>" --repo UPSTREAM_OWNER/UPSTREAM_REPO --json number,title,state 2>/dev/null | head -5
```

If found, note the issue number for the PR body (`Closes #NNN`).

#### 1.5 Create Feature Branch

```bash
git fetch upstream
git checkout -b <BRANCH_NAME> upstream/main
```

#### 1.6 Mark Bead In-Progress

```bash
bd update <BEAD_ID> -s in_progress
```

### Phase 2: Implementation

#### 2.1 Investigate and Plan

Read the relevant source files. Understand the current behavior. Plan the fix/feature.

Use EnterPlanMode if the change is non-trivial. For simple fixes, proceed directly.

#### 2.2 Edit-Build-Test Loop

Make changes, then verify:

```bash
# Quick build (fast iteration)
go build -o ./<BINARY> ./cmd/<BINARY>

# Run targeted tests
go test ./path/to/package/ -run "TestRelevantPattern" -v -count=1

# Manual verification if applicable
./<BINARY> <command-to-test>
```

Iterate until the fix/feature works correctly.

#### 2.3 Full Verification

Before asking user to confirm:

```bash
# Full test suite
go test ./...

# Format check
gofmt -l .

# Static analysis
go vet ./...

# Full build with version info
make build
```

Report any failures. Fix before proceeding.

#### 2.4 User Confirmation

Present a summary of changes:

```bash
git diff --stat
```

Show what changed and why. Ask:

> The fix/feature is implemented and tests pass. Please verify the behavior is correct.
> Would you like to proceed with creating the PR?

**Wait for explicit user approval before proceeding.**

### Phase 3: Submit PR

#### 3.1 Commit

Stage and commit with a conventional commit message:

```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<body explaining what changed and why>

<optional: Closes #NNN or (BEAD_ID)>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

**Commit message conventions:**
- Present tense ("Add feature" not "Added feature")
- First line under 72 characters
- Type: `fix`, `feat`, `docs`, `chore`, `refactor`
- Reference issues: `Closes #123` or `(bd-xxx)`

#### 3.2 Push Feature Branch

```bash
git push -u origin <BRANCH_NAME>
```

#### 3.3 Create PR

```bash
gh pr create --repo UPSTREAM_OWNER/UPSTREAM_REPO \
  --head FORK_OWNER:<BRANCH_NAME> \
  --title "<type>(<scope>): <description>" \
  --body "$(cat <<'EOF'
## Summary

<2-3 bullet points explaining the change>

## Problem

<What was wrong / what was missing>

## Solution

<What this PR does to fix/add it>

## Before / After

<If applicable, show behavior change>

## Testing

- <How the change was tested>
- <Relevant test commands>

<optional: Closes #NNN>
EOF
)"
```

**PR body guidelines:**
- Explain the problem clearly (what was wrong, or what was missing)
- Explain the solution (what the PR does)
- Include before/after if behavior changed
- Reference the upstream GitHub issue if one exists
- Keep it concise but informative for reviewers

#### 3.4 Report PR URL

Show the user the PR URL so they can review it on GitHub.

### Phase 4: Cherry-Pick to Main

#### 4.1 Squash Feature Branch

If the branch has multiple commits, squash to one:

```bash
# Soft-reset to branch point and re-commit
git reset --soft upstream/main
git commit -m "<type>(<scope>): <squashed description>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

**Note the squashed commit hash.**

#### 4.2 Cherry-Pick onto Main

```bash
git checkout main
git cherry-pick <BRANCH_NAME>
```

If cherry-pick conflicts (rare for fresh work):

```bash
# Resolve conflicts
git add <resolved-files>
git cherry-pick --continue
```

#### 4.3 Push Main

```bash
git push origin main
```

#### 4.4 Build and Install

```bash
make install
```

Verify:

```bash
<BINARY> version 2>/dev/null || <BINARY> --version 2>/dev/null
git rev-parse --short HEAD
```

Confirm the version hash matches HEAD.

#### 4.5 Update Other Clones

Discover and update rig directories:

```bash
# Find rig directories for this project
GT_HOME="${GT_HOME:-$HOME/gt}"
ls -d "$GT_HOME"/*/mayor/rig "$GT_HOME"/*/refinery/rig 2>/dev/null
# Filter to those tracking the same fork
```

For each rig directory:

```bash
cd <RIG_DIR>
git status --porcelain
# If dirty: git stash
git fetch origin && git pull --rebase origin main
# If stashed: git stash pop
```

#### 4.6 Run Doctor

```bash
bd doctor --fix
# or gt doctor --fix (depending on project)
```

Report any issues.

### Phase 5: Cleanup

#### 5.1 Close Bead

```bash
bd close <BEAD_ID> -r "Fixed in PR #<PR_NUMBER>: <PR_URL>"
```

#### 5.2 Return to Main

```bash
git checkout main
```

#### 5.3 Summary

Present final summary:

```
================================================================
FEATURE BRANCH PR COMPLETE
================================================================

  Branch:  <BRANCH_NAME>
  PR:      <PR_URL>
  Bead:    <BEAD_ID> (closed)

  Cherry-picked to main: <COMMIT_HASH>
  Binary installed: <BINARY> @ <VERSION>

  Rig clones updated: <list>
  Doctor: <pass/issues>

================================================================
```

## Common Issues

| Issue | Solution |
|-------|----------|
| Tests fail on feature branch | Fix before proceeding. Don't submit broken PRs. |
| Cherry-pick conflicts with main | Main has other cherry-picks. Resolve on main, or rebuild main from upstream + cherry-picks. |
| PR branch has upstream's cherry-pick too | You branched from `origin/main` not `upstream/main`. Start over from `upstream/main`. |
| `gh pr create` fails with "head not found" | Push the branch first (`git push -u origin <branch>`). |
| Multiple commits on feature branch | Squash before cherry-picking to main (step 4.1). |
| Build fails after cherry-pick to main | Cross-branch collision with another cherry-pick. Fix on feature branch, re-squash, rebuild main. |

## Red Flags

**Never:**
- Branch from `origin/main` (use `upstream/main` for clean PRs)
- Open a PR from your fork's `main` branch
- Push to main without building and testing
- Skip user confirmation before PR submission
- Force-push main (others may track it)

**Always:**
- One branch per PR, one concern per PR
- Run full test suite before submitting
- Include clear PR description with problem/solution
- Cherry-pick finalized work onto main
- Update all rig clones after pushing main
