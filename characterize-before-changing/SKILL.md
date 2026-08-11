---
name: characterize-before-changing
description: "Use before changing existing behavior that is insufficiently understood or insufficiently tested: establish what the code currently does, add focused characterization tests, then make the intended change; do not redesign poorly understood code and tests simultaneously."
---

# Characterize Before Changing

## Purpose

Use this skill before changing existing behavior that is insufficiently understood or insufficiently tested.

Its purpose is to reduce maintenance risk by establishing what the software currently does before modifying it.

This skill applies to:

- bug fixes;
- refactoring;
- migrations;
- dependency replacement;
- API changes;
- optimization;
- compatibility updates;
- deprecations;
- rewrites;
- maintenance of mature code;
- inherited or unfamiliar codebases.

The goal is not to preserve every accidental implementation detail.

The goal is to understand and protect the observable behavior that should remain stable while a specific change is made.

---

## Core Principle

> Before changing existing behavior, establish the behavioral boundary you intend to preserve.

When code is poorly tested or poorly understood, do not redesign it and discover its behavior at the same time unless the task explicitly requires that.

Prefer:

```text
observe
    ↓
characterize
    ↓
change
    ↓
verify
```

over:

```text
rewrite
    ↓
discover regressions
    ↓
guess what was intended
```

---

## What Characterization Means

Characterization is the process of documenting existing observable behavior through focused evidence.

That evidence may include:

- unit tests;
- integration tests;
- reproducible examples;
- snapshots used carefully;
- result comparisons;
- class assertions;
- dimension and naming checks;
- error-condition checks;
- metadata checks;
- compatibility checks;
- numerical tolerance checks.

A characterization test answers:

> What does this code do today at the boundary that matters?

It does not necessarily answer:

> What is the ideal design?

Those are separate questions.

---

## When Characterization Is Required

Use this skill when one or more of the following are true:

- the code has little or no test coverage;
- the code is unfamiliar;
- the implementation is complex;
- behavior depends on several helpers;
- the code is being moved between packages;
- a dependency is being replaced;
- a public API is being renamed or restructured;
- the implementation will be substantially rewritten;
- numerical behavior must remain stable;
- object classes or metadata may be affected;
- current behavior is relied upon by users;
- bug reports suggest edge cases are poorly understood.

Characterization is especially important when the intended change is:

```text
behavioral change:
none
```

as in refactoring or migration.

---

## When Full Characterization Is Not Necessary

Do not build a large test suite before every trivial change.

A narrow bug fix may require only:

- one regression test;
- one or two nearby preservation checks.

A documentation typo may require no behavioral characterization.

A new isolated function with no existing behavior should be tested against its new contract rather than characterized.

Use the smallest amount of characterization needed to make the intended change safe.

---

## Characterization Versus Specification

These are different.

### Characterization

Describes observed current behavior.

```text
The function currently returns a matrix with samples in rows.
```

### Specification

Describes intended supported behavior.

```text
The function should return a matrix with samples in rows.
```

Do not automatically convert every observed behavior into a permanent specification.

Some current behaviors may be:

- bugs;
- accidental;
- undocumented;
- unstable;
- implementation-specific;
- undesirable.

Characterization provides evidence.

Engineering judgment determines what should be preserved.

---

## Required Questions

Before changing existing code, answer:

### 1. What is the intended change?

State what should differ after the change.

If behavior should remain unchanged, say:

```text
Intended behavioral change:
None.
```

---

### 2. What behavior is exposed to users or callers?

Identify relevant observable behavior such as:

- accepted input types;
- argument meaning;
- defaults;
- return type;
- result dimensions;
- names;
- ordering;
- metadata;
- side effects;
- error conditions;
- warnings;
- numerical values;
- delegated behavior;
- compatibility with downstream methods.

---

### 3. Which behavior must remain unchanged?

Not everything needs protection.

Choose behavior relevant to the current change.

---

### 4. Which behavior is uncertain?

List areas where the implementation or contract is unclear.

These deserve investigation before editing.

---

### 5. What evidence will establish the current boundary?

Choose the smallest suitable tests or examples.

---

## Characterization Procedure

### Step 1 — Read the Public Contract First

Before inspecting implementation details, review:

- function documentation;
- exported signatures;
- examples;
- vignettes;
- NEWS entries;
- package-level documentation;
- issue discussions when relevant;
- downstream use if known.

This provides context for what users may reasonably rely on.

Do not treat implementation as the only source of truth.

---

### Step 2 — Inspect Existing Tests

Determine:

- what behavior is already protected;
- what important behavior is not protected;
- whether tests assert public behavior or private structure;
- whether historical regression tests reveal design intent.

Reuse existing tests as the starting boundary.

Do not duplicate coverage unnecessarily.

---

### Step 3 — Trace the Relevant Execution Path

Inspect only the code necessary to understand the target behavior.

Identify:

```text
public entry point
    ↓
relevant validation
    ↓
relevant helper(s)
    ↓
upstream/dependency call
    ↓
result construction
```

Avoid broad repository cleanup or unrelated archaeology.

The purpose is to understand the change surface, not fully redesign the subsystem.

---

### Step 4 — Build the Smallest Representative Input

Prefer a minimal object that exercises the behavior clearly.

Good fixtures are:

- small enough to reason about;
- deterministic;
- explicit;
- fast;
- free from irrelevant setup.

For scientific software, use the smallest dataset that preserves the relevant semantics.

Avoid large example datasets when a tiny constructed object is sufficient.

---

### Step 5 — Observe Current Behavior

Record outputs that matter.

Depending on the function, inspect:

- `class()`;
- `dim()`;
- `names()`;
- row names;
- column names;
- ordering;
- metadata;
- attributes;
- numerical values;
- error type or message when important;
- warnings;
- side effects;
- downstream compatibility.

Do not assert every available property.

Assert only behavior relevant to the maintenance boundary.

---

### Step 6 — Decide What Is Intentional Enough to Preserve

Classify observed behavior into:

```text
A. documented/public and should be preserved
B. important compatibility behavior and should be preserved
C. uncertain but relevant; investigate further
D. incidental implementation detail; do not freeze
E. known bug; do not preserve as desired behavior
```

This classification prevents characterization tests from accidentally canonizing bugs or internals.

---

### Step 7 — Add Focused Characterization Tests

Write tests for categories A and B, and possibly C after clarification.

Avoid tests for category D.

For category E, prefer a regression test expressing the desired corrected behavior as part of the bug-fix PR.

---

### Step 8 — Make the Intended Change

Only after the relevant boundary is understood should implementation change.

Follow the scope defined by `one-semantic-change`.

Do not use characterization as permission for broad refactoring.

---

### Step 9 — Compare Before and After

Confirm:

```text
target behavior:
changed as intended

preserved behavior:
unchanged

unrelated behavior:
not intentionally modified
```

For numerical code, compare with appropriate tolerances.

For migrations, compare source and destination behavior directly where practical.

---

## What to Characterize

The correct boundary depends on the type of software.

### Function Interface

Consider:

- formal arguments;
- defaults;
- accepted types;
- missing-value behavior;
- `...` forwarding;
- return type.

---

### Dimensions and Orientation

For matrix-like scientific results, explicitly check:

- rows represent what?
- columns represent what?
- are dimensions preserved?
- are transpositions occurring?
- are names preserved?

Orientation errors are common and often difficult to notice from value-only tests.

---

### Ordering and Alignment

For multi-object data, characterize:

- sample order;
- feature order;
- matching rules;
- dropped observations;
- intersection versus union behavior;
- duplicate handling where relevant.

Do not assume container alignment without verifying the relevant contract.

---

### Metadata

When results or containers carry metadata, inspect only meaningful fields.

Examples:

- assay names;
- sample annotations;
- feature annotations;
- reduced-dimension names;
- result attributes;
- method parameters;
- provenance fields.

Do not freeze volatile metadata without need.

---

### Classes

Assert result classes when downstream compatibility depends on them.

For example:

```r
expect_s3_class(result, "rda")
```

may be valuable if existing plotting and summary methods rely on that class.

Do not create or assert a package-specific wrapper unless required.

---

### Numerical Results

For deterministic numerical methods:

- use small known inputs;
- assert key values;
- use explicit tolerances;
- avoid over-precise floating-point equality.

For stochastic methods:

- set a seed if the upstream method supports deterministic testing;
- or test stable structural properties rather than exact values.

---

### Errors and Warnings

Protect error behavior when it is part of the public contract or relevant to the intended change.

Prefer asserting:

- that an error occurs;
- the relevant error class if available;
- a stable identifying message fragment when necessary.

Avoid freezing entire error messages unless wording is itself important.

---

### Side Effects

If a function mutates or augments an object, characterize:

- where data are stored;
- what names are used;
- whether original data are retained;
- what unrelated parts remain unchanged.

---

## Characterization for Different Change Types

## Bug Fix

A bug-fix characterization usually consists of:

```text
1. reproduce the failing case;
2. establish nearby valid behavior;
3. fix the bug;
4. prove the regression case now succeeds;
5. verify nearby valid behavior remains unchanged.
```

Do not characterize the entire subsystem unless necessary.

---

## Refactor

For a refactor:

```text
Intended behavioral change:
None.
```

Before refactoring, characterize the public boundary.

After refactoring, all relevant characterization tests should continue to pass unchanged.

Avoid changing tests merely to accommodate a new internal structure.

---

## Migration

When moving functionality:

```text
source implementation
        ↓
characterization
        ↓
destination implementation
        ↓
parity check
```

Where practical, run identical inputs through both implementations and compare:

- classes;
- dimensions;
- names;
- key values;
- metadata;
- errors;
- important argument behavior.

Migration parity should be established before redesign.

---

## Dependency Replacement

When replacing a dependency or private API:

1. characterize behavior provided by the old path;
2. implement the new path;
3. compare outputs;
4. document intentional differences separately.

Do not assume two mathematically similar implementations are behaviorally identical.

Check:

- defaults;
- missing-value behavior;
- names;
- orientation;
- tolerances;
- edge cases.

---

## API Rename

For a pure rename, characterize the existing function before changing names.

The new function should preserve:

- arguments;
- defaults;
- return type;
- results;
- errors;
- side effects.

The semantic change should be the name, not the behavior.

---

## Optimization

Before optimizing, capture:

- correctness;
- output structure;
- relevant numerical behavior.

Performance improvement should not silently alter semantics.

Benchmarking may demonstrate the optimization, but correctness tests remain necessary.

---

## What Not to Freeze

Avoid characterization tests that bind the package to accidental internals.

Do not ordinarily assert:

- private helper call order;
- exact internal temporary object structure;
- local variable names;
- implementation-specific warning sequence;
- invisible attributes with no supported role;
- exact formatting of internal debug output;
- private function names;
- source-file organization.

A test suite should allow implementation to evolve while protecting user-relevant behavior.

---

## Snapshot Testing

Snapshots can be useful but should be used selectively.

Use snapshots when:

- output is naturally textual and user-visible;
- the complete representation is meaningful;
- manual review of changes is valuable.

Avoid snapshots when:

- output contains unstable metadata;
- large numerical objects make differences opaque;
- only a few structural assertions are relevant;
- updates would normalize accidental behavioral changes.

Prefer explicit assertions when they communicate the contract more clearly.

---

## Golden Files and Reference Outputs

Reference outputs may be appropriate for:

- parsers;
- serializers;
- complex reports;
- deterministic scientific transformations.

Keep them:

- small;
- reviewable;
- versioned;
- justified.

Do not use opaque binary reference files when a small explicit test is sufficient.

---

## Upstream Behavior

If the package delegates to an external implementation, distinguish:

```text
behavior owned by this package
```

from:

```text
behavior inherited from upstream
```

The package should test its integration contract, not duplicate the upstream package's entire test suite.

For example, test that:

- data are prepared correctly;
- arguments are forwarded;
- result classes are preserved;
- names and orientation are correct.

Do not retest the statistical theory of a mature upstream package unless the package modifies it.

---

## Minimal Characterization Strategy

For each public method, a small initial characterization suite may cover:

```text
1. smallest valid input
2. expected result class
3. expected dimensions/orientation
4. names or alignment
5. one important argument
6. one meaningful invalid input
```

This is often enough to make future maintenance substantially safer.

Expand only when real requirements or regressions justify it.

---

## Test Naming

Name characterization tests by behavior, not implementation.

Prefer:

```text
"getJointRDA preserves sample names"
"numeric altExp selection preserves requested order"
"getMantel returns a vegan mantel result"
```

Avoid:

```text
".prepare_data calls .align_samples"
".internal_helper returns list element x"
```

Behavioral names make maintenance intent clear.

---

## Characterization Notes

When behavior is surprising or historically important, include a brief test comment explaining why it is protected.

Example:

```r
# Preserve feature order because downstream loadings are matched by row name.
```

Avoid comments that merely restate the code.

---

## Handling Ambiguous Existing Behavior

If implementation, documentation, and tests disagree:

1. identify the discrepancy explicitly;
2. determine which behavior users are likely relying on;
3. consult release history or issues when necessary;
4. choose the intended contract deliberately;
5. treat any correction as its own semantic change.

Do not silently choose whichever behavior is easiest to preserve.

---

## Handling Known Bugs

Do not write a characterization test that says a known incorrect result is permanently expected.

Instead distinguish:

```text
current observed bug:
<what happens>

desired behavior:
<what should happen>
```

Then create a regression test for the desired behavior as part of the bug fix.

If surrounding behavior must remain stable, characterize that separately.

---

## PR Pattern: Characterization Only

A characterization PR is valuable when preparing a poorly tested mature subsystem for later change.

Its semantic purpose can be:

```text
Add behavioral coverage for `getMantel()` without changing implementation.
```

Such a PR should:

- add tests;
- avoid implementation changes;
- document surprising behavior only when necessary;
- avoid opportunistic cleanup.

This makes later changes easier to review.

---

## PR Pattern: Characterize and Fix

For a narrow bug, characterization and fix can belong in the same PR when the tests are directly required to demonstrate the bug and its correction.

Example:

```text
Problem:
numeric selection fails

Evidence:
new regression test

Implementation:
one targeted correction
```

A separate characterization PR is not required for every bug.

---

## Review Checklist

### Understanding

- [ ] The relevant public contract was inspected.
- [ ] Existing tests were reviewed.
- [ ] The relevant execution path was understood.
- [ ] Uncertain behavior was identified.

### Boundary

- [ ] The behavior that must remain unchanged is explicit.
- [ ] Known bugs are not accidentally frozen.
- [ ] Private implementation details are not treated as public contract.

### Evidence

- [ ] Tests use small representative inputs.
- [ ] Assertions focus on observable behavior.
- [ ] Numerical comparisons use appropriate tolerances.
- [ ] Classes, dimensions, names, and alignment are tested when relevant.

### Change Safety

- [ ] The intended change is separate from characterization.
- [ ] Unrelated behavior remains protected.
- [ ] Tests do not require the new internal implementation.

---

## Examples

### Example 1 — Untested Public Method

Suppose a public function has no meaningful tests.

Before refactoring, add:

```text
- valid minimal input;
- expected result class;
- expected dimensions;
- preservation of sample names;
- one invalid-input case.
```

Then refactor in a later PR.

---

### Example 2 — Bug in Experiment Selection

Observed behavior:

```text
numeric selection errors
character selection succeeds
```

Desired behavior:

```text
numeric selection succeeds
character selection remains unchanged
```

Tests should cover:

```text
- regression case for numeric selection;
- preservation of character selection if collateral risk exists.
```

Do not rewrite all selection tests unless needed.

---

### Example 3 — Method Migration

Before moving `getJointMethod()` to another package:

```text
input fixture
    ↓
source result
    ↓
destination result
```

Compare:

```text
class
dimensions
names
key numerical values
important metadata
```

Do not redesign the result during the migration PR.

---

### Example 4 — Replacing a Private Dependency Call

Before replacing:

```r
dependency:::private_transform(x)
```

characterize the behavior your package actually relies on:

```text
input handling
output values
names
missing-value handling
important attributes
```

Then implement the public or local replacement against that boundary.

---

## Completion Criteria

This skill is satisfied when:

- the intended change is explicit;
- the relevant current behavior is understood;
- the preservation boundary is explicit;
- the smallest useful characterization evidence exists;
- known bugs are not accidentally made permanent;
- private internals are not unnecessarily frozen;
- the implementation change can be evaluated against a known behavioral baseline;
- preserved behavior remains unchanged after the change.

---

## Final Rule

> Do not change what you have not yet decided whether to preserve.

Understand first.

Characterize what matters.

Then change only what you intend.
