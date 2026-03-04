# Feature Branch PR Plugin

End-to-end workflow for creating feature branch PRs against upstream (steveyegge/beads or steveyegge/gastown) from your fork.

## The Core Principle

PR branches start from `upstream/main`, not your fork's main. Your fork's main gets the change via cherry-pick after the PR is submitted. This keeps PRs clean and your fork current.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install feature-branch-pr@xexr-marketplace
```

## When to Use

- You have a bead describing work to contribute upstream
- You want to create a clean PR from `upstream/main`
- Applies to both beads (steveyegge/beads) and gastown (steveyegge/gastown) forks

## When NOT to Use

- Upstream sync (use `beads-upstream-sync` or `gastown-upstream-sync`)
- Work that stays on your fork only (just commit to main)
- Finishing a branch already created (use `finishing-branch-rebase`)

## Prerequisites

- A fork of steveyegge/beads or steveyegge/gastown with `origin` and `upstream` remotes configured
- `bd` (beads) installed for issue tracking
- `gh` CLI for PR creation
- Go toolchain for build/test

## How It Works

A five-phase process covering the full PR lifecycle:

1. **Setup** - Auto-detect fork/upstream from git remotes, select a bead (from `$ARGUMENTS` or `bd ready`), create feature branch from `upstream/main`, mark bead in-progress
2. **Implement** - Investigate, code, edit-build-test loop, full verification (`go test`, `gofmt`, `go vet`, `make build`), user confirmation before proceeding
3. **Submit PR** - Commit with conventional message, push branch to origin, create PR against upstream via `gh pr create`
4. **Cherry-pick to main** - Squash if needed, cherry-pick onto fork's main, push, rebuild and install binary, update other rig clones, run doctor
5. **Cleanup** - Close the bead with PR reference, return to main, print summary

## Key Guardrails

- Never branch from `origin/main` (always `upstream/main` for clean PRs)
- Never open a PR from your fork's `main` branch
- Never push to main without building and testing
- User confirmation required before PR submission
- One branch per PR, one concern per PR

## Commands

| Command | Purpose |
|---------|---------|
| `/feature-branch-pr` | Start the workflow (optionally pass a bead ID) |
| `/feature-branch-pr bd-xxx` | Start with a specific bead |

## License

MIT
