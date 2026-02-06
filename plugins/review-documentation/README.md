# Review Documentation Plugin

Multi-LLM documentation review for catching inconsistencies, codebase mismatches, and gaps. Supports Opus, GPT, and Gemini in parallel with synthesis and guided resolution.

## The Core Principle

One LLM misses things. Multiple LLMs catch each other's blind spots. This plugin dispatches up to three models (Opus, GPT, Gemini) to review your documentation in parallel, then synthesizes their findings into a single actionable report.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install review-documentation@xexr-marketplace
```

## When to Use

- After an LLM generates or updates documentation
- Before implementing a spec or task list
- When reviewing PRDs, plans, or beads issue epics
- When documentation references code that may have changed

## When NOT to Use

- Simple typo fixes (just fix them)
- Documentation you haven't written yet (write first, review after)

## Requirements

**Required:**
- Claude Code (Opus 4.6 always available)

**Optional (for multi-LLM comparison):**
- [Codex CLI](https://github.com/openai/codex) - for GPT 5.3 Codex reviews
- [Gemini CLI](https://github.com/google/gemini-cli) - for Gemini 3 Pro reviews

The plugin works with just Opus, but cross-validation with multiple models catches more issues.

## The Process

### Phase 1: Scope & Configuration

Quick setup questions:
1. What documentation to review (free text description)
2. Which models to use (Opus, GPT, Gemini)
3. Review all categories or customize

Categories include:
- **Accuracy:** Codebase match, cross-document consistency, API assumptions, security
- **Design:** Design quality, TDD alignment, project standards, architectural consistency
- **Robustness:** Error handling, performance, data/schema validity, task dependencies

### Phase 2: Path Discovery

A fast Haiku agent discovers relevant files and beads issues from your scope description. You don't need to list specific paths.

### Phase 3: Parallel Review

Selected models review in parallel (10-15 minutes). Each receives:
- The dynamically-built review brief
- Pre-read beads issue contents
- Access to explore the codebase

### Phase 4: Synthesis

Results are merged into:
- Deduplicated issues by severity (CRITICAL > HIGH > MEDIUM > LOW)
- Agent comparison table (for multi-model reviews)
- Reasoning for each finding
- Remaining ambiguities requiring human input

### Phase 5: Resolution

After reviewing findings, you choose actions:
- Create beads issues for tracking
- Save review to markdown
- Fix documentation directly
- Fix code issues (if identified)

## Commands

| Command | Purpose |
|---------|---------|
| `/review-documentation` | Review documentation for issues |

## Output Example

```markdown
# Documentation Review: Phase 5 Epic

## Synthesized Issues (Risk-Weighted)

### CRITICAL
- File path `src/legacy/adapter.ts` doesn't exist (renamed to `src/adapters/legacy.ts`)

### HIGH
- Spec mentions "batch processing" but codebase only handles single items
- Task cgt-45 marked complete but implementation is partial

### MEDIUM
- Cross-document inconsistency: README says v2 API, spec says v3

## Agent Comparison

| # | Issue | Opus | GPT | Agree? | Recommendation |
|---|-------|------|-----|--------|----------------|
| 1 | Missing file | HIGH | HIGH | Yes | Update path |
| 2 | Batch support | HIGH | MEDIUM | Partial | Verify scope |

## Summary
- **Total Issues:** 4 (1 critical, 2 high, 1 medium)
- **Agent Agreement Rate:** 75%
```

## What to Expect

The review phase takes 10-15 minutes when using multiple models - they're doing thorough work. The synthesis provides a single actionable list, not three separate outputs to reconcile yourself.

The guided resolution in Phase 5 prevents review findings from becoming stale - you can create tracking issues, save the review, or fix things immediately.

## License

MIT
