---
name: gastown-rig-setup
description: Use when adding a new Git repository as a Gas Town rig with Dolt-native beads. Triggers on "add a rig", "set up a rig", "onboard repo to Gas Town", "new rig".
---

# Gas Town Rig Setup

## Overview

Add a Git repository as a Gas Town rig with Dolt-native beads integration. The process requires seeding `.beads/` in the upstream repo first, then adding the rig, initializing Dolt, and configuring remotes.

**Why seed `.beads/` first?** `gt rig add` behaves differently depending on whether the cloned repo has a tracked `.beads/` directory. Seeding ensures the correct code path runs. See "Why Seeding Matters" below for the technical details.

## When to Use

- Adding a new repository as a Gas Town rig
- User says "add a rig", "set up a rig", "onboard this repo"

## When NOT to Use

- Setting up beads in a standalone repo (use `beads-setup` skill)
- Migrating an existing rig from SQLite to Dolt (see MEMORY.md migration notes)

## Prerequisites

Before starting, confirm with the user:

1. **Rig name** - Short, single-word, no hyphens (hyphens cause SQL quoting issues). E.g., `toolkit` not `gt-toolkit`.
2. **Beads prefix** - 2-3 letter abbreviation. E.g., `tk`. Must not conflict with existing prefixes in `~/gt/.beads/routes.jsonl`.
3. **Git remote URL** - SSH (`git@github.com:Org/repo.git`) or HTTPS (`https://github.com/Org/repo.git`). Both work with `gt rig add`.

Check existing prefixes:
```bash
cat ~/gt/.beads/routes.jsonl
```

## Steps

### 1. Clone repo into ~/projects/

```bash
cd ~/projects && git clone <SSH_URL> <repo-dir-name>
```

### 2. Seed minimal .beads/ in the clone

Create four files using an existing rig as template (e.g., `~/projects/onyx/.beads/`):

**`.beads/.gitignore`** — Copy from template rig verbatim.

**`.beads/config.yaml`**:
```yaml
sync.mode: "dolt-native"
```

**`.beads/metadata.json`** — Set `dolt_database` to the rig name:
```json
{
  "backend": "dolt",
  "database": "dolt",
  "dolt_database": "<RIG_NAME>",
  "dolt_mode": "server",
  "dolt_server_port": 3307
}
```

**`.beads/README.md`** — Copy from template rig verbatim.

### 3. Commit and push .beads/

```bash
cd ~/projects/<repo-dir>
git add .beads/
git commit -m "feat: add minimal .beads/ for Gastown rig integration"
git push
```

### 4. Add the rig

```bash
gt rig add <RIG_NAME> <SSH_URL> --prefix <PREFIX>
```

This creates `~/gt/<RIG_NAME>/` with the full rig structure (mayor/rig, crew/, refinery/, witness/, polecats/).

### 5. Initialize the Dolt database

```bash
gt dolt init-rig <RIG_NAME>
```

Creates `~/gt/.dolt-data/<RIG_NAME>/`.

### 6. Restart Dolt server

```bash
gt dolt stop && gt dolt start
```

Verify the new database appears:
```bash
gt dolt list
```

**STOP if the rig database doesn't appear.** Investigate before proceeding.

### 7. Configure Dolt remote

```bash
mkdir -p ~/dolt-remotes/<RIG_NAME>
cd ~/gt/.dolt-data/<RIG_NAME> && dolt remote add origin file:///home/xexr/dolt-remotes/<RIG_NAME>
cd ~/gt/.dolt-data/<RIG_NAME> && dolt push origin main
```

### 8. Verify configuration

Check these files have correct values:

| File | Key fields |
|------|-----------|
| `~/gt/<RIG_NAME>/config.json` | `git_url`, `prefix` |
| `~/gt/<RIG_NAME>/.beads/redirect` | Points to `mayor/rig/.beads` |
| `~/gt/<RIG_NAME>/mayor/rig/.beads/metadata.json` | `dolt_database: "<RIG_NAME>"`, `dolt_mode: "server"`, `dolt_server_port: 3307` |

### 9. Run diagnostics

```bash
cd ~/gt/<RIG_NAME>/mayor/rig && bd doctor
gt doctor
```

Investigate and fix any issues before declaring complete.

**Known false positive:** `bd doctor` reports `✖ Database: No dolt database found` in server mode. This is cosmetic — it checks for a local `.beads/dolt/` directory which doesn't exist when using the centralized Dolt server at `.dolt-data/`. Verify actual connectivity with `bd list` instead; if that returns results, the database is working.

## Verification Checklist

```
[ ] gt rig list — shows new rig
[ ] gt dolt list — shows new database
[ ] bd list — returns results (from within rig's mayor/rig/)
[ ] gt doctor — passes (marketplace-related checks)
[ ] ~/dolt-remotes/<RIG_NAME>/ — has data after push
[ ] cat ~/gt/.dolt-data/<RIG_NAME>/.dolt/repo_state.json — shows origin remote
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hyphen in rig name (`gt-toolkit`) | Use single word (`toolkit`). Hyphens require SQL backtick quoting everywhere. |
| Skipping `.beads/` seeding | Without seeding, `bd init` writes `dolt_database: "beads_<prefix>"` which mismatches the actual database name. See "Why Seeding Matters". |
| Forgetting to restart Dolt | New database won't be visible until `gt dolt stop && gt dolt start`. |
| Prefix conflicts | Check `routes.jsonl` before choosing. Conflicts cause routing errors. |
| Not pushing `.beads/` before `gt rig add` | The clones created by `gt rig add` pull from remote. If `.beads/` isn't pushed, they won't have it. |
| Treating `bd doctor` "No dolt database found" as real | False positive in server mode — checks for local `.beads/dolt/` not the centralized server. Use `bd list` to verify connectivity instead. |

## Why Seeding Matters

`gt rig add` (`internal/rig/manager.go:AddRig`) takes two different code paths depending on whether the cloned repo has a tracked `.beads/` directory. **The unseeded path has a database naming bug that breaks Dolt connectivity.**

### The bug in the unseeded path

When `.beads/` is NOT in the repo, the following sequence creates a name mismatch:

1. `doltserver.InitRig(townRoot, "toolkit")` at `manager.go:485` creates a Dolt database named **`toolkit`**
2. `InitBeads` at `manager.go:765` runs `bd init --prefix tk --server` (no existing `.beads/` to redirect to)
3. `bd init` at `init.go:457` writes `dolt_database: "beads_tk"` to `metadata.json` — this is bd's own naming convention (`"beads_" + prefix`)
4. `EnsureMetadata` at `doltserver.go:1570` checks `if dolt_database == nil || == ""` — it's `"beads_tk"`, **so it does not override**
5. **Result:** `metadata.json` says `dolt_database: "beads_tk"` but the actual database is `"toolkit"`. `bd` cannot connect.

### How seeding prevents this

When `.beads/` IS in the repo (with the correct `dolt_database: "<RIG_NAME>"` in `metadata.json`):

1. After cloning into `mayor/rig/`, the code detects `.beads/` at `manager.go:419`
2. `doltserver.InitRig` at `manager.go:485` creates database `"toolkit"` and calls `EnsureMetadata` — which finds `mayor/rig/.beads/metadata.json` already has the correct `dolt_database: "toolkit"` and leaves it alone
3. `InitBeads` at `manager.go:734` sees `mayor/rig/.beads` exists → creates a **redirect file** at `<rig>/.beads/redirect` → `mayor/rig/.beads` → returns early without running `bd init` (so the correct `dolt_database` is never overwritten)
4. Route registration at `manager.go:644` sets the route path to `<name>/mayor/rig`
5. `EnsureMetadata` at `manager.go:503` confirms `dolt_mode: "server"`, `dolt_database: "toolkit"` — all correct

**Result:** `mayor/rig/.beads/` is the canonical beads location (tracked in git). The rig-level `.beads/redirect` points all agents there. The database name matches. Config persists across clones.

### Summary

Seeding `.beads/` with the correct `metadata.json` is not just a preference — it's a workaround for a bug where `bd init --server` writes `dolt_database: "beads_<prefix>"` which conflicts with the centralized Dolt database named `<rigName>`. Without seeding, `bd` connects to a non-existent database.

## Quick Reference

```bash
# Full sequence for adding rig "myrig" with prefix "mr" from git@github.com:Org/repo.git

# 1. Clone and seed
cd ~/projects && git clone git@github.com:Org/repo.git repo-dir
# Create .beads/ files (see Step 2 above)
cd ~/projects/repo-dir && git add .beads/ && git commit -m "feat: add .beads/" && git push

# 2. Add rig + init dolt
gt rig add myrig git@github.com:Org/repo.git --prefix mr
gt dolt init-rig myrig
gt dolt stop && gt dolt start
gt dolt list  # verify "myrig" appears

# 3. Configure remote
mkdir -p ~/dolt-remotes/myrig
cd ~/gt/.dolt-data/myrig && dolt remote add origin file:///home/xexr/dolt-remotes/myrig
cd ~/gt/.dolt-data/myrig && dolt push origin main

# 4. Verify
cd ~/gt/myrig/mayor/rig && bd doctor
gt doctor
```
