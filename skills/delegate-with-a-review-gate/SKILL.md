---
name: delegate-with-a-review-gate
description: "Use when implementation work is handed to other agents, tools or contributors while one reviewer keeps commit and push rights: delegate only scoped tasks with acceptance checks, write a self-contained brief, isolate each task on its own branch, treat every handoff claim as unverified, and commit only what was reviewed and approved."
---

# Delegate with a Review Gate

## Purpose

Use this skill when implementation is done by someone else and one reviewer holds the gate to the repository:

- coding agents or IDE assistants working on a task;
- contributors preparing a branch for a maintainer;
- several workers on independent tasks in parallel.

Its purpose is to gain speed from delegation without losing:

- the project's conventions;
- the evidence behind each change;
- control over what reaches the repository.

---

## Core Principle

> Delegate execution, never verification. Only changes the reviewer has checked, and the maintainer has approved, are committed.

The workflow is:

```text
brief → isolated branch → implementation → handoff (diff + short note)
      → review (review-a-pull-request) → approval → commit and push by the gate holder
```

---

## Relationship to Other Skills

- The brief follows `one-semantic-change`, `fix-one-bug` or the skill matching the task type.
- The review step is `review-a-pull-request`.
- The final checks are `validate-a-package-change`.

---

## What to Delegate

Delegate tasks that have:

- one semantic purpose;
- a known set of files;
- an acceptance check that can be run (tests, examples, package check);
- no open design decision.

Examples:

- implementing a function whose signature and behavior are already decided;
- adding tests for documented behavior;
- removing functions whose deprecation period has ended;
- updating documentation to match merged behavior.

Do not delegate:

- root-cause investigation of an unexplained failure;
- design decisions (names, arguments, defaults, scope);
- API changes whose consequences span packages;
- replies to issues, reviews and pull requests;
- anything whose acceptance check would itself need judgement.

These depend on context the delegate does not have, and reviewing them costs more than doing them.

---

## The Task Brief

Every delegated task gets a self-contained brief. The delegate should not need any other conversation to do the task.

```text
Goal:
<one sentence: what should be true after the change>

Context:
<issue or discussion to read; existing code to reuse>

In scope:
<files or functions to change>

Out of scope:
<everything else, explicitly: formatting, unrelated refactors, dependencies>

Conventions:
<project rules the delegate must follow: naming, argument names, style,
 documentation, which skills apply; how generated files are produced (do not
 hand-edit files a generator owns); the version bump the project expects>

Simplicity:
<no repeated logic, no guards that cannot fail, base idioms over hand-written
 loops; the reviewer will send back known duplication>

Test data:
<the real example datasets the package family offers; let the delegate
 pick the ones that show the behavior best and say why; list known
 pitfalls of each dataset; constructed objects only for cases real data
 cannot express>

When done:
<the one command the delegate runs (usually the tests); a stop rule: if the
 same error survives two attempts, stop and report it>

Acceptance checks:
<for the reviewer: the full package check, deliberate breakages, tests that
 must fail on the old code>

Handoff:
<the diff, plus at most three lines: what changed, what was run, open questions>

Not allowed:
<commit, push, branch or history operations; new dependencies unless listed;
 changes to files outside the scope>
```

Keep the delegate's part of the brief light. Long lists of checks for the delegate to enforce lead to endless loops, chasing one error, or editing source files in place to test deliberate breakages. The full package check, deliberate breakages and judgement calls belong to the reviewer.

Keep briefs where the project keeps working notes. If the project does not track design notes, keep briefs untracked.

---

## Isolation

- One branch or worktree per task.
- No two delegates on overlapping files at the same time.
- Each branch starts from the current base.
- The delegate never merges, rebases or pushes.

---

## Reviewing the Handoff

Apply `review-a-pull-request` to the diff.

In particular:

- read the diff, not the delegate's working log;
- treat every statement in the handoff note as an unverified claim;
- run the acceptance checks yourself, including the full package check, not
  only the tests;
- check that new or changed tests fail on the old code;
- run a separate simplicity pass (`review-a-pull-request`, Step 7);
- revert regenerated files the task did not require;
- remove leftovers the delegate created (check folders, tarballs, empty
  directories, stray repositories), in the worktree and next to it;
- check names and arguments against the existing code, not against the brief alone.

Only what the reviewer observed goes into commit messages, pull request descriptions or replies.

---

## Iterating

When the handoff fails review, return:

- the failing check and its output;
- the specific lines involved;
- what the result should be.

If a second round still fails, re-scope the task or do it yourself. Repeated rounds on the same task usually mean the brief was missing a decision.

---

## Committing

- The gate holder commits and pushes, after the maintainer has approved the diff.
- Commit identity, messages and attribution follow project policy.
- One task becomes one commit or one pull request, matching `one-semantic-change`.

---

## Context Hygiene

Delegation saves the reviewer's attention only if handoffs stay small:

- ask for the diff and a short note, not transcripts;
- do not paste long tool output into the review record; summarize what was checked;
- keep one task per handoff.

---

## Review Checklist

### Before delegating

- [ ] The task has one semantic purpose and no open design decision.
- [ ] The brief is self-contained.
- [ ] The delegate runs one command and has a stop rule; acceptance checks are runnable commands for the reviewer.
- [ ] The branch is isolated and current.

### After the handoff

- [ ] The diff stays inside the scope.
- [ ] Acceptance checks were re-run by the reviewer.
- [ ] Tests fail on the old code.
- [ ] Handoff claims were verified or discarded.
- [ ] The maintainer approved before commit.

---

## Completion Criteria

This skill is satisfied when:

- only scoped, decidable tasks were delegated;
- each task had a self-contained brief;
- each handoff was reviewed with `review-a-pull-request`;
- every committed line was verified by the reviewer;
- nothing reached the repository without the maintainer's approval.

---

## Final Rule

> Others may write the change; the reviewer owns the evidence, and the maintainer owns the decision to commit.
