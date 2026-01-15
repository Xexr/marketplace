---
name: identity
description: Use when user wants to examine their life direction, break patterns, or make meaningful change. Signs - feeling stuck, recognizing self-sabotage, wanting clarity on identity and goals. Supports transformation sessions, daily interrupts, and weekly synthesis.
---

# Identity

A system for intentional identity change based on Dan Koe's methodology: identity precedes behavior. You become the person whose lifestyle naturally produces desired outcomes.

## Components

| Workflow | Purpose | Duration |
|----------|---------|----------|
| **transformation** | Deep excavation → produce identity framework | 45-60 min |
| **interrupt** | Daily reality check → log patterns | 2-3 min |
| **synthesis** | Weekly review → track progress, refine framework | 10-15 min |
| **setup** | Configure output path and settings | 2 min |

## Invocation

**With argument** → go directly to that workflow:
- `/identity transformation`
- `/identity interrupt`
- `/identity synthesis`
- `/identity setup`

**Without argument** → show menu (see Menu Flow below)

## Configuration

Config file: `~/.claude/identity-config.json`

```json
{
  "outputPath": "/path/to/identity/folder",
  "createdAt": "2025-01-15",
  "gitInitialized": true
}
```

**Output folder contains:**
- `framework.md` - the user's living identity framework
- `interrupt-log.md` - accumulated check-in entries

## Menu Flow

### Overview (show every time `/identity` is invoked without arguments)

Always start with the overview to orient the user. Present this directly (not in a code block):

**Identity**

A system for intentional identity change, based on this insightful article by Dan Koe: https://letters.thedankoe.com/p/how-to-fix-your-entire-life-in-1

The core insight: identity precedes behavior. You don't change what you do and hope to become someone different - you become someone different and your actions follow.

All behavior is goal-oriented, including self-sabotage. The patterns you want to break are protecting something. This system helps you surface what, then reconstruct around what you actually want.

**How it works**

- **Transformation** (45-60 min) - Deep excavation session. Surfaces your anti-vision, identifies the protective pattern, constructs your vision and identity statement. Do this first, then revisit monthly or quarterly.

- **Interrupt** (2-3 min) - Daily reality check. Catches you in actual behavior, tests it against your framework, logs the moment. Set phone reminders throughout your day.

- **Synthesis** (10-15 min) - Weekly review. Analyzes your interrupt logs, shows alignment trends, identifies triggers and escape patterns. This is where you see progress.

- **Setup** - Configure where your framework and logs are saved.

### First-Time Setup (no config exists)

After showing overview, proceed to setup:

```
Before we begin, where would you like to save your framework and logs?

1. ~/.claude/identity/ - Global, persists across all projects
2. ./ (current directory) - Good if you have a dedicated repo for this
3. Custom path - Specify your own location
```

After path selection, check if it's a git repo. If not:

```
Would you like to initialize this folder as a git repository?

This lets you:
- Track how your identity framework evolves over time
- See your growth through commit history
- Back up to a private GitHub repo if you want

Requirements:
- Git must be installed on your system

[Yes / No]
```

If yes, run `git init` in the output folder.

Save config to `~/.claude/identity-config.json`.

Then proceed to main menu.

### Main Menu (after overview, whether first-time or returning)

```
What would you like to do?

1. Transformation - Full excavation session (45-60 min)
2. Interrupt - Quick daily check-in (2-3 min)
3. Synthesis - Weekly pattern review (10-15 min)
4. Setup - Change output path or settings
```

Use AskUserQuestion to present these options.

## Workflow Execution

After menu selection (or direct invocation with argument):

1. **Read config** to get outputPath
2. **Load the workflow file** from `workflows/[name].md`
3. **Follow that workflow's instructions**

The workflow files contain the detailed facilitation guidance.

## Setup Workflow

When user selects Setup (or invokes `/identity setup`):

1. Show current configuration:
   ```
   Current settings:
   - Output path: [path]
   - Git initialized: [yes/no]
   - Framework exists: [yes/no]
   - Interrupt log entries: [count]
   ```

2. Ask what they want to change:
   ```
   What would you like to do?
   1. Change output path
   2. Initialize/reinitialize git
   3. Back to main menu
   ```

3. Handle the selection appropriately.

**If changing output path:**
- Ask if they want to move existing files to new location
- Update config
- Optionally initialize git in new location

## File Templates

When creating new framework or interrupt log, use templates from `templates/` folder:
- `templates/framework.md` - structure for identity framework
- `templates/interrupt-log.md` - header for interrupt log

## Error Handling

**No config + direct workflow invocation:**
> "I need to set up Identity first. Let me walk you through a quick setup."
> [Run first-time setup, then proceed to requested workflow]

**Config exists but output path missing/inaccessible:**
> "Your configured output path ([path]) doesn't exist or isn't accessible. Would you like to create it, or choose a new location?"

**Interrupt/Synthesis invoked but no framework exists:**
> "You haven't created an identity framework yet. Would you like to start with a transformation session first?"
