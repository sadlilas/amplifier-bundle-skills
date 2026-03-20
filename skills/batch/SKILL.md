---
name: batch
description: "Decomposes a large change into independent work units and executes them in parallel via delegate agents. Use /batch <description of change> to start."
context: fork
disable-model-invocation: true
user-invocable: true
model_role: reasoning
---

# /batch — Parallel Batch Execution

Decompose a large change into independent work units and execute them in parallel via delegate agents.

Change description: `$ARGUMENTS`

---

## Step 1 — Understand the Change

Research the codebase to fully understand the scope of the requested change: `$ARGUMENTS`

- Explore the relevant directories, files, and modules affected
- Identify the existing patterns, conventions, and test structure in use
- Understand dependencies between components
- Clarify any ambiguities before proceeding to decomposition

Use tools like file exploration, grep, and code reading to build a complete picture of what needs to change.

---

## Step 2 — Decompose into Independent Work Units

Break the change into **5–30 independent work units**. Each work unit must:

- Be **completable in isolation** — no runtime dependencies on other units in this batch
- Have a **clear, bounded scope** — one module, one feature, one concern
- **Include tests** — each unit delivers both implementation and test coverage
- Be **named clearly** — use a short kebab-case identifier (e.g., `add-user-validation`, `refactor-cache-layer`)

For each work unit, define:
- **Name**: short kebab-case identifier used for the branch name
- **Files**: list of files to create or modify
- **Description**: what the unit accomplishes and why
- **Complexity**: Low / Medium / High (estimated effort)
- **Dependencies**: any other units that must complete before this one (keep these minimal)

If the change cannot be decomposed into independent units (tight coupling, required ordering), restructure until it can, or reduce scope.

---

## Step 3 — Present Plan and Request Approval

Present the decomposition as a numbered list before executing anything:

```
## Batch Plan: <change description>

Total work units: N

1. **<unit-name>** [Complexity: Low/Medium/High]
   Files: path/to/file.py, path/to/test_file.py
   Description: <what this unit does>

2. **<unit-name>** [Complexity: Low/Medium/High]
   Files: ...
   Description: ...

...
```

Ask: **"Does this decomposition look correct? Approve to execute all units in parallel, or provide feedback to adjust."**

Wait for explicit approval before proceeding to Step 4.

---

## Step 4 — Execute in Parallel via Delegate Agents

Upon approval, launch all work units concurrently. For each unit:

1. Create a git branch named `batch/<unit-name>`
2. Delegate to a `foundation:modular-builder` agent with:
   - The full unit specification (files, description, acceptance criteria)
   - Instruction to work on branch `batch/<unit-name>`
   - TDD requirement: write failing test first, then implement
3. Each agent works independently on its branch

Launch all agents simultaneously — do not wait for one to finish before starting the next.

---

## Step 5 — Integration Summary

After all agents complete, produce an integration summary:

```
## Batch Execution Summary

### Completed Successfully
- **<unit-name>** (branch: batch/<unit-name>) — <brief outcome>
- ...

### Failed or Incomplete
- **<unit-name>** — <what went wrong>
- ...

### Next Steps
- Review and merge completed branches
- Address any failures
- Run full test suite after merging all branches
```

Report the count of successes vs. failures, highlight any cross-unit conflicts to resolve during merge, and recommend the merge order for units with declared dependencies.
