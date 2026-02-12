# Beads Upstream Sync Plugin

Fork maintenance for beads (steveyegge/beads) - upstream sync, cherry-pick management, binary rebuild, and branch cleanup.

## The Core Principle

PR branches are the source of truth. Cherry-picks on main are copies. Conflicts are always resolved on the PR branch first, then the fixed version is brought back to main.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install beads-upstream-sync@xexr-marketplace
```

## When to Use

- Upstream has new commits to pull into your fork
- After your PRs are merged upstream and cherry-picks need dropping
- Periodic fork hygiene (branch cleanup, binary rebuild)

## Prerequisites

- A beads crew clone with `origin` pointing to your fork and `upstream` pointing to steveyegge/beads
- On the `main` branch
- `gh` CLI authenticated with GitHub

## Auto-Detection

The skill automatically detects your environment before starting:

| Value | Detection Method | Used For |
|-------|-----------------|----------|
| Fork owner | Parsed from `git remote get-url origin` | `gh pr list --author`, fork references |
| Crew name | Parsed from current working directory | Navigation back to crew clone |
| DCG installed | `command -v dcg` | Choosing safe vs direct reset patterns |

You'll be asked to confirm or correct these values before the sync begins.

## The Process

1. **Discovery** - Detect fork owner, crew name, DCG presence
2. **Fetch & Analyze** - Compare upstream, identify merged/unmerged PRs
3. **Rebase Main** - Drop merged cherry-picks, replay unmerged ones
4. **Rebase PR Branches** - Update active branches against upstream
5. **Build & Test** - Catch compilation errors and package-level collisions
6. **Checkpoint** - Full summary for user approval before any push
7. **Push** - Main and PR branches (force-with-lease where needed)
8. **Rebuild Binary** - `make install` and verify with `bd version`
9. **Update Clones** - Sync mayor and refinery rig directories
10. **Clean Up** - Delete merged branches, prune stale refs
11. **Doctor** - Run `bd doctor --fix` for post-upgrade checks
12. **Summarize** - Categorized upstream changes and config impact

## Commands

| Command | Purpose |
|---------|---------|
| `/beads-upstream-sync` | Sync your beads fork with upstream |

## License

MIT
