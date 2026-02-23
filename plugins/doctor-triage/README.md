# Doctor Triage Plugin

Diagnose and fix Gas Town and beads workspace health issues. Runs `gt doctor` and `bd doctor`, interprets findings, auto-fixes what it can, and optionally captures upstream bugs as beads.

## The Core Principle

Doctor output is noisy. Some findings are auto-fixable, some need manual investigation, some are false positives, and some reveal genuine upstream bugs. This skill triages each finding, applies fixes in the right order, and surfaces real bugs instead of burying them in terminal output.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install doctor-triage@xexr-marketplace
```

## When to Use

- After upgrades, upstream syncs, or rig additions
- When workspace or tooling feels misconfigured
- User asks to run or fix doctor warnings
- Periodic health checks

## When NOT to Use

- Setting up a new rig from scratch (use `gastown-rig-setup`)
- Debugging a specific test failure (use `systematic-debugging`)

## How It Works

A five-step process covering diagnosis through optional upstream bug capture:

1. **Pre-flight** - Check which tools (`gt`, `bd`) are available and constrain options accordingly
2. **Diagnose** - Run `gt doctor -v` and/or `bd doctor -v`
3. **Triage** - Categorize each finding as auto-fixable, manually fixable, or upstream bug
4. **Fix** - Apply fixes in order, verify each one, run a final clean check
5. **Reflect** - Review the full run for upstream bugs. If any found, offer to capture them as beads (opt-in, not assumed)

## Key Guardrails

- **No fix loops** - If `--fix` doesn't resolve on the first pass, investigates manually
- **No daemon restarts** - Unless the diagnostic specifically recommends it or the user asks
- **Known false positives** - Skips `bd doctor` "No dolt database found" in server mode (cosmetic, not real)
- **Upstream bugs are opt-in** - Asks before filing anything, user can decline

## Commands

| Command | Purpose |
|---------|---------|
| `/doctor-triage` | Diagnose and fix workspace health issues |

## License

MIT
