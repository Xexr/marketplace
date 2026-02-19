# Gastown Rig Setup Plugin

Add Git repositories as Gas Town rigs with Dolt-native beads integration. Handles `.beads/` seeding, rig creation, Dolt initialization, and remote configuration.

## The Core Principle

`.beads/` must be seeded in the upstream repo before running `gt rig add`. Without seeding, `bd init` writes a mismatched database name (`beads_<prefix>` instead of the actual rig name), breaking Dolt connectivity. The skill automates the correct sequence.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install gastown-rig-setup@xexr-marketplace
```

## When to Use

- Adding a new Git repository as a Gas Town rig
- User says "add a rig", "set up a rig", "onboard this repo to Gas Town"

## Prerequisites

Before starting, you'll need:

1. **Rig name** - Short, single-word, no hyphens (hyphens cause SQL quoting issues)
2. **Beads prefix** - 2-3 letter abbreviation that doesn't conflict with existing prefixes
3. **Git remote URL** - SSH or HTTPS

## The Process

1. **Clone** - Clone the repo into `~/projects/`
2. **Seed `.beads/`** - Create minimal `.beads/` with correct `metadata.json` (sets `dolt_database` to the rig name)
3. **Commit & Push** - Push `.beads/` to remote so `gt rig add` clones pick it up
4. **Add Rig** - `gt rig add <name> <url> --prefix <prefix>`
5. **Init Dolt** - `gt dolt init-rig <name>` + restart Dolt server
6. **Configure Remote** - Set up local Dolt remote for the new database
7. **Verify** - Check config files, run `bd doctor` and `gt doctor`

## Known Issues

### Database naming bug (why seeding is required)

`gt rig add` takes two code paths depending on whether `.beads/` exists in the cloned repo. The unseeded path has a bug where `bd init --server` writes `dolt_database: "beads_<prefix>"` but the actual Dolt database is named `<rigName>`. This mismatch prevents `bd` from connecting. Seeding `.beads/` with the correct `dolt_database` value prevents `bd init` from running its naming logic.

### `bd doctor` false positive in server mode

`bd doctor` reports `No dolt database found` when using the centralized Dolt server. This is cosmetic — it checks for a local `.beads/dolt/` directory which doesn't exist in server mode. Use `bd list` to verify actual connectivity instead.

## Commands

| Command | Purpose |
|---------|---------|
| `/gastown-rig-setup` | Add a new Git repository as a Gas Town rig |

## License

MIT
