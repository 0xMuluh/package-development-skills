---
name: fix-one-bug
description: "Use when correcting a specific defect in existing software: describe the observable bug independently of the fix, add a regression test reproducing the failure, implement the narrowest sufficient fix, and preserve unrelated behavior."
---

# Fix One Bug

## Purpose

Use this skill when correcting a specific defect in existing software.

Its purpose is to produce bug-fix pull requests that are:

- narrow in semantic scope;
- easy to review;
- protected by regression evidence;
- conservative with unrelated behavior;
- explicit about what is and is not being fixed.

This skill applies to:

- incorrect outputs;
- runtime errors;
- crashes;
- invalid input handling;
- naming or alignment defects;
- compatibility regressions;
- incorrect defaults;
- broken edge cases;
- accidental behavior changes;
- integration failures;
- numerical defects;
- state or metadata corruption.

The goal is not to improve the surrounding codebase.

The goal is to correct one bug completely and avoid unrelated changes.

---

## Core Principle

> Reproduce the bug, define the correct behavior, make the smallest sufficient correction, and add evidence that prevents the bug from returning.

A good bug-fix PR should usually be explainable as:

```text
Given this input,
the old implementation does X incorrectly,
the corrected implementation does Y,
and unrelated behavior remains unchanged.
```

---

## Relationship to Other Skills

This skill assumes the principles from:

- `core-development-philosophy`;
- `one-semantic-change`;
- `reuse-before-implementing`;
- `characterize-before-changing`.

In particular:

- the bug fix should have one semantic purpose;
- existing solutions should be reused where suitable;
- nearby behavior should be characterized only as needed;
- unrelated cleanup belongs elsewhere.

---

## Required Bug Statement

Before changing code, write the bug as:

```text
When <specific condition/input>,
<actual behavior> occurs,
but <expected behavior> should occur.
```

Example:

```text
When alternative experiments are selected by valid numeric indices,
the function errors because it calls a misspelled accessor,
but it should return the requested experiments.
```

This statement should identify:

1. the triggering condition;
2. the observed incorrect behavior;
3. the expected behavior.

Avoid vague bug statements such as:

```text
The selector is broken.
```

or:

```text
The utility code is fragile.
```

---

## Bug-Fix Scope Contract

Before implementation, define:

```text
Bug:
<single reproducible defect>

Expected behavior:
<what should happen instead>

Regression evidence:
<test or reproducible case that fails before the fix and passes after>

Behavior to preserve:
- <nearby behavior>
- <public contract>

Non-goals:
- <related cleanup>
- <refactor>
- <additional feature>
```

---

## Procedure

### Step 1 — Reproduce the Bug

Create the smallest practical reproduction.

Prefer:

- a minimal constructed object;
- a small deterministic input;
- a short function call;
- an isolated failing case.

Avoid using a large dataset or full workflow unless the bug cannot be reproduced more narrowly.

The reproduction should answer:

```text
What exact input causes the defect?
```

---

### Step 2 — Confirm the Bug Is Real

Before editing code, verify that the observed behavior is actually incorrect.

Check:

- documentation;
- tests;
- public API contract;
- established ecosystem behavior;
- issue description;
- relevant historical behavior if necessary.

Do not "fix" behavior that is surprising but intentional.

If the expected behavior is ambiguous, resolve that ambiguity before redesigning implementation.

---

### Step 3 — Identify the Fault Boundary

Trace only enough code to determine where the defect occurs.

A useful model is:

```text
input
  ↓
validation
  ↓
selection/preparation
  ↓
core operation
  ↓
result construction
```

Determine which stage causes the bug.

Avoid broad codebase inspection unless needed.

The purpose is to find the defect, not to discover all possible improvements.

---

### Step 4 — Establish Nearby Behavior to Preserve

Identify behavior that the fix could accidentally affect.

Examples:

- character selection still works;
- valid indices retain requested order;
- invalid indices still error;
- return type is unchanged;
- names are preserved;
- default arguments are unchanged;
- numerical output remains unchanged outside the failing case.

Protect only what is relevant to the risk of the fix.

Do not build an exhaustive test suite unless the code is being prepared for broader change.

---

### Step 5 — Search for Existing Correct Behavior

Before implementing a custom correction, check whether the bug arises from not using an existing public API or established behavior.

Ask:

```text
Is there already a correct function, accessor, class, helper, or upstream method
that should be used here?
```

Prefer the established solution when suitable.

Do not introduce a new abstraction merely to fix the bug.

---

### Step 6 — Write the Regression Test

Prefer writing or defining the regression evidence before changing implementation.

The regression test should:

- reproduce the bug;
- express the correct expected behavior;
- fail on the buggy version;
- pass after the fix;
- be as small as practical.

A regression test protects the behavior, not the exact implementation.

---

## Regression Test Pattern

A good regression test usually has three parts:

```text
Arrange:
construct the smallest failing input

Act:
call the affected public or relevant internal interface

Assert:
verify the corrected behavior
```

For example:

```r
test_that("numeric alternative-experiment selection works", {
    x <- make_minimal_example()

    result <- select_experiments(x, 2)

    expect_identical(names(result), "experiment_2")
})
```

The actual API will differ by package.

The principle remains the same.

---

### Step 7 — Make the Narrowest Sufficient Correction

Change only what is necessary to correct the bug.

Prefer:

```text
wrong accessor
    ↓
correct accessor
```

over:

```text
rewrite entire selection subsystem
```

Prefer:

```text
incorrect conditional
    ↓
correct conditional
```

over:

```text
replace validation framework
```

Do not optimize for the fewest lines.

Optimize for the fewest independent behavioral changes.

---

### Step 8 — Verify the Regression Test

Confirm that:

```text
before fix:
test fails for the expected reason

after fix:
test passes
```

When practical, explicitly verify the pre-fix failure during development.

This prevents accidentally writing a test that never exercised the bug.

---

### Step 9 — Run Relevant Existing Tests

Run:

- the new regression test;
- nearby tests for the affected behavior;
- the package test suite when practical;
- package checks required by project policy.

The purpose is to verify both:

```text
bug fixed
```

and:

```text
collateral behavior preserved
```

---

### Step 10 — Inspect the Diff

Review every changed line.

Ask:

```text
Would this line need to change if this bug did not exist?
```

If no, remove it from the PR.

This is the strongest scope-control test for bug fixes.

---

## Correct Level for the Regression Test

Test at the highest stable boundary that reproduces the defect clearly.

Prefer:

```text
public API
```

when the bug is user-visible.

Use an internal helper test when:

- the defect is genuinely internal;
- the public setup would make the test unnecessarily complex;
- the helper has a stable enough contract worth testing.

Do not expose a private helper solely to make testing easier.

---

## Test the Bug, Not the Patch

Avoid tests that encode the implementation correction directly.

Poor:

```r
expect_true(grepl("altExps", deparse(body(.select_altexps))))
```

Better:

```r
expect_no_error(selecting_valid_numeric_altexp())
```

Best when possible:

```r
expect_identical(
    selected_experiment_names(...),
    expected_names
)
```

The test should remain valid if the implementation is later refactored.

---

## Error Bugs

When fixing incorrect error behavior, distinguish among:

- missing validation;
- wrong error condition;
- wrong error class;
- misleading message;
- error where success is expected.

If the bug is:

```text
valid input errors
```

the regression test should primarily assert successful behavior.

If the bug is:

```text
invalid input is silently accepted
```

the test should assert the correct rejection.

Avoid freezing complete error wording unless wording is part of the public contract.

---

## Numerical Bugs

For numerical defects:

1. construct the smallest case exposing the problem;
2. identify a trustworthy expected result;
3. use suitable numerical tolerance;
4. verify dimensions, names, or orientation if relevant;
5. avoid replacing one numerical implementation with another unless necessary.

Expected values may come from:

- a known analytic result;
- an established upstream implementation;
- a trusted reference method;
- an invariant.

Document the source of the expected result when it is not obvious.

---

## Alignment Bugs

For bugs involving samples, rows, features, or experiments, test explicitly:

- names;
- order;
- dimensions;
- matching behavior;
- dropped or retained observations.

A numerically correct result with incorrect alignment is still a bug.

Prefer tiny fixtures where expected alignment is visually obvious.

---

## Metadata Bugs

When fixing metadata placement or preservation:

- assert the relevant metadata field;
- assert that required original metadata remains;
- avoid snapshotting unrelated metadata.

Example boundary:

```text
new result is stored in the intended metadata location
original sample annotations remain unchanged
```

---

## Compatibility Bugs

For compatibility failures caused by dependency or platform changes:

1. reproduce the affected path;
2. determine the public replacement or compatible implementation;
3. preserve package behavior where possible;
4. avoid unrelated dependency modernization;
5. test the compatibility boundary.

The semantic purpose should remain:

```text
restore compatibility with X
```

not:

```text
modernize dependency handling
```

---

## Bugs in Private API Usage

If the defect arises because the package calls a dependency's private API:

1. characterize the required behavior;
2. search for a public replacement;
3. use the public API if suitable;
4. otherwise implement the minimal local behavior;
5. preserve existing package semantics.

Do not replace one private dependency call with another private call.

---

## Bug Fix Versus Refactor

A bug may reveal poor structure.

That does not automatically make refactoring part of the fix.

Prefer:

```text
PR 1:
fix bug

PR 2:
refactor now-tested area
```

Combine refactoring with the bug fix only when:

- the current structure prevents a safe narrow fix;
- the refactor is strictly necessary to correct the defect;
- the refactor itself preserves unrelated behavior.

Explain that necessity explicitly.

---

## Bug Fix Versus Generalization

A bug in one supported case does not automatically justify generalized support.

Example:

```text
Bug:
numeric experiment selection fails.

Not implied:
support logical matrices, formulas, selectors, predicates, or arbitrary query objects.
```

Fix the supported case first.

Add broader capabilities separately.

---

## Bug Fix Versus Error-Message Cleanup

If a bug fix changes an error path, do not rewrite all nearby messages for consistency unless that is necessary for correctness.

Prefer:

```text
fix incorrect condition
```

over:

```text
fix condition + rewrite error style across subsystem
```

Message consistency can be another PR.

---

## Bug Fix Versus Formatting

Do not reformat the entire affected file.

If automatic tooling changes unrelated lines:

- restrict the formatting scope;
- revert unrelated formatting;
- or separate formatting into another PR.

A clean diff improves bug-fix review substantially.

---

## Bug Fix Versus Dependency Change

Do not add or remove an unrelated dependency because you noticed it while debugging.

If the bug itself is caused by dependency usage, then dependency modification may be part of the same semantic change.

Otherwise, separate it.

---

## Bug Fix Versus Documentation

Documentation changes belong in the bug-fix PR when:

- the documentation currently describes the wrong behavior;
- the corrected behavior needs explanation;
- an example was broken by the bug.

Independent documentation improvements do not.

---

## Minimal Bug-Fix PR Template

```text
## Bug

When <condition>,
<incorrect behavior> occurs.

## Expected behavior

<correct behavior>

## Cause

<brief technical cause, if known>

## Fix

<brief description of the narrow correction>

## Regression evidence

<test that fails before and passes after>

## Preserved behavior

- <nearby behavior>
- <public contract>

## Non-goals

- <unrelated cleanup>
- <refactor/generalization>
```

---

## Examples

### Example 1 — Misspelled Accessor

```text
Bug:
Valid numeric alternative-experiment selection errors.

Cause:
The numeric branch calls `alExps()` instead of the established `altExps()` accessor.

Fix:
Use `altExps()`.

Regression evidence:
A minimal object with two alternative experiments can select the second one numerically.

Preserved:
- character selection;
- return type;
- ordering;
- invalid-index behavior.

Non-goals:
- no selector redesign;
- no utility cleanup.
```

The implementation may be one line.

The test may be much longer.

That is a good bug-fix PR.

---

### Example 2 — Incorrect Sample Alignment

```text
Bug:
When assays contain the same samples in different orders,
the model pairs rows by position rather than sample name.

Expected behavior:
Samples are aligned by the established identifier before fitting.

Fix:
Use the existing container/alignment mechanism before the upstream call.

Regression evidence:
Two tiny assays with deliberately reversed sample order produce the same result as manually aligned inputs.

Non-goals:
- no new alignment framework;
- no support for partially overlapping samples unless already specified.
```

---

### Example 3 — Private API Breakage

```text
Bug:
A dependency update removes a private helper used by the package.

Expected behavior:
The public package function continues to produce the same output.

Fix:
Replace the private helper with an exported equivalent or minimal local operation.

Regression evidence:
Known input produces equivalent output and preserves names.

Non-goals:
- no broad dependency rewrite;
- no unrelated transformation refactor.
```

---

### Example 4 — Wrong Default

```text
Bug:
The documented default is X, but the implementation currently uses Y.

Expected behavior:
The implementation uses X.

Regression evidence:
Calling the function without the argument behaves identically to explicitly passing X.

Non-goals:
- no redesign of other defaults;
- no argument renaming.
```

---

## Handling Multiple Bugs Found Together

If debugging reveals additional defects, classify them.

### Same root cause, same behavioral correction

They may belong together if one correction necessarily fixes the same bug across equivalent paths.

Example:

```text
the same incorrect accessor is used in three methods
```

A single PR may correct all three if the semantic purpose is:

```text
Use the correct accessor for numeric experiment selection.
```

### Independent defects

Split them.

Example:

```text
numeric selection fails
and
error message for duplicate names is misleading
```

These are separate bugs.

---

## Handling a Bug With Unclear Expected Behavior

If the actual behavior is clearly wrong but the correct behavior is uncertain:

1. inspect public documentation and examples;
2. inspect ecosystem conventions;
3. inspect tests and history;
4. identify the smallest contract consistent with existing use;
5. avoid redesigning beyond that contract.

If multiple plausible behaviors remain, the issue should first be clarified at the design level.

Do not hide a design decision inside a bug fix.

---

## Regression Test Quality Checklist

A good regression test is:

- [ ] directly connected to the reported defect;
- [ ] minimal enough to understand;
- [ ] deterministic;
- [ ] behavior-focused;
- [ ] independent of private implementation where possible;
- [ ] failing before the fix;
- [ ] passing after the fix;
- [ ] unlikely to fail because of irrelevant formatting or metadata.

---

## Review Checklist

### Scope

- [ ] Exactly one defect is being corrected.
- [ ] The bug statement is reproducible.
- [ ] The expected behavior is explicit.
- [ ] Non-goals are clear.

### Implementation

- [ ] The correction is the narrowest sufficient one.
- [ ] Existing public APIs are reused where suitable.
- [ ] No speculative abstraction was introduced.
- [ ] No unrelated refactor or cleanup is present.

### Evidence

- [ ] A regression test demonstrates the defect.
- [ ] The regression test protects behavior, not implementation.
- [ ] Relevant nearby behavior is preserved.
- [ ] Existing tests still pass.

### Diff

- [ ] Every changed line can be justified by the bug.
- [ ] Unrelated formatting is absent.
- [ ] Unrelated documentation changes are absent.
- [ ] Unrelated dependency changes are absent.

---

## Completion Criteria

This skill is satisfied when:

- the bug can be stated precisely;
- the defect is reproducible;
- the correct behavior is established;
- the relevant preservation boundary is understood;
- a regression test demonstrates the expected behavior;
- the implementation change is narrowly sufficient;
- unrelated behavior remains unchanged;
- unrelated cleanup is excluded;
- the final diff has one semantic purpose;
- the bug cannot reappear without causing the regression test to fail.

---

## Final Rule

> Fix the defect, preserve the neighborhood, and leave the rest of the codebase alone.

A bug report is not a license to redesign adjacent software.

Correct one thing completely.
