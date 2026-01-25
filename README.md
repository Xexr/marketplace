# xexr's Claude Code Marketplace

Personal plugins and skills for Claude Code.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add xexr/marketplace
```

## Available Plugins

### Identity

A system for intentional identity change based on [this insightful article by Dan Koe](https://letters.thedankoe.com/p/how-to-fix-your-entire-life-in-1).

**Install:**
```
/plugin install identity@xexr-marketplace
```

#### The Core Insight

Identity precedes behavior. You don't change what you do and hope to become someone different - you become someone different and your actions follow.

All behavior is goal-oriented, including self-sabotage. The patterns you want to break are *protecting* something. This system helps you surface what, then reconstruct around what you actually want.

#### How It Works

The plugin creates a three-part feedback loop:

```
transformation → framework.md
      ↑              ↓
synthesis ←←← interrupt-log.md ←←← interrupts
```

**1. Transformation** (45-60 min, monthly/quarterly)

A guided excavation session. You'll write freely about dissatisfactions, patterns, and gaps. Claude probes beneath surface answers, builds a vivid anti-vision of the future you're avoiding, surfaces the protective identity keeping you stuck, then pivots to construct your vision and identity statement.

Output: A personal framework document with your anti-vision, the pattern you're breaking, core identity statement, role-specific identities, and concrete daily actions.

**2. Interrupt** (2-3 min, daily)

Quick reality checks throughout your day. Claude asks what you're doing and what you're avoiding, reflects your own framework back at you, and logs the moment.

Set phone reminders to invoke `/identity interrupt` at key points in your day. Each interrupt catches you in actual behavior - not what you think you do, but what you actually do.

**3. Synthesis** (10-15 min, weekly)

Analyzes your interrupt logs to surface patterns you can't see from inside them:
- When does your old pattern show up? (time of day, triggers, situations)
- What are your escape routes? (specific avoidance behaviors)
- What conditions make alignment easier?

Shows your alignment trend over time, tests whether your identified pattern was accurate, and suggests framework refinements based on real data.

#### Commands

| Command | Duration | Purpose |
|---------|----------|---------|
| `/identity` | - | Show menu and overview |
| `/identity transformation` | 45-60 min | Deep excavation session |
| `/identity interrupt` | 2-3 min | Daily reality check |
| `/identity synthesis` | 10-15 min | Weekly pattern review |
| `/identity setup` | 2 min | Configure output path |

#### First Run

On first use, you'll configure where to save your framework and logs. You can optionally initialize git to track your evolution over time.

#### What to Expect

The transformation session is uncomfortable by design. The anti-vision work makes the cost of not changing viscerally real. That discomfort is the engine for change.

The interrupts are where the real work happens - daily friction against the old pattern. The synthesis is where you see progress and refine your understanding.

This isn't a productivity hack. It's identity work with a feedback loop.

---

### Design to Beads

Convert design documents, PRDs, and task lists into beads issues with lossless conversion, proper epic hierarchy, and validated dependencies.

**Install:**
```
/plugin install design-to-beads@xexr-marketplace
```

#### The Core Principle

Every piece of actionable content in the source document must map to a beads issue. Nothing gets lost. Dependencies must enable maximum parallelization. Three independent subagent reviews catch what self-review misses.

#### How It Works

The plugin follows a rigorous six-phase process:

1. **Analyze** - Read the entire document, identify epics (phases, features, components) and issues (actionable items)

2. **Draft** - Create a written structure with a coverage matrix ensuring every source section maps to a beads location. Track detail density to catch information loss.

3. **Review** - Three mandatory subagent review passes:
   - Pass 1: Completeness + detail density
   - Pass 2: Dependencies (catch false sequencing, backward phase deps)
   - Pass 3: Clarity + implementation readiness

4. **Checkpoint** - Present final structure for user approval before creating anything

5. **Execute** - Create epics, then issues with parent-child links, then blocker dependencies

6. **Report** - Summary, dependency graph, ready work queue, coverage verification

#### Critical Rules

- DO NOT rush - quality conversion prevents downstream delays
- DO NOT skip review passes - single-pass creation misses gaps
- DO NOT self-review - launch subagents for each pass
- DO NOT assume sequential ordering - prove each blocker
- DO NOT create sparse issues - if source has detail, issue must have detail

#### Commands

| Command | Purpose |
|---------|---------|
| `/design-to-beads` | Convert a design document to beads issues |

## License

MIT
