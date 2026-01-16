# Identity Plugin

A system for intentional identity change based on [this insightful article by Dan Koe](https://letters.thedankoe.com/p/how-to-fix-your-entire-life-in-1).

## The Core Insight

Identity precedes behavior. You don't change what you do and hope to become someone different - you become someone different and your actions follow.

All behavior is goal-oriented, including self-sabotage. The patterns you want to break are *protecting* something. This system helps you surface what, then reconstruct around what you actually want.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install identity@xexr-marketplace
```

## How It Works

The plugin creates a three-part feedback loop:

```
transformation → framework.md
      ↑              ↓
synthesis ←←← interrupt-log.md ←←← interrupts
```

### 1. Transformation (45-60 min)

A guided excavation session. Do this first, then revisit monthly or quarterly.

**What happens:**
- You write freely about dissatisfactions, patterns, and the gap between who you are and who you want to be
- Claude probes beneath surface answers - "What's underneath that?" "What does your behavior reveal you actually want?"
- You build a vivid anti-vision of the future you're avoiding (5 years, 10 years, end of life)
- You surface the protective identity keeping you stuck - what it protects and what that protection costs
- You pivot to construct your vision and identity statement
- You expand beyond the presenting issue to your whole self - roles, relationships, health, learning

**Output:** A personal framework document containing:
- Anti-vision statement (one brutal sentence)
- The pattern you're breaking
- Core identity statement
- Role-specific identities (as a builder, parent, partner, friend, etc.)
- Vision MVP
- Goal framework (one year, one month, daily actions)

### 2. Interrupt (2-3 min)

Quick reality checks throughout your day. Set phone reminders to invoke `/identity interrupt` at key points.

**What happens:**
- Claude asks one catch question: "What are you doing right now, and what are you avoiding by doing it?"
- You answer honestly
- Claude reflects your own framework back at you: "You said you're the type of person who... Does this serve that?"
- You decide what the person you're becoming would do
- The moment gets logged

**Why it matters:**
Each interrupt catches you in actual behavior - not what you think you do, but what you actually do. Over time, patterns emerge that you can't see from inside them.

### 3. Synthesis (10-15 min)

Weekly review that transforms interrupt logs into visible progress.

**What happens:**
- Claude analyzes your logged interrupts
- Shows alignment rate and trend over time
- Identifies triggers (when does your old pattern show up?)
- Identifies escape routes (how does the pattern manifest?)
- Tests whether your identified pattern was accurate
- Suggests framework refinements based on real data
- Gives you one focus for the coming week

**Why it matters:**
Without synthesis, you're just logging. With it, you're learning. This is where daily discipline becomes visible progress.

## Commands

| Command | Duration | Purpose |
|---------|----------|---------|
| `/identity` | - | Show overview and menu |
| `/identity transformation` | 45-60 min | Deep excavation session |
| `/identity interrupt` | 2-3 min | Daily reality check |
| `/identity synthesis` | 10-15 min | Weekly pattern review |
| `/identity setup` | 2 min | Configure output path |

## First Run

On first use, you'll configure where to save your framework and logs:
- `~/.claude/identity/` (default, global)
- Current directory (if you have a dedicated repo)
- Custom path

You can optionally initialize git to track your evolution over time.

## What to Expect

**The transformation session is uncomfortable by design.** The anti-vision work makes the cost of not changing viscerally real. You'll describe futures you're avoiding, name fears you'd rather not admit, and surface beliefs you've been protecting. That discomfort is the engine for change.

**The interrupts are where the real work happens.** Daily friction against the old pattern. Each one is small, but they accumulate into data that reveals what you actually do versus what you think you do.

**The synthesis is where you see progress.** Alignment trends, identified triggers, patterns you couldn't see from inside. The framework evolves based on evidence, not assumptions.

This isn't a productivity hack. It's identity work with a feedback loop.

## License

MIT
