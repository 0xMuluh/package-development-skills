---
name: test-at-the-behavioral-boundary
description: "Use when adding or revising tests for package behavior: test externally meaningful behavior such as accepted inputs, outputs, dimensions, names, classes, alignment, metadata, and error conditions; avoid freezing private helper structure or incidental implementation details."
---

# Test at the Behavioral Boundary

## Purpose

Use this skill when adding or revising tests for package behavior.

Its purpose is to ensure that tests protect meaningful user-facing contracts without unnecessarily freezing private implementation details.

This skill applies to:

- new features;
- bug fixes;
- refactors;
- migrations;
- API changes;
- dependency replacements;
- numerical methods;
- adapters;
- data alignment;
- serialization;
- metadata handling;
- compatibility work.

The governing principle is:

> Test what the software promises, not how the current implementation happens to achieve it.

A strong test suite should make safe refactoring easier, not harder.

---

## Core Principle

Prefer testing at the highest stable boundary that clearly expresses the behavior of interest.

Typical preference order:

```text
public function or method
    ↓
stable package-level abstraction
    ↓
internal helper only when necessary
```

Tests should focus on:

- inputs;
- outputs;
- classes;
- dimensions;
- names;
- ordering;
- metadata;
- side effects;
- errors;
- warnings;
- numerical behavior;
- argument forwarding;
- interoperability.

Avoid assertions about internal call structure unless that structure is itself a contract.

---

## Why Boundary Choice Matters

Tests are part of package architecture.

A test that asserts private implementation details can accidentally make those details permanent.

For example, this test:

```r
expect_true(grepl(".align_samples", deparse(body(getJointMethod))))
```

protects an implementation choice.

This test:

```r
expect_identical(
    getJointMethod(reordered_input),
    getJointMethod(aligned_input)
)
```

protects the behavior users care about.

The second test allows future refactoring.

---

## Required Testing Statement

Before writing tests, identify:

```text
Behavior under test:
<what the package promises>

Stable boundary:
<public function / generic / object contract>

Representative input:
<small input that exercises the behavior>

Expected observation:
<class / value / names / error / side effect>

Implementation details deliberately not tested:
<private helpers / call order / temporary objects>
```

---

## Procedure

### Step 1 — State the Behavior

Write the behavior as a sentence.

Good:

```text
Numeric experiment selection preserves requested order.
```

Good:

```text
The adapter aligns samples by identifier before fitting.
```

Good:

```text
The function returns the established upstream result class.
```

Poor:

```text
`.prepare_input()` calls `.match_samples()`.
```

The first group describes contracts.

The last describes implementation.

---

### Step 2 — Choose the Highest Stable Boundary

Ask:

```text
Where would a user or downstream caller observe this behavior?
```

Test there if practical.

Use internal tests only when:

- public setup would be disproportionately complex;
- the helper has a meaningful stable contract;
- failure localization materially improves maintenance.

Do not expose private helpers solely for testing.

---

### Step 3 — Construct the Smallest Revealing Input

Use small explicit fixtures.

Good test data should make mistakes obvious.

Examples:

- different sample orders;
- non-square matrices for orientation tests;
- duplicated names for validation;
- two or three features with known coefficients;
- a tiny deterministic tree;
- a small object with two alternative experiments.

Avoid large datasets when a tiny constructed case is sufficient.

---

### Step 4 — Assert Only Relevant Properties

Test the contract needed for the behavior.

Examples:

```text
class
dimensions
names
order
one or more key values
specific metadata field
error class
warning condition
```

Do not assert every field simply because it is available.

Over-assertion creates brittle tests.

---

### Step 5 — Separate Structural and Numerical Assertions

For scientific software, distinguish:

```text
structure:
class, dimensions, names, orientation

numerics:
statistics, coefficients, distances, scores
```

Both may matter.

Do not rely solely on numerical equality when row/column identity could be wrong.

---

### Step 6 — Use Appropriate Numerical Tolerance

Avoid exact floating-point equality unless mathematically guaranteed.

Use tolerances appropriate to:

- algorithm;
- scale;
- upstream behavior;
- platform variation.

Do not use excessively loose tolerances that would allow real regressions.

---

### Step 7 — Test Known Failure Modes

Include errors or warnings when they represent meaningful contract boundaries.

Examples:

- missing experiment;
- invalid assay name;
- no common samples;
- unsupported input type;
- duplicate identifier when uniqueness is required.

Avoid exhaustive validation testing unless validation is the feature under test.

---

### Step 8 — Keep Tests Deterministic

For stochastic methods:

- set a seed where appropriate;
- use deterministic options where available;
- or assert stable structural/invariant properties instead of exact values.

Do not write flaky tests that rely on chance.

---

### Step 9 — Run Tests Before and After the Change

For bug fixes or refactors, when practical confirm:

```text
before:
new test fails or existing characterization passes

after:
intended test passes
preservation tests still pass
```

This validates that the test actually covers the change.

---

## Test the Contract, Not the Helper Graph

Avoid tests such as:

```r
expect_called(.helper_a)
expect_called(.helper_b)
```

unless delegation itself is the intended contract.

Prefer testing the observable result of that delegation.

Call-graph tests make harmless refactors expensive.

---

## Test Public Result Classes

If downstream interoperability depends on an established class, test it.

Example:

```r
expect_s3_class(result, "rda")
```

This protects compatibility with upstream summary/plot methods.

It also discourages unnecessary future wrapper classes.

---

## Test Dimensions and Orientation

For scientific matrices and model inputs, test:

- row meaning;
- column meaning;
- output dimensions;
- transposition behavior.

Use asymmetric data.

Bad fixture:

```text
3 × 3
```

Better fixture:

```text
2 features × 4 samples
```

A mistaken transpose becomes detectable.

---

## Test Names and Ordering

Names are often part of the data contract.

Test:

- sample names;
- feature names;
- assay names;
- result names;
- requested ordering.

A result can be numerically correct but semantically wrong if names are misaligned.

---

## Test Alignment Explicitly

For multi-input methods, construct inputs with identical identifiers in different orders.

Then compare:

```text
adapter(reordered inputs)
```

to:

```text
adapter(manually aligned inputs)
```

This directly tests the integration rule.

---

## Test Missing-Observation Rules

If the method defines behavior for partially overlapping samples, test the exact rule:

```text
intersection
rejection
padding
explicit missingness
```

Do not leave alignment semantics implicit.

---

## Test Argument Forwarding

When a thin adapter forwards `...` or upstream options, test at least one argument whose effect is observable.

Prefer:

```text
adapter(..., upstream_option = value)
→ result reflects value
```

over inspecting internal calls.

---

## Test Side Effects

If a function modifies or augments a container, assert:

- where the new data appear;
- the expected name;
- preservation of original data;
- preservation of unrelated metadata when relevant.

Do not snapshot the whole object if a few explicit checks describe the contract better.

---

## Test Errors at Stable Granularity

Prefer:

```r
expect_error(..., "no common samples")
```

or an error class when available.

Avoid requiring complete exact wording unless wording is intentionally part of the API.

Small message fragments are often more robust.

---

## Regression Tests

A regression test should reproduce a concrete historical defect.

It should:

- fail before the correction;
- pass after the correction;
- stay focused on the user-visible bug;
- avoid asserting the implementation patch itself.

Name regression tests by behavior.

Good:

```text
"numeric altExp selection accepts valid indices"
```

Poor:

```text
"uses altExps instead of alExps"
```

---

## Characterization Tests

Characterization tests protect existing behavior before maintenance.

Use them for:

- poorly tested public methods;
- migrations;
- refactors;
- dependency replacement.

Do not characterize every incidental attribute.

Protect only behavior you intend to preserve.

---

## Adapter Tests

For a thin adapter, test the integration boundary rather than the upstream algorithm.

Typical adapter tests:

```text
1. selects correct data
2. aligns observations
3. converts orientation
4. forwards one important argument
5. returns upstream result class
6. rejects one meaningful invalid input
```

Do not duplicate the upstream package's statistical test suite.

---

## Migration Tests

For migration, compare source and destination behavior.

Possible checks:

```text
class
dimensions
names
key values
important metadata
errors
```

Ignore incidental differences such as namespace-qualified calls or environment objects unless user-visible.

---

## Dependency-Replacement Tests

When changing implementation provider:

1. characterize old package behavior;
2. run new implementation;
3. compare relevant outputs;
4. document intentional differences.

Do not assume mathematically equivalent packages share defaults or edge behavior.

---

## Refactor Tests

A refactor should ideally leave behavior tests unchanged.

If tests must change because they inspect private implementation:

- move them toward the behavioral boundary;
- do not simply rewrite them around the new internals.

The goal is to make future refactoring easier.

---

## API Tests

For public API changes, test the changed contract directly.

Examples:

### Rename

```text
new function behaves as old function did
deprecated old function delegates during transition
```

### Default Change

```text
implicit call uses new default
explicit old value still behaves as specified
```

### Argument Rename

```text
new argument works
old argument warns/delegates during transition
conflicting use is handled clearly
```

---

## Snapshot Tests

Use snapshots selectively.

Good uses:

- stable textual summaries;
- user-facing print methods;
- error reports where full structure matters.

Avoid snapshots for:

- large numerical objects;
- volatile metadata;
- platform-sensitive output;
- objects where only a few fields matter.

Snapshots should not replace thinking about the contract.

---

## Golden/Reference Outputs

Reference files may be justified for:

- parsers;
- serializers;
- complex deterministic transformations;
- external format compatibility.

Keep them:

- small;
- reviewable;
- human-understandable where possible.

Avoid opaque binary fixtures unless necessary.

---

## Fixture Design

Prefer fixtures that are:

- minimal;
- explicit;
- deterministic;
- reusable only when semantics are genuinely shared.

Do not build a large test-fixture framework before repeated needs justify it.

A few clear local fixture constructors are often better.

---

## Helper Functions in Tests

Test helpers are useful when they remove irrelevant setup.

Good helper:

```text
make_two_assay_object()
```

if many tests need the same meaningful structure.

Weak helper:

```text
assert_everything_about_result()
```

that hides what each test actually protects.

Keep assertions visible enough that test intent is readable.

---

## One Behavior Per Test

Prefer tests with one coherent behavioral purpose.

A test may contain several assertions if they all describe one contract.

Example:

```text
"joint method preserves aligned sample identity"
```

may assert:

- dimensions;
- sample names;
- order.

Avoid giant tests that verify an entire workflow and make failures ambiguous.

---

## Test Naming

Use names that describe behavior.

Prefer:

```text
"getJointRDA returns an rda object"
"joint methods align samples by name"
"invalid assay names are rejected"
```

Avoid:

```text
"test helper 1"
"works"
"internal branch"
```

The test name should help reviewers understand the contract.

---

## Test Coverage Is Not the Goal

High line coverage does not guarantee good behavioral coverage.

Do not add tests solely to execute every branch.

Prioritize:

- important contracts;
- historically fragile behavior;
- integration boundaries;
- supported edge cases;
- user-visible failures.

Coverage metrics may help identify gaps but should not define test architecture.

---

## Avoid Over-Mocking

Mocks can be useful for expensive or external interactions.

For package-internal scientific methods, excessive mocking may test wiring instead of behavior.

Prefer real small inputs and real upstream calls when:

- they are fast;
- deterministic;
- stable.

Mock only what materially improves isolation or reliability.

---

## Upstream Package Testing Boundary

Do not retest mature dependencies in full.

If your package calls:

```r
vegan::rda()
```

test that your adapter:

- supplies correct inputs;
- forwards relevant options;
- returns the expected class.

Do not reproduce vegan's full ordination validation suite.

---

## Test Performance

Unit tests should generally remain fast enough for frequent execution.

Prefer:

- tiny datasets;
- few iterations;
- deterministic paths.

Long-running integration or benchmark tests may belong in separate test categories if the project supports them.

Do not weaken correctness merely for speed.

---

## Platform Robustness

Avoid tests that depend unnecessarily on:

- file-path separators;
- locale-specific formatting;
- floating-point exactness across BLAS implementations;
- ordering of unordered structures;
- wall-clock timing;
- network availability.

Test semantic outcomes.

---

## Test Data Provenance

For non-trivial scientific expected values, document how the reference was derived when not obvious.

Possible sources:

- analytic calculation;
- published reference;
- established upstream method;
- manually verified invariant.

This helps future maintainers decide whether a changed result is a bug or legitimate update.

---

## Testing Private Helpers

Private-helper tests are appropriate when the helper has:

- a clear semantic contract;
- multiple callers;
- edge behavior expensive to reproduce through public APIs.

Even then, prefer behavior-based assertions.

Do not test trivial wrappers simply because they exist.

---

## Deleting Private-Helper Tests

If a refactor removes a private helper, its tests should not block deletion if equivalent public behavior remains covered.

This is evidence that boundary-oriented tests are working correctly.

---

## Test PR Scope

A test-only PR can be a valid semantic change.

Examples:

```text
Add characterization tests for `getMantel()`.
```

```text
Add regression coverage for sample alignment.
```

Do not use test PRs for unrelated production cleanup.

---

## Minimal Test Set for a Public Analytical Method

A useful initial baseline is often:

```text
1. smallest valid input
2. expected result class
3. expected dimensions/orientation
4. names/alignment
5. one important argument
6. one meaningful invalid input
```

Add more only when real use cases or regressions justify them.

---

## PR Template

```text
## Behavior protected

<one clear contract>

## Boundary

<public function/generic/object>

## Fixture

<small representative input>

## Assertions

- <class/value/dimension/name>
- <error/warning/metadata if relevant>

## Implementation details intentionally not frozen

- <private helper>
- <internal call structure>
```

---

## Examples

### Example 1 — Sample Alignment

Behavior:

```text
Two assays with the same samples in different orders are aligned by sample identifier.
```

Test:

```text
construct assay A in order 1,2,3
construct assay B in order 3,1,2
run adapter
compare to manually aligned input
```

This tests semantics directly.

---

### Example 2 — Upstream Result Preservation

Behavior:

```text
Joint RDA returns the established upstream RDA object.
```

Test:

```r
expect_s3_class(result, "rda")
```

Do not test the internal wrapper call graph.

---

### Example 3 — Orientation

Behavior:

```text
features stored as rows are supplied to upstream method in expected orientation.
```

Use a non-square matrix and a result property that would differ under accidental transpose.

---

### Example 4 — Regression Bug

Bug:

```text
valid numeric experiment index throws an error
```

Regression test:

```text
minimal object
valid numeric index
expected selected experiment
```

Do not inspect function source text.

---

### Example 5 — Bad Test

```r
expect_identical(
    names(formals(.prepare_joint_inputs)),
    c("x", "assay", "margin")
)
```

unless that private signature is intentionally a stable internal contract.

This test would make harmless refactoring harder.

---

## Review Checklist

### Boundary

- [ ] The test targets a meaningful stable interface.
- [ ] Private structure is avoided where possible.
- [ ] The test name describes behavior.

### Fixture

- [ ] Input is small and revealing.
- [ ] Orientation/alignment mistakes would be detectable.
- [ ] Randomness is controlled.

### Assertions

- [ ] Only relevant properties are asserted.
- [ ] Names and dimensions are checked where semantically important.
- [ ] Numerical tolerances are appropriate.
- [ ] Errors are checked at stable granularity.

### Architecture

- [ ] Upstream algorithms are not unnecessarily retested.
- [ ] Tests permit implementation refactoring.
- [ ] No unnecessary test framework has been introduced.

### Scope

- [ ] Each test has one coherent behavioral purpose.
- [ ] Test additions support the intended PR.
- [ ] Unrelated production changes are absent.

---

## Completion Criteria

This skill is satisfied when:

- behavior under test is explicit;
- the highest practical stable boundary is used;
- fixtures are minimal and deterministic;
- assertions focus on meaningful contracts;
- names, dimensions, orientation, and alignment are tested when relevant;
- numerical comparisons use appropriate tolerances;
- regression tests fail before and pass after fixes;
- upstream implementations are tested only at the integration boundary;
- private implementation details remain free to change;
- tests make future maintenance safer rather than more rigid.

---

## Final Rule

> Protect behavior strongly and implementation lightly.

A good test should tell future maintainers what must remain true without telling them how the code must be written.
