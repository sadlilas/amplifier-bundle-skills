---
name: simplify
description: "Spawns parallel review agents to analyze recent code changes for simplification opportunities — code reuse, quality, and efficiency. Use /simplify to review, or /simplify <focus> to narrow scope."
context: fork
disable-model-invocation: true
user-invocable: true
model_role: critique
---

# /simplify — Parallel Code Simplification Review

Analyze recent code changes for simplification opportunities across three dimensions: code reuse, code quality, and efficiency. Three parallel review agents run concurrently and their findings are aggregated by impact.

Optional focus area: `$ARGUMENTS`

---

## Step 1 — Identify Recently Changed Files

Run `git diff` to identify the files changed in recent commits or in the working tree:

```bash
# Staged and unstaged changes vs HEAD
git diff HEAD --name-only

# Or for a specific range (e.g., last N commits)
git diff HEAD~3 HEAD --name-only
```

If `$ARGUMENTS` is provided, narrow the scope to files matching that focus area (directory, module name, or glob pattern). Collect the list of changed files and their diffs for review.

---

## Step 2 — Spawn 3 Parallel Review Agents

Launch the following three agents concurrently. Each agent receives:
- The list of changed files
- The full diff content
- Their specific review instructions below

### Agent A — Code Reuse Review

**Mission:** Find duplication and missed reuse opportunities.

Review the changed files for:
- Duplicated logic that could be extracted into shared helpers or utilities
- Copy-paste patterns across files or functions
- Inline logic that already exists in a library or module elsewhere in the codebase
- Similar data transformations that could be unified
- Constants or configuration repeated across files

For each finding, report:
- **Location**: file and line range
- **Pattern**: what is duplicated or could be reused
- **Suggestion**: how to extract or reference the shared logic
- **Impact**: High / Medium / Low

---

### Agent B — Code Quality Review

**Mission:** Find readability, maintainability, and correctness issues introduced in the diff.

Review the changed files for:
- Functions or methods that do too many things (single responsibility violations)
- Unclear variable or function names that hurt readability
- Missing or incorrect error handling
- Dead code, unused imports, or commented-out code left behind
- Magic numbers or unexplained constants
- Overly complex conditionals that could be simplified
- Missing type annotations (in typed codebases)

For each finding, report:
- **Location**: file and line range
- **Issue**: what the quality problem is
- **Suggestion**: the simpler or clearer alternative
- **Impact**: High / Medium / Low

---

### Agent C — Efficiency Review

**Mission:** Find performance, resource, and algorithmic inefficiencies.

Review the changed files for:
- Unnecessary loops, repeated iterations, or O(n²) patterns where O(n) is possible
- Redundant I/O operations (repeated file reads, repeated API calls, repeated DB queries)
- Objects or resources allocated inside loops that could be hoisted
- Missing caching for expensive or repeated computations
- Inefficient data structure choices (e.g., list when set lookup is needed)
- Blocking operations that could be made async

For each finding, report:
- **Location**: file and line range
- **Inefficiency**: what the performance issue is
- **Suggestion**: the more efficient alternative
- **Impact**: High / Medium / Low

---

## Step 3 — Aggregate and Apply Findings

Collect the output from all three agents. Deduplicate overlapping findings and sort by impact (High → Medium → Low).

Present a unified report:

```
## Simplification Opportunities

### High Impact
- [finding source: Reuse/Quality/Efficiency] file.py:L42-L67 — ...

### Medium Impact
- ...

### Low Impact
- ...
```

For any High-impact finding, propose the specific code change. Apply fixes with user confirmation, starting with the highest-impact items first.

If `$ARGUMENTS` was provided, confirm that the review was scoped to: `$ARGUMENTS`
