---
name: review-a-pull-request
description: "Use when reviewing a pull request or a diff written by someone else (a contributor, a co-maintainer or another agent): state what the change should do, verify its claims by running them, check that its tests can actually fail, separate blocking problems from optional suggestions, and choose the review state from verified evidence."
---

# Review a Pull Request

## Purpose

Use this skill when reviewing a change written by someone else:

- a pull request from a contributor or co-maintainer;
- a diff handed over by another agent or tool;
- a branch prepared for you to merge or commit.

Its purpose is to produce reviews that are:

- based on verified behavior, not on the description;
- focused on what the change must and must not do;
- concrete enough that the author can act on every point;
- honest about what is blocking and what is optional.

The goal is not to rewrite the change in your own style.

The goal is to decide, with evidence, whether the change does what it says and nothing else.

---

## Core Principle

> Review the behavior, not the description. Every statement in a review is either something you ran or something you can point to in the diff.

A good review can usually be summarized as:

```text
The change is meant to do X.
I checked X by running Y; it holds (or it does not, because Z).
Nothing outside X changed, except: ...
```

---

## Relationship to Other Skills

This skill applies the same standards a change must meet before it is opened:

- `one-semantic-change` for scope;
- `fix-one-bug` and `test-at-the-behavioral-boundary` for tests;
- `evolve-public-apis` and `deprecate-and-replace` for API changes;
- `validate-a-package-change` for checks.

It is also the review step of `delegate-with-a-review-gate`.

---

## Procedure

### Step 1 — State What the Change Should Do

Read the pull request description and the issue or discussion it refers to.

Write one sentence:

```text
After this change, <behavior> should hold, and <nearby behavior> should be unchanged.
```

If you cannot write that sentence, ask the author before reviewing details.

---

### Step 2 — Check Scope and Base

Check that:

- the change has one semantic purpose;
- the branch is current with its base, or conflicts are limited to version and changelog lines;
- generated files changed only where the change requires it.

Unrelated changes are a review point on their own, even if they are correct.

---

### Step 3 — Read the Diff Against the Sentence

Classify every changed hunk:

```text
intended          → implements the stated behavior
necessary         → direct consequence (documentation, generated files, NEWS)
unrelated         → not needed for the stated behavior
```

Unrelated hunks should be removed or moved to another pull request.

---

### Step 4 — Verify Claims by Running Them

Treat statements in the description as claims:

- "tests pass";
- "fixes the error on platform X";
- "no behavior change";
- "faster";
- "matches existing convention".

Verify each claim that matters for the decision:

- run the relevant tests locally or read the CI results for this exact commit;
- reproduce the original problem and confirm it is gone;
- compare outputs before and after where behavior should be unchanged;
- check conventions against the existing code, not against memory.

If a claim cannot be verified, say so in the review instead of repeating it.

---

### Step 5 — Check That the Tests Can Fail

A test only protects behavior if it fails when the behavior is broken.

For a bug fix or a regression test:

1. revert the fix locally (or use the base branch code);
2. run the new or changed test;
3. it must fail, ideally with the original error.

Repeat this check whenever a pull request rewrites, combines, simplifies or "hardens" an existing regression test. Changes that make a test more robust can also remove the condition that made it fail, for example:

- qualifying a function with its namespace where the bug was about name lookup;
- mocking or stubbing the component that caused the failure;
- changing the environment, backend or data so the failing path is no longer reached.

A test that passes on the broken code is not a regression test, however well it reads.

---

### Step 6 — Check Public Behavior and Documentation

Check, where relevant:

- renamed or removed exported functions are deprecated if they were released;
- defaults and argument names are unchanged unless the change is about them;
- documentation, examples and NEWS match the new behavior;
- version and changelog follow project policy.

---

### Step 7 — Write the Review

Write each point so the author can act on it:

- what you observed;
- how you observed it (the command, test or input);
- what you suggest.

Prefer inline suggested changes when the fix is a few lines, so the author can apply them directly.

Keep one point per comment.

Mark each point as blocking or optional.

Thank the author for the parts that work. Do not pad the review with praise or with restatements of the pull request.

---

### Step 8 — Choose the Review State

```text
Approve          → the stated behavior is verified, tests can fail, scope is clean
Comment          → optional suggestions, or questions that do not block merging
Request changes  → a correctness, scope or regression-protection problem
```

Do not approve on unverified claims.

Do not request changes for style preferences that the project does not enforce.

---

## Review Checklist

### Understanding

- [ ] The intended behavior is stated in one sentence.
- [ ] The linked issue or discussion has been read.

### Scope

- [ ] One semantic purpose.
- [ ] Branch current with base.
- [ ] No unrelated hunks or regenerated files.

### Evidence

- [ ] Claims that matter were verified by running them.
- [ ] Regression tests fail on the old code.
- [ ] Rewritten tests still fail on the old code.
- [ ] Preserved behavior was compared where relevant.

### Public behavior

- [ ] Released names are deprecated, not removed.
- [ ] Documentation, examples and NEWS match.

### Review text

- [ ] Every point states what was observed and how.
- [ ] Blocking and optional points are separated.
- [ ] Review state matches the evidence.

---

## Completion Criteria

This skill is satisfied when:

- the intended behavior is explicit;
- every decision-relevant claim was verified or reported as unverified;
- tests were shown to fail without the change;
- scope problems are listed;
- each review point is actionable;
- the review state follows from the evidence.

---

## Final Rule

> Approve what you verified, question what you could not, and never let a well-written description stand in for a test that can fail.
