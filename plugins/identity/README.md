# Identity Plugin

A system for intentional identity change based on [this insightful article by Dan Koe](https://letters.thedankoe.com/p/how-to-fix-your-entire-life-in-1).

## Overview

The core insight: identity precedes behavior. You don't change what you do and hope to become someone different - you become someone different and your actions follow.

All behavior is goal-oriented, including self-sabotage. The patterns you want to break are protecting something. This system helps you surface what, then reconstruct around what you actually want.

## How it works

| Workflow | Duration | Purpose |
|----------|----------|---------|
| **Transformation** | 45-60 min | Deep excavation session. Surfaces your anti-vision, identifies the protective pattern, constructs your vision and identity statement. Do this first, then revisit monthly or quarterly. |
| **Interrupt** | 2-3 min | Daily reality check. Catches you in actual behavior, tests it against your framework, logs the moment. Set phone reminders throughout your day. |
| **Synthesis** | 10-15 min | Weekly review. Analyzes your interrupt logs, shows alignment trends, identifies triggers and escape patterns. This is where you see progress. |
| **Setup** | 2 min | Configure where your framework and logs are saved. |

## Installation

```
/plugin install identity@xexr-marketplace
```

## Usage

```
/identity                    # Show menu
/identity transformation     # Start transformation session
/identity interrupt          # Quick daily check-in
/identity synthesis          # Weekly pattern review
/identity setup              # Configure settings
```

## The Loop

```
transformation → framework.md
      ↑              ↓
synthesis ←←← interrupt-log.md ←←← interrupts
```

The framework evolves based on real behavioral data. Transformation produces the initial framework, interrupts test it against reality, synthesis identifies patterns and suggests refinements.

## Output

Your framework and interrupt logs are saved to a configurable location (default: `~/.claude/identity/`). You can optionally initialize git to track your evolution over time.

## License

MIT
