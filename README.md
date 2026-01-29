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

---

### Review Documentation

Multi-LLM documentation review for catching inconsistencies, codebase mismatches, and gaps. Supports Opus, GPT, and Gemini in parallel with synthesis and guided resolution.

**Install:**
```
/plugin install review-documentation@xexr-marketplace
```

#### The Core Principle

One LLM misses things. Multiple LLMs catch each other's blind spots. This plugin dispatches up to three models (Opus, GPT, Gemini) to review your documentation in parallel, then synthesizes their findings into a single actionable report.

#### How It Works

The plugin follows a five-phase process:

1. **Scope & Configuration** - Quick questions: what to review, which models, which categories (accuracy, design, robustness)

2. **Path Discovery** - A fast Haiku agent discovers relevant files and beads issues from your description

3. **Parallel Review** - Selected models review simultaneously (10-15 min). Each gets the review brief, pre-read beads issues, and codebase access

4. **Synthesis** - Results merged into:
   - Deduplicated issues by severity
   - Agent comparison table
   - Reasoning for each finding
   - Ambiguities requiring human input

5. **Resolution** - Choose actions: create beads issues, save to markdown, fix docs, or fix code

#### Requirements

- Claude Code (Opus 4.5 always available)
- Optional: [Codex CLI](https://github.com/openai/codex) for GPT reviews
- Optional: [Gemini CLI](https://github.com/google/gemini-cli) for Gemini reviews

Works with just Opus, but cross-validation catches more issues.

#### Commands

| Command | Purpose |
|---------|---------|
| `/review-documentation` | Review documentation for issues |

---

### Review Implementation

Multi-LLM implementation review for verifying code matches spec, identifying gaps, and tracking completion state. Supports Opus, GPT, and Gemini in parallel.

**Install:**
```
/plugin install review-implementation@xexr-marketplace
```

#### The Core Principle

Code should match spec. This plugin verifies implementation against requirements, produces a completion matrix showing what's done/partial/missing, and helps you close the gaps.

#### How It Works

The plugin follows a five-phase process:

1. **Scope & Configuration** - Define spec scope (epic, spec file) and implementation scope (branch, directory), select models and categories

2. **Path Discovery** - Haiku discovers spec files, implementation files, and test files

3. **Parallel Review** - Models compare implementation against spec requirement-by-requirement

4. **Synthesis** - Results merged into completion matrix, summary stats, and risk-weighted issues

5. **Resolution** - Create tracking issues, update specs, mark tasks complete, or fix gaps directly

#### Sequential with Review Documentation

Use these plugins together:
1. `/review-documentation` - Ensure spec is accurate
2. `/review-implementation` - Ensure code matches spec

#### Preflight Mode

Can be invoked with args for automation:
```bash
/review-implementation --models opus,gpt --categories all --scope-spec "epic cgt-22" --scope-impl "this branch"
```

#### Commands

| Command | Purpose |
|---------|---------|
| `/review-implementation` | Review implementation against spec |

## License

MIT
