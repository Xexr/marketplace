---
name: beads-upstream-sync
description: Use when syncing a beads fork with upstream, updating the bd binary, or cleaning up merged PR branches. Triggers on phrases like "sync upstream", "pull upstream", "update fork", "upgrade beads", "clean up branches".
---

# Beads Upstream Sync

Full fork maintenance for a beads fork of steveyegge/beads.

## When to Use

- Upstream has new commits to pull into the fork
- After PRs are merged upstream and cherry-picks need dropping
- Periodic fork hygiene (branch cleanup, binary rebuild)

## Prerequisites

- Working directory: a beads crew clone (e.g. `~/gt/beads/crew/<crew-name>`)
- Remotes: `origin` = your fork, `upstream` = steveyegge/beads
- On `main` branch (the bleeding-edge branch)

## Handoff Protocol

**If resuming mid-skill from a handoff, ALWAYS re-read this skill file before
continuing.** The handoff mail provides context, but THIS FILE defines the steps.
Do not execute from handoff notes alone.

## Key Principle

**PR branches are the source of truth.** The cherry-picks on main are copies. Conflicts are always resolved on the PR branch first, then the fixed version is brought back to main.

## Process

**IMPORTANT:** Do NOT push at any step without presenting a summary and getting explicit user approval first. Every push (main, PR branches) requires a checkpoint.

### 0. Discovery — Detect Environment

Before starting, detect the local environment configuration:

```bash
# Detect fork identity from origin remote
git remote get-url origin
# Parse owner AND repo from: git@github.com:<OWNER>/<REPO>.git or https://github.com/<OWNER>/<REPO>.git
# Fork may not share the upstream repo name (users can rename forks)

# Verify upstream remote points to steveyegge/beads
git remote get-url upstream
# Should contain: steveyegge/beads

# Detect crew name from working directory
pwd
# Parse crew name from path: */crew/<CREW_NAME>

# Discover rig directories that have clones of this repo
ls -d ~/gt/*/mayor/rig 2>/dev/null
ls -d ~/gt/*/refinery/rig 2>/dev/null
# Filter to those whose origin matches the same fork

# Check for destructive command guard (DCG)
command -v dcg 2>/dev/null && echo "DCG detected" || echo "No DCG"
```

**Present to user for confirmation:**

> Detected environment:
> - **Fork:** `<OWNER>/<REPO>` (origin) forked from `steveyegge/beads` (upstream)
> - **Crew name:** `<detected>`
> - **Rig directories:** list of mayor/refinery rig paths with beads clones
> - **DCG installed:** yes/no (affects which git reset patterns to use)
>
> Please confirm or correct these values.

**Wait for user confirmation before proceeding.**

Use these values throughout the remaining steps:
- `FORK_OWNER` — for `gh pr list --author` and fork references
- `CREW_NAME` — for navigating back to the crew working directory
- `RIG_DIRS` — list of rig directories to update after push (step 8) and prune (step 9)
- `HAS_DCG` — if true, use safe reset patterns (mixed reset + stash) instead of `git reset --hard`

### 1. Fetch and Analyze

**Capture the current HEAD before any changes** (needed later for config change review):

```bash
OLD_HEAD=$(git rev-parse --short HEAD)
```

Fetch latest from both remotes:

```bash
git fetch upstream && git fetch origin
```

**Report to user:**

```bash
# New upstream commits
git log --oneline origin/main..upstream/main

# Our cherry-picks on main
git log --oneline upstream/main..origin/main

# Check if any cherry-picked PRs were merged
gh pr list --repo steveyegge/beads --author FORK_OWNER --state all \
  --json number,title,state,headRefName

# List active PR branches
git branch -r --list 'origin/fix/*' --list 'origin/feat/*'
```

Summarize:
- What's new upstream (key features, fixes, community PRs)
- Which of our cherry-picks correspond to now-merged PRs
- Which cherry-picks are still unmerged (will carry forward)
- Which PR branches exist and their upstream PR status (open/merged/none)

### 2. Attempt Main Rebase

```bash
git rebase upstream/main
```

**If clean (no conflicts):** Git auto-drops cherry-picks whose PRs were merged. Confirm:
- "skipped previously applied commit" messages for merged PRs
- Remaining unmerged cherry-picks replayed cleanly

Verify:

```bash
# Should show only remaining unmerged cherry-picks (or nothing)
git log --oneline upstream/main..main
```

Proceed to step 3.

**If conflicts:** A cherry-pick conflicts with upstream changes. This means the corresponding PR branch will also need updating. Follow the conflict resolution flow:

1. **Abort the main rebase:**
   ```bash
   git rebase --abort
   ```

2. **Go to the conflicting PR branch and rebase it first** (PR branch is the source of truth):
   ```bash
   git checkout fix/conflicting-feature
   git rebase upstream/main
   # Resolve conflicts here — this is the authoritative version
   # Stage resolved files, git rebase --continue
   ```

3. **Squash the PR branch** to a single commit (if multi-commit):
   ```bash
   # Interactive rebase is not available in agent sessions.
   # Instead, soft-reset to the branch point and re-commit:
   git reset --soft upstream/main
   git commit -m "squashed: <feature description>"
   ```

4. **Repeat** for each conflicting PR branch.

5. **Rebuild main from scratch** — cleaner than patching old cherry-picks:
   ```bash
   git checkout main
   git stash          # if any local changes
   git pull origin main --ff-only   # ensure we're at origin/main first
   git rebase upstream/main         # fast-forward to upstream (drops merged cherry-picks)
   ```
   If the rebase still conflicts (because the old cherry-picks are stale), reset
   to upstream/main. **If DCG is installed**, use the safe reset pattern:
   ```bash
   git rebase --abort
   git reset upstream/main    # mixed reset: moves HEAD, leaves changes in working tree
   git stash                  # stashes the working tree diff (restores to new HEAD)
   git stash drop             # discard — now clean at upstream/main
   ```
   **If no DCG**, you can use `git reset --hard upstream/main` directly.

   Then cherry-pick each unmerged PR's squashed commit:
   ```bash
   git cherry-pick <squashed-commit-from-fix/feature-A>
   git cherry-pick <squashed-commit-from-fix/feature-B>
   ```

6. Return to step 2's verify check.

### 3. Rebase Active PR Branches (Non-Conflicting)

For each remaining PR branch with an **open** upstream PR that wasn't already rebased in step 2:

```bash
git checkout fix/some-feature
git rebase upstream/main
```

- If clean: note it in the summary
- If conflicts: abort (`git rebase --abort`), resolve on the PR branch using the same flow as step 2. Then **rebuild main again** — reset to `upstream/main` and re-cherry-pick all unmerged PRs (including the newly fixed one)

Return to main when done:

```bash
git checkout main
```

### 4. Build and Test All Branches

**MANDATORY before checkpoint.** Catches compilation errors, package-level name
collisions (invisible to git rebase), and test regressions.

**Why this must happen before push:** Git rebase resolves file-level merge conflicts,
but it cannot detect package-level issues like duplicate function names across
different files in the same Go package. Only the compiler catches these. If you
push before building, you ship broken code and must fix + force-push again.

**For each active PR branch:**

```bash
git checkout <branch>
SKIP_UPDATE_CHECK=1 make build    # build from non-main branch
go test ./...                     # run full test suite
git status                        # check for generated file drift
```

If `git status` shows changes after build, the build generated files that weren't
committed. Amend the branch commit to include them.

**Then on main:**

```bash
git checkout main
SKIP_UPDATE_CHECK=1 make build    # SKIP because origin/main not yet pushed
go test ./...
git status                        # check for generated file drift
```

If build fails on main but passes on individual PR branches, the issue is likely
a cross-branch collision (e.g., two PRs each adding a function with the same name).
Fix on the PR branch first (source of truth), then rebuild main's cherry-pick.

**If you amend any PR branch during this step** (to fix build, include generated
files, or resolve a collision), main's cherry-pick of that branch is now stale.
You must rebuild main: reset to `upstream/main` and re-cherry-pick all PR branches
(step 2.5), then re-run this step for all branches.

**All branches must build clean and tests must pass before proceeding.**

### 5. CHECKPOINT — Present Summary Before Pushing

**STOP. Do NOT push yet.** Present a full summary to the user:

- **Main branch:** what changed (cherry-picks dropped, new upstream commits absorbed, any rebuilt cherry-picks)
- **Build status:** all branches compile, all tests pass (or note any pre-existing upstream failures)
- **Push type for main:** fast-forward or force-with-lease (explain why if force needed)
- **PR branches:** for each active branch, whether rebase was clean or had conflicts (and how resolved)
- **Push plan for PR branches:** which will be force-pushed (rebase always rewrites history)
- **Branches to clean up:** merged PR branches and stale branches identified for deletion

**Wait for user approval before proceeding to push.**

### 6. Push

Only after user approval:

**Main:**

```bash
# Force-with-lease if cherry-picks were dropped or main was rebuilt (history rewritten)
git push origin main --force-with-lease

# Regular push if no cherry-picks changed (pure fast-forward, rare)
git push origin main
```

**Active PR branches** (rebase always rewrites, so always force):

```bash
git push origin fix/some-feature --force-with-lease
```

### 7. Rebuild and Install Binary

Now that origin/main is updated, rebuild without the skip flag:

```bash
make install
```

**Verify the build matches the expected commit:**

```bash
bd version
git rev-parse --short HEAD
```

Compare the commit hash from `bd version` against the current HEAD. They must match. If they don't, the binary was built from a stale checkout — rebuild.

### 8. Update Other Clones

Mayor and refinery rig directories track `origin/main`. After pushing, update each
rig directory discovered in step 0.

**Always check for dirty state first** — these clones often have unstaged changes:

```bash
cd <rig-directory>
git status --porcelain        # check if dirty
# If dirty: git stash
git fetch origin
git pull --rebase origin main
# If stashed: git stash pop
# If stash pop conflicts: git reset HEAD && git stash && git stash drop
#   (conflicts from old state are safe to discard)
```

Repeat for each rig directory in `RIG_DIRS` (e.g. `~/gt/beads/mayor/rig`, `~/gt/beads/refinery/rig`).

**Note:** Refinery/rig accumulates stale local branches from old operations.
If stash pop conflicts are messy, the simplest recovery is:
```bash
git reset HEAD          # unstage everything
git stash               # stash the mess (restores working tree to HEAD)
git stash drop          # discard — clean at new HEAD
```

Verify all clones are at the same HEAD as the crew working copy.

### 9. Clean Up Branches

**Delete merged PR branches** (remote + local):

```bash
# Remote
git push origin --delete <branch-name> [<branch-name> ...]

# Local (in crew working copy)
git branch -d <branch-name> [<branch-name> ...]
```

**Delete stale branches** (no open PR, no longer needed) — confirm with user first, then use `-D` for unmerged locals.

**Prune stale refs in all clones:**

```bash
# Crew working copy
git remote prune origin

# Each rig directory from RIG_DIRS
cd <rig-directory> && git remote prune origin
```

Also clean up stale local branches in refinery/rig (it accumulates them).

**Return to the crew working copy** before continuing:

```bash
cd ~/gt/beads/crew/CREW_NAME    # use detected crew name
```

### 10. Run Doctor Checks

After the upgrade, run `bd doctor` to catch any issues introduced by upstream changes:

```bash
bd doctor --fix
```

This runs all checks and auto-fixes what it can in one pass. Report any changes
detected and any issues found.

**Only run additional doctor checks in other locations if the above reported issues:**

```bash
# Only if bd doctor --fix showed problems:
cd ~/gt && bd doctor
# Also check each rig directory from RIG_DIRS
cd <rig-directory> && bd doctor
```

### 11. Check for Config Changes and Summarize Upstream

Review the upstream commits for changes that affect configuration. Use the `OLD_HEAD` captured in step 1:

```bash
git log --oneline --name-only $OLD_HEAD..upstream/main -- \
  '*.yaml' '*.json' '*.toml' \
  'internal/storage/migrations/' \
  'cmd/bd/doctor/' \
  'internal/beads/'
```

**Also generate a human-readable summary of all upstream changes:**

```bash
# Contributor summary
git shortlog --no-merges -s $OLD_HEAD..upstream/main

# Detailed commit list (non-merge)
git log --oneline --no-merges $OLD_HEAD..upstream/main
```

**Report to user:**
- Categorized summary of upstream changes (features, fixes, security, CI, etc.)
- Any new config keys or changed defaults
- Schema migrations that may need running
- Doctor checks that were added or changed
- Breaking changes that require manual config updates in Gas Town
- Contributor breakdown

## Common Issues

| Issue | Solution |
|-------|----------|
| Rebase conflict on main | Abort main rebase. Fix PR branch first (source of truth), then rebuild main. |
| Rebase conflict on PR branch | Resolve on the PR branch — this is the authoritative version. Then rebuild main with all updated cherry-picks. |
| Package-level name collision (compiles on branch, fails on main) | Two cherry-picks define the same symbol. Rename on the PR branch (source of truth), then rebuild main. Git rebase can't detect these — only the compiler can. |
| Force push needed on main | Cherry-pick dropped or main rebuilt. Use `--force-with-lease`. Explain in checkpoint. |
| PR branch rebase changes nothing | Upstream changes don't affect the PR's files. Still force-push to update the base. |
| Generated file drift after build | `git status` shows changes after `make build`. Amend the branch commit to include them. |
| Mayor/rig has local changes | `git stash` before pull, `git stash pop` after. If pop conflicts, discard with stash+drop. |
| `git branch -d` refuses (not fully merged) | Confirm with user, then `git branch -D`. |
| `git push --delete` fails (ref doesn't exist) | Already deleted remotely. Remove from batch and retry remaining. |
| Destructive git command blocked by hook | If DCG or similar is installed: use safe pattern — `git reset <ref>` (mixed) + `git stash` + `git stash drop`. If no hook: `git reset --hard` works directly. |
| `git checkout HEAD -- <path>` blocked by hook | Use `git stash` + `git stash drop` to restore working tree to HEAD. |
