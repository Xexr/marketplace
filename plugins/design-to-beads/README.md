# Design to Beads Plugin

Convert design documents, PRDs, and task lists into beads issues with lossless conversion, proper epic hierarchy, and validated dependencies.

## The Core Principle

Every piece of actionable content in the source document must map to a beads issue. Nothing gets lost. Dependencies must enable maximum parallelization. Three independent subagent reviews catch what self-review misses.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install design-to-beads@xexr-marketplace
```

## When to Use

- Converting finalized design docs to trackable work
- Breaking down PRDs into implementation tasks
- Creating beads structure from any markdown plan

## When NOT to Use

- Design isn't finalized (use brainstorming first)
- Simple single-task work (just create an issue directly)

## The Process

### Phase 1: Analyze

Read the entire document, identify natural groupings (phases, features, components) as epics, and actionable items within each grouping as issues.

### Phase 2: Draft Structure

Create a written draft with a **coverage matrix** ensuring every source section maps to a beads location. Track **detail density** - if beads detail is <50% of source detail, the conversion lost information.

### Phase 3: Subagent Review Passes (Mandatory)

Three independent subagent reviews catch what self-review misses:

| Pass | Focus | Subagent Mandate |
|------|-------|------------------|
| 1 | Completeness + Density | Every actionable item covered? Detail preserved? |
| 2 | Dependencies | False sequencing? Missing blockers? Backward phase deps? |
| 3 | Clarity + Density | Implementation ready? Vague language? Lost detail? |

Fixes are applied between each pass.

### Phase 4: User Checkpoint

Present the final structure for user approval before creating any beads issues. Shows epic/issue counts, dependency counts, items ready for parallel start, and detail density metrics.

### Phase 5: Execute

Only after user confirmation:
1. Create epics first
2. Create issues with parent-child links
3. Add blocker dependencies (only TRUE blockers)
4. Verify no cycles and expected items ready

### Phase 6: Report

Generate creation summary, dependency graph, ready work queue, and coverage verification.

## Critical Rules

- **DO NOT rush under time pressure.** Quality conversion takes time.
- **DO NOT skip review passes.** Single-pass creation misses gaps.
- **DO NOT self-review.** Launch subagents for each pass.
- **DO NOT create flat issue lists.** Use epics with parent-child relationships.
- **DO NOT assume sequential ordering.** Question every dependency.
- **DO NOT create sparse issues.** If the source has detail, the issue must have detail.

## Commands

| Command | Purpose |
|---------|---------|
| `/design-to-beads` | Convert a design document to beads issues |

## What to Expect

The process is thorough by design. A poorly-structured beads hierarchy causes more delay than taking extra time upfront. The three review passes and user checkpoint prevent creating the wrong structure.

The output is a set of beads issues that:
- Capture all information from the source document
- Have proper epic hierarchy
- Have validated dependencies enabling maximum parallelization
- Are implementation-ready without referring back to the source

## License

MIT
