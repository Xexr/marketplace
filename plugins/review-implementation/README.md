# Review Implementation Plugin

Multi-LLM implementation review for verifying code matches spec, identifying gaps, and tracking completion state. Supports Opus, GPT, and Gemini in parallel.

## The Core Principle

Code should match spec. This plugin verifies implementation against requirements, produces a completion matrix showing what's done/partial/missing, and helps you close the gaps.

## Installation

First, add the marketplace:
```
/plugin marketplace add xexr/marketplace
```

Then install the plugin:
```
/plugin install review-implementation@xexr-marketplace
```

## When to Use

- After completing implementation work against a spec
- During implementation to check progress against requirements
- Before creating a PR to verify all requirements are met
- When you need to verify implementation matches original plan

**Sequential with review-documentation:** Review docs first (ensure spec is accurate), then review implementation (ensure code matches spec).

## Requirements

**Required:**
- Claude Code (Opus 4.6 always available)

**Optional (for multi-LLM comparison):**
- [Codex CLI](https://github.com/openai/codex) - for GPT 5.3 Codex reviews
- [Gemini CLI](https://github.com/google/gemini-cli) - for Gemini 3 Pro reviews

## The Process

### Phase 1: Scope & Configuration

Quick setup:
1. Spec scope - what requirements to review against (epic, spec file, task list)
2. Implementation scope - what code to review (branch, directory, files)
3. Which models to use
4. Review all categories or customize

Categories include:
- **Completeness:** Feature completeness, acceptance criteria, task status accuracy
- **Quality:** Edge cases, error handling, test coverage
- **Scope:** No scope creep, approach alignment, dependency completion
- **Standards:** Project conventions, API contract match

### Phase 2: Path Discovery

A fast Haiku agent discovers spec files, implementation files, and test files from your descriptions.

### Phase 3: Parallel Review

Selected models review in parallel (10-15 minutes), comparing implementation against spec.

### Phase 4: Synthesis

Results are merged into:
- **Completion matrix** - requirement-by-requirement status
- Summary stats (X% complete, Y partial, Z missing)
- Risk-weighted issues
- Agent comparison (for multi-model)

### Phase 5: Resolution

Choose actions:
- Create beads issues for gaps
- Update spec to match implementation (if intentional deviation)
- Save review to markdown
- Mark tasks complete in beads
- Fix implementation gaps directly

If beads issues are created, you're asked whether to implement them immediately.

## Preflight Mode (Automation)

Can be invoked with args to skip interactive questions:

```bash
/review-implementation --models opus,gpt --categories all --scope-spec "epic cgt-22" --scope-impl "this branch"
```

Useful for automation from other skills.

## Commands

| Command | Purpose |
|---------|---------|
| `/review-implementation` | Review implementation against spec |

## Output Example

```markdown
# Implementation Review: Epic cgt-22

## Completion Matrix

| Requirement | Status | Location | Notes |
|-------------|--------|----------|-------|
| User login | DONE | src/auth/login.ts:45 | - |
| Password reset | PARTIAL | src/auth/reset.ts:12 | Missing email validation |
| Session timeout | MISSING | - | Not implemented |

## Summary Stats
- **Total Requirements:** 8
- **Complete:** 5 (62%)
- **Partial:** 2
- **Missing:** 1

## Issues (Risk-Weighted)

### HIGH
- Password reset missing email validation
- Session timeout not implemented

### INFORMATIONAL
- Scope creep: Added "remember me" checkbox (not in spec)
```

## What to Expect

The review compares spec against implementation requirement-by-requirement. Approach deviations are noted as informational (not failures) if the outcome still matches spec.

After review, you can create tracking issues for gaps, update specs for intentional deviations, or fix implementation directly.

## License

MIT
