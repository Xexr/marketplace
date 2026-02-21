# xexr's Claude Code Marketplace

Personal plugins and skills for Claude Code.

## Overview

| Plugin | Description |
|--------|-------------|
| [Identity](#identity) | A system for intentional identity change through transformation sessions, daily interrupts, and weekly synthesis. |
| [Design to Beads](#design-to-beads) | Convert design documents and PRDs into beads issues with lossless conversion and validated dependencies. |
| [Review Documentation](#review-documentation) | Multi-LLM documentation review for catching inconsistencies and codebase mismatches. |
| [Review Implementation](#review-implementation) | Multi-LLM implementation review for verifying code matches spec and tracking completion. |
| [Beads Upstream Sync](#beads-upstream-sync) | Fork maintenance for beads - upstream sync, cherry-pick management, and branch cleanup. |
| [Gastown Upstream Sync](#gastown-upstream-sync) | Fork maintenance for gastown - upstream sync, cherry-pick management, formula sync, and branch cleanup. |
| [Epic Delivery](#epic-delivery) | Deliver beads epics through polecats and the refinery with wave-based dispatch, monitoring, and quality gates. |
| [Gastown Rig Setup](#gastown-rig-setup) | Add Git repos as Gas Town rigs with Dolt-native beads, including .beads/ seeding workaround. |

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

### Epic Delivery

Orchestrate full delivery of a beads epic through polecats, refinery, and integration branches. The crew member acts as dispatcher and monitor; polecats do the implementation work.

**Install:**
```
/plugin install epic-delivery@xexr-marketplace
```

#### Where This Fits

Epic delivery is the final stage of the design-to-delivery pipeline. The earlier stages live in the [gt-toolkit formulas](https://github.com/Xexr/gt-toolkit/tree/main/formulas):

| Stage | Tool | Output |
|-------|------|--------|
| 1-4 | `spec-workflow` formula | `spec.md` |
| 5 | `plan-writing` formula | `plan.md` |
| 6 | `plan-review-to-spec` formula | `plan-review.md` |
| 7 | `beads-creation` formula | Beads epic with validated dependencies |
| 8 | `beads-review-to-plan` formula | `beads-review.md` |
| **9** | **`epic-delivery` plugin** | **Delivered code on integration branch** |

#### How It Works

The plugin follows a four-phase process:

1. **Setup** - Create or reuse integration branch and convoy. Gather all leaf beads (tasks, bugs, features, chores) for tracking.

2. **Dispatch** - Find unblocked leaves via `bd ready`, sling to polecats respecting `max_polecats` limits. Dispatches in waves as dependencies resolve.

3. **Monitor** - Check integration branch status (MR merges) and convoy progress at 2-minute intervals. Dispatch newly unblocked work when dependencies land. Escalate after 5 no-change cycles. Handoff to fresh session after 15 cycles.

4. **Validation** - Verify integration branch, run quality gates (setup, types, lint, build, test), handle failures via bug-fix sub-epics, produce plan-vs-actual summary, and offer deep review via `review-implementation`.

#### Commands

| Command | Purpose |
|---------|---------|
| `/epic-delivery` | Deliver a beads epic through polecats and the refinery |

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

- Claude Code (Opus 4.6 always available)
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

---

### Beads Upstream Sync

Fork maintenance for your beads fork (steveyegge/beads) - upstream sync, cherry-pick management, binary rebuild, and branch cleanup.

**Install:**
```
/plugin install beads-upstream-sync@xexr-marketplace
```

#### The Core Principle

PR branches are the source of truth. Cherry-picks on main are copies. Conflicts are always resolved on the PR branch first, then the fixed version is brought back to main.

#### Auto-Detection

The skill auto-detects your environment before starting:

| Value | Detection Method |
|-------|-----------------|
| Fork owner | Parsed from `git remote get-url origin` |
| Crew name | Parsed from current working directory |
| DCG installed | `command -v dcg` |

You confirm or correct these values before the sync begins.

#### How It Works

An 11-step process covering the full sync lifecycle:

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
11. **Doctor & Summarize** - Post-upgrade checks and upstream change report

#### Commands

| Command | Purpose |
|---------|---------|
| `/beads-upstream-sync` | Sync your beads fork with upstream |

---

### Gastown Upstream Sync

Fork maintenance for your gastown fork (steveyegge/gastown) - upstream sync, cherry-pick management, binary rebuild, formula sync, and branch cleanup.

**Install:**
```
/plugin install gastown-upstream-sync@xexr-marketplace
```

#### The Core Principle

PR branches are the source of truth. Cherry-picks on main are copies. Conflicts are always resolved on the PR branch first, then the fixed version is brought back to main.

#### Auto-Detection

The skill auto-detects your environment before starting:

| Value | Detection Method |
|-------|-----------------|
| Fork owner | Parsed from `git remote get-url origin` |
| Crew name | Parsed from current working directory |
| DCG installed | `command -v dcg` |

You confirm or correct these values before the sync begins.

#### How It Works

An 11-step process covering the full sync lifecycle:

1. **Discovery** - Detect fork owner, crew name, DCG presence
2. **Fetch & Analyze** - Compare upstream, identify merged/unmerged PRs
3. **Rebase Main** - Drop merged cherry-picks, replay unmerged ones
4. **Rebase PR Branches** - Update active branches against upstream
5. **Build & Test** - Catch compilation errors, package-level collisions, formula drift
6. **Checkpoint** - Full summary for user approval before any push
7. **Push** - Main and PR branches (force-with-lease where needed)
8. **Rebuild Binary** - `make install` and verify with `gt version`
9. **Update Clones** - Sync mayor and refinery rig directories
10. **Clean Up** - Delete merged branches, prune stale refs
11. **Formulas, Doctor & Summarize** - Sync formulas, post-upgrade checks, upstream change report

#### Commands

| Command | Purpose |
|---------|---------|
| `/gastown-upstream-sync` | Sync your gastown fork with upstream |

---

### Gastown Rig Setup

Add Git repositories as Gas Town rigs with Dolt-native beads integration. Handles `.beads/` seeding, rig creation, Dolt initialization, and remote configuration.

**Install:**
```
/plugin install gastown-rig-setup@xexr-marketplace
```

#### The Core Principle

`.beads/` must be seeded in the upstream repo before running `gt rig add`. Without seeding, `bd init` writes a mismatched database name (`beads_<prefix>` instead of the actual rig name), breaking Dolt connectivity.

#### How It Works

A 9-step process covering the full rig onboarding lifecycle:

1. **Clone** - Clone the repo into `~/projects/`
2. **Seed `.beads/`** - Create minimal `.beads/` with correct `dolt_database` value in `metadata.json`
3. **Commit & Push** - Push `.beads/` so `gt rig add` clones pick it up
4. **Add Rig** - `gt rig add <name> <url> --prefix <prefix>`
5. **Init Dolt** - `gt dolt init-rig <name>` + restart Dolt server
6. **Configure Remote** - Set up local Dolt remote for the new database
7. **Verify Config** - Check `config.json`, `.beads/redirect`, `metadata.json`
8. **Run Diagnostics** - `bd doctor` and `gt doctor`
9. **Verification Checklist** - Confirm rig list, dolt list, bd list, remotes

#### Known Issues

**Database naming bug (why seeding is required):** The unseeded code path in `gt rig add` triggers `bd init --server` which writes `dolt_database: "beads_<prefix>"`, but the actual Dolt database is named `<rigName>`. This mismatch prevents `bd` from connecting. Seeding `.beads/` with the correct `dolt_database` value prevents this bug.

**`bd doctor` false positive:** In server mode, `bd doctor` reports `No dolt database found` because it checks for a local `.beads/dolt/` directory that doesn't exist with the centralized Dolt server. This is cosmetic only. Use `bd list` to verify actual connectivity.

#### Commands

| Command | Purpose |
|---------|---------|
| `/gastown-rig-setup` | Add a new Git repository as a Gas Town rig |

## License

MIT
