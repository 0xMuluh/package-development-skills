---
name: one-semantic-change
description: Use when planning, implementing, reviewing, or revising a pull request to enforce one semantic purpose per PR: write scope contracts with problem, intended change, non-goals, and behavioral boundary; justify every changed line; split entangled or independent changes; review for scope creep and formatting noise.
---

# One Semantic Change

## Purpose

Use this skill when planning, implementing, reviewing, or revising a pull request.

Its purpose is to ensure that one PR represents one coherent intended change.

This skill applies to:

- bug fixes;
- new features;
- refactoring;
- API changes;
- dependency changes;
- documentation changes;
- migrations;
- deprecations;
- compatibility work;
- test additions;
- maintenance changes.

The goal is not to minimize the number of changed lines.

The goal is to minimize the number of independent reasons for changing the codebase.

---

## Core Rule

> One PR should have one semantic purpose.

A semantic purpose is a single intended change in behavior, structure, contract, or supported workflow.

A PR may touch many files if all of those edits are necessary to accomplish that one purpose.

For example, a function rename may require changes to:

- implementation;
- exports;
- documentation;
- tests;
- examples;
- vignettes;
- NEWS.

That is still one semantic change if every edit exists only because of the rename.

By contrast, a PR that:

- fixes a bug;
- renames a nearby helper;
- removes an unused dependency;
- reformats the file;

contains several semantic changes even if the diff is small.

---

## When to Use This Skill

Use this skill before starting any non-trivial code change.

Use it again when:

- the implementation expands beyond the original plan;
- an adjacent issue is discovered;
- a reviewer asks for additional cleanup;
- a test exposes unrelated behavior;
- a dependency change becomes tempting;
- a migration reveals design problems;
- a refactor begins changing outputs;
- the PR becomes difficult to summarize.

This skill should also be used during review to decide whether a PR is appropriately scoped.

---

## Required Inputs

Before implementation, identify:

### Problem

What single problem is being solved?

State the problem independently of the proposed solution.

Good:

```text
Numeric alternative-experiment selection fails with an error.
```

Poor:

```text
We should clean up the alternative-experiment utilities.
```

The first describes an observable problem.

The second describes an open-ended activity.

---

### Intended Change

What should be different after the PR is merged?

Good:

```text
Numeric alternative-experiment selection succeeds for valid indices.
```

Good:

```text
The public function is named `getJointRDA()` instead of `getRDA()`.
```

Good:

```text
The package no longer calls a non-exported function from dependency X.
```

Poor:

```text
Improve the RDA code.
```

Poor:

```text
Make utilities more robust.
```

The intended change should be specific enough that a reviewer can determine whether it happened.

---

### Non-Goals

List nearby improvements that are explicitly excluded.

Examples:

```text
Non-goals:
- redesigning alternative-experiment selection;
- changing error messages;
- renaming helper functions;
- adding new selection modes;
- reformatting the utility file.
```

Non-goals are not filler.

They create a boundary that protects the PR from accidental expansion.

---

### Behavioral Boundary

State what existing behavior must remain unchanged.

Examples:

```text
Behavior that must remain unchanged:
- character-based experiment selection;
- existing return type;
- default argument behavior;
- sample ordering;
- error behavior for invalid indices.
```

This is especially important for:

- bug fixes;
- refactors;
- dependency replacements;
- migrations;
- optimization;
- internal rewrites.

---

### Evidence

Identify the evidence that will prove the intended change.

Prefer:

- a focused unit test;
- a regression test;
- a package check;
- a reproducible example;
- a before/after result comparison;
- an API-level compatibility assertion.

Do not rely only on visual inspection of the implementation.

---

## Semantic Scope Test

Before coding, complete this sentence:

> This PR exists to __________.

The answer should contain one main action.

Good:

```text
This PR exists to fix numeric `altExp` selection.
```

Good:

```text
This PR exists to rename `getRDA()` to `getJointRDA()`.
```

Good:

```text
This PR exists to add characterization tests for `getMantel()`.
```

Poor:

```text
This PR exists to fix selection and clean up the utilities.
```

Poor:

```text
This PR exists to migrate the method, improve the API, and simplify the implementation.
```

If the sentence requires multiple independent conjunctions, the work probably needs to be split.

---

## The Diff Justification Rule

Every changed line should answer:

> Why must this line change for the stated purpose of the PR?

Acceptable answers include:

- implementation required by the intended change;
- test required to prove the intended change;
- documentation required because the public contract changed;
- export required because a public name changed;
- compatibility code required because behavior must be preserved;
- example update required because the old example would become invalid.

Unacceptable answers include:

- "it was nearby";
- "it looked old";
- "this was cleaner";
- "while I was here";
- "the formatter changed it";
- "this helper could also be improved";
- "we might need this later";
- "this is more modern";
- "the old name bothered me".

If a changed line cannot be justified by the PR's semantic purpose, revert it or move it to another PR.

---

## Procedure

### Step 1 — Write the Scope Contract

Before implementation, write:

```text
Problem:
<one concrete problem>

Intended change:
<one resulting change>

Non-goals:
- <excluded adjacent change>
- <excluded adjacent change>

Behavior that must remain unchanged:
- <existing behavior>
- <existing behavior>

Evidence:
<test or check that will prove completion>
```

This contract may be brief.

It should be specific.

---

### Step 2 — Identify the Minimum Change Surface

Determine which files or components genuinely need modification.

Do not minimize files artificially.

Instead ask:

```text
Which parts of the repository must change for this one behavior to change correctly?
```

For a public rename, the correct surface may include:

```text
R/
NAMESPACE
man/
tests/
vignettes/
NEWS
```

For a one-character bug fix, the correct surface may include:

```text
R/
tests/
```

The number of files is not the measure of scope.

The number of independent intentions is.

---

### Step 3 — Establish the Behavioral Boundary

Before editing, identify behaviors that could accidentally change.

For example, when modifying a selector:

```text
target behavior:
numeric selection

possible collateral behavior:
character selection
NULL handling
ordering
duplicate handling
error messages
return type
```

Protect important collateral behavior with existing tests or focused additional tests when justified.

Do not broaden the implementation merely because those neighboring cases exist.

---

### Step 4 — Implement Only the Intended Change

Make the narrowest sufficient implementation.

Do not automatically:

- generalize the solution;
- create a helper;
- rename related objects;
- redesign the API;
- reorganize code;
- remove duplication;
- optimize;
- modernize syntax.

Those actions may be justified, but they require their own reason.

If they are not necessary for the stated change, exclude them.

---

### Step 5 — Add Evidence

Add the smallest test or check that demonstrates the intended change at the appropriate behavioral boundary.

For bug fixes, prefer a regression test reproducing the failure.

For refactors, prefer tests demonstrating unchanged public behavior.

For API changes, test the new public contract.

For migrations, test parity with the source behavior.

For dependency changes, test behavior previously provided through the old dependency path.

---

### Step 6 — Inspect the Diff for Scope Creep

Review the final diff line by line.

Classify every change as one of:

```text
A. necessary implementation
B. necessary test
C. necessary documentation
D. necessary compatibility/supporting edit
E. unrelated
```

Remove category E.

If category E contains worthwhile work, record it separately.

Do not retain it in the current PR.

---

### Step 7 — Re-run the Semantic Scope Test

At completion, finish this sentence again:

> This PR exists to __________.

If the answer changed materially from the original scope contract, determine whether:

1. the scope legitimately evolved but remains one semantic change; or
2. independent changes became entangled.

If independent changes are present, split them before review.

---

## When to Split Work

Split work when two changes can be:

- implemented independently;
- reviewed independently;
- tested independently;
- reverted independently;
- released independently.

A useful rule is:

> If one part could be merged safely without the other, they are probably separate changes.

---

## Common Split Patterns

### Bug fix + refactor

Avoid:

```text
Fix selector bug and rewrite selector utilities.
```

Prefer:

```text
PR 1:
Fix selector bug and add regression test.

PR 2:
Refactor selector utilities without changing behavior.
```

---

### Migration + redesign

Avoid:

```text
Move method to new package and redesign its API.
```

Prefer:

```text
PR 1:
Port method with behavior parity.

PR 2:
Deprecate or delegate old location.

PR 3:
Redesign API.
```

---

### Feature + dependency cleanup

Avoid:

```text
Add method and remove an unrelated dependency.
```

Prefer:

```text
PR 1:
Add method using current dependency structure.

PR 2:
Remove unused dependency.
```

---

### Rename + behavioral change

Avoid:

```text
Rename function and change defaults.
```

Prefer:

```text
PR 1:
Rename function without changing behavior.

PR 2:
Change default behavior.
```

---

### Documentation + behavior change

Documentation updates required by a behavior change belong in that PR.

Independent documentation improvements do not.

Acceptable:

```text
Rename `getRDA()` to `getJointRDA()`
→ update examples that call the renamed function.
```

Separate:

```text
Rewrite the entire RDA tutorial for clarity.
```

---

## Long Diffs Can Still Be Simple

Do not reject a change merely because it is large.

Examples of semantically simple but mechanically broad changes include:

### Public Rename

```text
R/getRDA.R
R/AllGenerics.R
NAMESPACE
man/getRDA.Rd
tests/testthat/test-getRDA.R
vignettes/example.Rmd
NEWS.md
```

All edits support one semantic purpose:

```text
Rename `getRDA()` to `getJointRDA()`.
```

---

### Compatibility Fix

A compatibility update may require changes across several methods because the external API changed.

If every edit has the same purpose:

```text
Support version X of dependency Y without changing package behavior.
```

the PR can still be semantically small.

---

### Regression Fix

The implementation fix may be one line while the regression fixture and tests require dozens of lines.

That is still a simple PR.

Tests are part of the solution.

---

## Short Diffs Can Still Be Too Broad

Example:

```diff
- old_function(x)
+ new_function(x)

- helper_name <- ...
+ clearer_helper_name <- ...

- Imports: dplyr
```

Three lines may represent:

1. a behavior change;
2. a rename;
3. a dependency change.

That is not one semantic change.

---

## Scope Creep Triggers

Stop and reassess if you think:

- "while I am here";
- "this is a good opportunity to";
- "I may as well";
- "this helper is ugly";
- "the surrounding code should also";
- "we could generalize this";
- "this dependency can probably go";
- "we should rename this too";
- "I noticed another failing case";
- "the formatter changed the whole file".

These are signals that another task may exist.

They are not evidence that it belongs in the current PR.

---

## Handling Newly Discovered Bugs

If work on one issue reveals another bug:

1. determine whether the second bug prevents completion of the current change;
2. if not, do not fix it in the same PR;
3. record it separately;
4. add a separate test or issue later.

If the second bug must be fixed for the original change to work, explain the dependency explicitly in the PR.

The criterion is necessity, not proximity.

---

## Handling Reviewer Requests

Not every reviewer suggestion belongs in the current PR.

Classify requested changes as:

### Required for correctness

Include them.

### Required for the stated contract

Include them.

### Required for maintainability of the introduced change

Include them when directly relevant.

### Independent improvement

Prefer a follow-up PR.

A review should improve the current change without turning it into a container for unrelated maintenance.

---

## Refactoring Rule

Refactoring is itself a semantic purpose when the goal is structural improvement without intended behavioral change.

A refactor PR should state:

```text
Intended behavioral change:
None.
```

Its evidence should demonstrate behavior preservation.

Do not combine refactoring with new behavior unless the refactor is strictly required to implement that behavior safely.

---

## Formatting Rule

Avoid unrelated formatting changes in functional PRs.

If formatting tools modify unrelated areas:

- restrict formatting to touched regions where possible;
- revert unrelated formatting;
- or make formatting a separate PR.

Mechanical formatting can obscure review and hide unintended behavioral changes.

---

## Generated Files

Generated documentation or metadata may legitimately change as a consequence of one semantic change.

Examples:

- `.Rd` files generated by documentation tooling;
- namespace changes generated from annotations;
- lockfile changes required by a dependency update.

These are not separate semantic changes when they are direct consequences of the intended change.

Avoid regenerating unrelated artifacts unnecessarily.

---

## Commit Structure

Commits inside a PR may separate mechanical steps, but the PR as a whole should still have one semantic purpose.

Useful commit decomposition:

```text
1. add regression test
2. implement fix
3. update required documentation
```

Avoid using commits to justify unrelated work inside one PR.

The unit of review remains the complete PR.

---

## PR Description Template

Use this template where practical:

```text
## Problem

<What concrete problem exists?>

## Intended change

<What changes after this PR?>

## Reuse

<What existing implementation, class, API, or dependency is reused?>

## Non-goals

- <What is deliberately not changed?>
- <What adjacent work is deferred?>

## Behavior preserved

- <Important behavior that should remain unchanged>
- <Important behavior that should remain unchanged>

## Evidence

<Tests, examples, or checks demonstrating the change>
```

---

## Review Checklist

Before requesting review:

### Purpose

- [ ] The PR has one semantic purpose.
- [ ] The purpose can be stated in one sentence.
- [ ] The problem and intended change are distinct and clear.

### Scope

- [ ] Every changed line supports the intended change.
- [ ] No unrelated cleanup remains.
- [ ] No opportunistic rename remains.
- [ ] No unrelated formatting remains.
- [ ] No unrelated dependency change remains.

### Behavior

- [ ] The intended behavior is demonstrated.
- [ ] Important unrelated behavior is preserved.
- [ ] Public contracts are unchanged unless changing them is the purpose.

### Reviewability

- [ ] A reviewer can understand why each file changed.
- [ ] The diff does not require understanding unrelated subsystems.
- [ ] Independent work has been split into follow-up tasks.

---

## Examples

### Example 1 — Good Bug Fix

```text
Problem:
Numeric alternative-experiment selection errors because the implementation calls
a misspelled accessor.

Intended change:
Valid numeric selection succeeds.

Non-goals:
- redesign selection;
- change character selection;
- change errors for invalid indices;
- refactor utilities.

Behavior preserved:
- existing character selection;
- return type;
- ordering.

Evidence:
A regression test using numeric selection.
```

This is semantically small even if the test setup is long.

---

### Example 2 — Bad Bug Fix

```text
Fix numeric selection, simplify all selection helpers, rename several utilities,
and improve error messages.
```

This contains at least four independent changes.

Split it.

---

### Example 3 — Good Rename

```text
Problem:
The current public function name collides conceptually with another established API.

Intended change:
Rename `getRDA()` to `getJointRDA()`.

Non-goals:
- no argument changes;
- no result changes;
- no algorithm changes;
- no plotting changes.

Evidence:
Tests verifying the renamed function preserves existing behavior.
```

The PR may touch many files and still be simple.

---

### Example 4 — Good Migration

```text
Problem:
A method belongs in a different package.

Intended change:
Expose the same method from the destination package with behavior parity.

Non-goals:
- no API redesign;
- no algorithm optimization;
- no new result class;
- no visualization work.

Evidence:
Parity tests against the existing implementation.
```

---

### Example 5 — Bad Migration

```text
Move the method, rename it, simplify the algorithm, introduce a new result class,
and add plotting support.
```

Each of those changes should be evaluated independently.

---

## Completion Criteria

This skill is satisfied when:

- one concrete problem is stated;
- one intended change is stated;
- non-goals define the boundary;
- important preserved behavior is identified;
- existing components are reused where appropriate;
- evidence demonstrates the intended change;
- unrelated edits have been removed;
- the final PR can be accurately summarized in one sentence.

---

## Final Rule

> A PR should be as large as necessary to make one change complete, and no larger.

Do not optimize for fewer lines.

Optimize for fewer independent intentions.
