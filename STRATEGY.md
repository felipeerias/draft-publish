# Draft-Publish Feature Development Strategy

Instructions for a coding agent implementing a complex multi-change feature.


## Overview

This document describes a strategy for developing complex features distributed over multiple upstream changes.

The Draft phase breaks down the problem in concrete tasks and creates an initial implementation for them, building a detailed understanding of what the complete solution will look like. During Drafting, expect tasks to split, merge, reorder, or replace as the real problem emerges. Tasks may be renumbered only during initial Planning; task numbers stabilize once Drafting begins because other sections reference them; gaps from merges or deletions are fine.

The Publish phase works with the user to define and prepare each change to be submitted upstream. A *change* in this context is a single reviewable unit: a CL, PR, or patch, depending on the project's workflow.

The *feature plan document* is a separate output file, independent of any other internal planning tools. It is stored in `plans/{feature}.md`. This is a living document that is updated and refined as development progresses. Changes to this document are committed with explanations of what changed.


## Files

```
draft-publish-strategy/
  STRATEGY.md         // this file
  plans/{feature}.md  // one *feature plan document* per feature
```


## Branches

Draft branches form a chain:

```
main → {feature}-draft-01 → {feature}-draft-02 → ...
```

```bash
git diff {feature}-draft-02..{feature}-draft-03  # one task
git diff main..{feature}-draft-03               # cumulative
```

Publish branches are named `{feature}-NN` and start fresh from upstream `main`.


## Sessions

### New feature

1. Read `STRATEGY.md`.
1. Read the spec and related documentation.
1. Identify scope of proposed feature, as well as what is deferred; explore related code patterns.
1. Create `plans/{feature}.md` following the *feature plan document* format below. Commit this file.
1. Summarize to user: task groups, risks, spec ambiguities.
1. Revise based on feedback; renumber tasks freely before Drafting starts.

### Continue feature

1. Read `STRATEGY.md`.
1. Read `plans/{feature}.md`.
1. Inform the user of where things stand and what you'll do next.
1. Proceed with the next task.

### Review feedback

The user returns with feedback on a task (during Draft or Publish).

1. Read `STRATEGY.md`.
1. Read `plans/{feature}.md`
1. Address the feedback in code, updating the associated Draft or Publish branch.
1. Revise downstream tasks and update the plan as needed.
1. If the feedback reveals a durable constraint, update Lessons.


## Draft phase

Goal: see the full solution end to end. Fast, not review-ready.

For each task:

1. Branch from previous task draft (or `main` for task 01), write the change, commit with rationale.
1. Build and run relevant tests.
1. Re-read the diff; verify acceptance criteria.
1. Capture lessons learned: if you discovered something non-obvious, append it to Lessons immediately. Tag with task number. These help future sessions avoid re-discovering the same things.
1. Update the *feature plan document*:
   - Status → draft-done
   - Record results summary
   - Update Architecture as needed
   - Review and revise downstream tasks if affected
   - Advance Current task
1. Report to user: what you did, what you learned, concerns, uncertainties.

Keep each change focused on one area of the codebase. A change that crosses subsystem or component boundaries should be split into separate tasks.

If blocked or wrong, stop. Update the task and tell the user.


## Publish phase

Goal: prepare each change for submission—upstream-based, self-contained, review-ready.

### Changeset planning

Before preparing individual changes, work with the user to determine how Draft tasks map to Publish upstream changesets:

1. Review all completed draft tasks and their dependencies.
2. Identify grouping opportunities.
3. Check for dependencies and ordering constraints.
4. Define which tasks combine into each changeset.
5. Document the mapping in the *feature plan document*.

Provide multiple planning options to the user and explain the implications of each.

### Preparing a changeset

For each changeset:

1. Read the Draft branch and other relevant information in the plan.
1. Sync with upstream; check for conflicts or relevant changes.
1. Create a fresh branch from upstream HEAD, named `{feature}-{description}`.
1. Implement the change carefully and thoroughly. Use the draft as reference, not as code to copy.
1. Review and iterate on your work.
1. Write a changeset description: component prefix, what and why, issue reference.
1. Update the plan: Status → in-review, record the change URL.
1. Tell the user the change is ready.

### Review and landing

1. Address feedback on the same branch.
1. If feedback affects other tasks, update Lessons and flag them.
1. When landed: Status → landed, review downstream impact, delete the Publish branch.
1. Throughout this workflow, work with the user to evaluate open options and decide next steps.


## Feature plan document format

```markdown
# Feature: {name}

Spec: {URL or description}
Phase: Planning | Draft | Publish | Complete
Current task: {NN} — {title}

## Summary

{2-3 sentences. Enough for a fresh session to understand the scope.}

## Scope

- In: {what this plan covers}
- Out: {what is deferred and why}
- Open: {ambiguities in spec or requirements}

## Architecture

{How the solution works: subsystems, interactions, key decisions.}

## Lessons

{Cross-cutting knowledge. Anything non-obvious, like an undocumented constraint, a required workaround, or a useful technique. Append-only. Tag each entry with its source.}

- (task 01) ...
- (task 03, reviewer) ...

## Tasks

### Task flow

{ASCII dependency graph showing independent and dependent tasks}

Draft chain: {linear order for the Draft branch chain}

Tasks are grouped into units that can be enabled or shipped together (e.g., feature flags, modules, API surfaces). Groups may depend on other groups.

### Group: {name}
Depends on: {other groups, or —}
{One sentence: what enabling or shipping this group gives you.}

#### 01 — {Title}
- Status: pending | in-progress | draft-done | in-review | landed | blocked
- Depends on: —
- Change: {URL when published}
- Description: {what to do, which files, which patterns to follow}
- Acceptance: {concrete, testable criteria}
- Rationale: {why this task exists, why in this position}

#### 02 — {Title}
...

### Group: {name}
Depends on: {previous group}

#### 03 — {Title}
...

## Open concerns

{Unresolved questions or risks. Remove when resolved.}
```
