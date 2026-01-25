---
name: design-to-beads
description: Use when converting a design document, PRD, or task list into beads issues - ensures lossless conversion with proper epic hierarchy, validated dependencies for maximum parallelization, and three independent subagent review passes before execution
---

# Design to Beads Conversion

## Overview

Convert design documents into fully-structured beads issues with **lossless information transfer**, proper epic hierarchy, and validated dependencies.

**Core principle:** Every piece of actionable content in the source document must map to a beads issue. Nothing gets lost. Dependencies must enable maximum parallelization. Three independent subagent reviews catch what self-review misses.

## When to Use

- Converting finalized design docs to trackable work
- Breaking down PRDs into implementation tasks
- Creating beads structure from any markdown plan

## When NOT to Use

- Design isn't finalized (use brainstorming skill first)
- Simple single-task work (just create issue directly)

## Critical Rules

**DO NOT rush under time pressure.** Quality conversion takes time. A poorly-structured beads hierarchy causes more delay than taking 30 extra minutes upfront.

**DO NOT skip review passes.** Single-pass creation misses gaps, creates false dependencies, loses information.

**DO NOT self-review.** Launch subagents for review passes. Self-review misses what independent reviewers catch.

**DO NOT create flat issue lists.** Use epics with parent-child relationships.

**DO NOT assume sequential ordering.** Question every dependency: "Does A truly BLOCK B, or could they run in parallel?"

**DO NOT create sparse issues.** If the source has detail, the issue must have detail. Brief descriptions = lost information = implementation failures.

## The Process

### Phase 1: Analyze Input Document

1. Read the entire document
2. Identify natural groupings (phases, features, components) → **epics**
3. Identify actionable items within each grouping → **issues**
4. Note explicit dependencies mentioned in text
5. Note technical details, acceptance criteria, edge cases

### Phase 2: Draft Structure (DO NOT EXECUTE YET)

Create a written draft:
```
Epic: [Name]
  - Issue: [Title]
    Dependencies: [list or "none - can start immediately"]
    Key details: [from source doc]
```

**Coverage matrix (REQUIRED):**
```
Source Section → Beads Location
────────────────────────────────
"Phase 1.1 Stripe Setup" → Epic 1, Issue 1.1
"Technical note: 30s timeout" → Issue 1.1 (acceptance criteria)
```

**Every source section MUST appear.** Gaps = work not captured = failure.

**Detail density tracking (REQUIRED):**
```
Phase/Section     | Source Detail | Beads Detail | Status
──────────────────|───────────────|──────────────|────────
Phase 1 (4 items) | 450 words     | 380 words    | ✓ OK
Phase 2 (6 items) | 600 words     | 120 words    | ⚠️ SPARSE
Phase 3 (3 items) | 200 words     | 190 words    | ✓ OK
```

**Sparse phase = red flag.** If beads detail is <50% of source detail, the conversion lost information. Go back and capture missing details before proceeding.

### Phase 3: Subagent Review Passes (MANDATORY)

**DO NOT self-review.** Launch independent subagents for each pass using the Task tool. Self-review is biased; subagents catch what you miss.

**If Task tool unavailable:** Present full draft to user for manual review at each pass checkpoint. Do NOT substitute self-review for subagent review.

**Pass 1: Completeness + Detail Density Review** (launch subagent with Task tool)

Subagent mandate:
> "Review this beads structure against the source document. Verify: (1) every actionable item has a corresponding issue, (2) every technical note is captured, (3) every acceptance criterion is attached, (4) coverage matrix has no gaps, (5) **detail density check: for each phase, compare word count of source vs beads - flag any phase where beads is <50% of source as SPARSE**. Report any missing content or sparse phases."

**Detail density criteria for each issue:**
- Description must be substantive (>50 words) OR source item was genuinely simple
- Acceptance criteria count should roughly match source complexity
- Technical notes from source preserved verbatim, not summarized
- If source bullet has sub-bullets, issue must capture them all

Apply fixes from Pass 1 before proceeding.

**Pass 2: Dependencies Review** (launch subagent)

Subagent mandate:
> "Review dependencies in this beads structure. Identify: false sequential ordering (could be parallel), missing blockers, incorrect dependency direction, **backward phase dependencies (Phase N task depending on Phase N+M task)**, over-constrained chains. For each dependency, answer: Does A truly BLOCK B from starting?"

**Backward phase dependency = structural error.** If a Phase 1 task depends on a Phase 3 task, something is wrong:
- The phases are ordered incorrectly in the source document
- The task belongs in a different phase
- The dependency is incorrect

When backward phase dependencies are found, ask the user: "Should I update the source spec/design document to fix the phase ordering, or should we adjust the task placement?"

Apply fixes from Pass 2 before proceeding.

**Pass 3: Clarity + Density Review** (launch subagent)

Subagent mandate:
> "Review each issue for implementation readiness. Check: title clarity, description completeness, acceptance criteria verifiability. Flag: vague language, assumed knowledge, missing context. **Also flag any issue where description is <50 words but the corresponding source section had >100 words - this indicates lost detail.** Could a fresh agent execute each issue without referring back to the source document?"

Apply fixes from Pass 3 before proceeding.

**Optional Pass 4: Implementation Readiness** (for complex projects)

Subagent mandate:
> "For each issue: Could a fresh agent with zero codebase context pick this up and execute? Flag: missing file paths, assumed knowledge, vague verbs like 'update the thing'."

### Phase 4: User Checkpoint

**Before executing bd commands, present the final structure to the user:**

```
Ready to create:
- X epics
- Y issues
- Z blocking dependencies
- W items ready immediately (parallel start)

Coverage: All N source sections mapped ✓

Detail Density:
  Phase 1: 450 → 380 words (84%) ✓
  Phase 2: 600 → 550 words (92%) ✓
  Phase 3: 200 → 190 words (95%) ✓
  Overall: 1250 → 1120 words (90%) ✓

Proceed with creation? [User must confirm]
```

**If any phase shows <50% density, DO NOT proceed.** Go back and flesh out the sparse phases before asking for confirmation.

**DO NOT skip this checkpoint.** User approval prevents creating wrong structure.

### Phase 5: Execute

Only after user confirmation:

1. **Create epics first:**
   ```bash
   bd create "Epic title" -t epic -p <priority> -d "Description"
   ```

2. **Create issues with parent-child links:**
   ```bash
   bd create "Issue title" -t task -d "Description" \
     --acceptance "- [ ] Criterion 1"
   bd dep add <epic-id> <issue-id> --type parent-child
   ```

3. **Add blocker dependencies** (only TRUE blockers):
   ```bash
   bd dep add <prerequisite-id> <dependent-id>
   ```

4. **Verify:**
   ```bash
   bd dep cycles          # Must return empty
   bd ready --json        # Check expected items ready
   ```

### Phase 6: Generate Report

**A. Creation Summary**
```
Created: X epics, Y issues
  Epic 1: [Name] (N issues)
  ...
```

**B. Dependency Graph** (show blocking relationships and parallel opportunities)

**C. Ready Work Queue** (items with no blockers)

**D. Coverage Verification**
```
Source sections: N
Mapped to beads: N ✓
Information loss: None
```

## Loophole Closures

**"I'll review it myself to save time"** → WRONG. Self-review is biased. Launch subagents.

**"The review passes are just a formality"** → WRONG. Apply fixes between each pass. Document what changed.

**"User checkpoint slows things down"** → WRONG. Wrong structure wastes more time. Get confirmation.

**"These items obviously need to be sequential"** → WRONG. Prove it. State why A must complete before B can START.

**"I captured the key points"** → WRONG. Capture ALL points. Lossless means lossless.

**"This is simple enough for one pass"** → WRONG. Even simple docs need review for dependencies.

**"Phase 1 legitimately depends on Phase 3"** → WRONG. Backward phase dependencies signal incorrect ordering. Ask user if source doc needs updating.

**"This phase is simpler, it doesn't need much detail"** → WRONG. If the source has detail, capture it. A 600-word source section becoming a 120-word beads issue = 80% information loss. Sparse issues cause implementation failures.

**"I'll add more detail when implementing"** → WRONG. The beads issue IS the specification. If detail isn't in the issue, it's not in the spec. Capture it now or lose it.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Flat issue list | Use epics with parent-child |
| Self-review | Launch subagents |
| Sequential-by-default | Prove each blocker |
| Backward phase dependencies | Reorder phases or move task; offer to update source doc |
| Summarizing details | Preserve exact wording |
| Sparse descriptions (<50% density) | Compare word counts; flesh out before proceeding |
| Skipping user checkpoint | Always get confirmation |
| Rushing for time pressure | Quality over speed |

## Quick Reference

```
1. READ entire source document
2. DRAFT structure with coverage matrix
3. SUBAGENT Pass 1: Completeness → Apply fixes
4. SUBAGENT Pass 2: Dependencies → Apply fixes
5. SUBAGENT Pass 3: Clarity → Apply fixes
6. (Optional) SUBAGENT Pass 4: Implementation readiness
7. USER CHECKPOINT: Present structure, get confirmation
8. EXECUTE bd commands (epics → issues → deps)
9. VERIFY no cycles, expected items ready
10. REPORT summary, graph, queue, coverage
```

**Remember:** Lossless. Epics. Subagent reviews. User checkpoint. Maximum parallelization.
