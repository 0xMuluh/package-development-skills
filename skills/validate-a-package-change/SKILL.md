---
name: validate-a-package-change
description: "Use as the final verification gate for a package change before review or merge: run focused evidence first, then broader tests, documentation/build checks, package checks, and ecosystem-specific checks; distinguish failures introduced by the change from pre-existing failures and never fix unrelated failures in the same PR."
---

# Validate a Package Change

## Purpose

Use this skill after implementing a package change and before requesting review,
merging, releasing, or declaring the work complete.

Its purpose is to verify that:

- the intended change works;
- the evidence promised by the PR exists;
- unrelated behavior remains stable;
- generated artifacts are synchronized where required;
- package-level checks still pass;
- failures are correctly attributed;
- validation does not become a new source of scope creep.

This skill applies to:

- bug fixes;
- new features;
- refactors;
- migrations;
- dependency changes;
- API changes;
- deprecations;
- documentation changes;
- packaging changes;
- compatibility updates.

The governing principle is:

> Validate from the narrowest evidence outward, and fix only failures caused by the intended change.

---

## Core Contract

Validation is not another development phase in which arbitrary cleanup is allowed.

A change is ready when:

```text
intended behavior
    is demonstrated

relevant neighboring behavior
    still works

package-level checks
    do not reveal regressions caused by the change

remaining unrelated failures
    are reported, not silently absorbed into the PR
```

---

## Relationship to Other Skills

This skill should be used after the implementation workflow defined by the
relevant task skill, especially:

- `one-semantic-change`;
- `fix-one-bug`;
- `characterize-before-changing`;
- `refactor-without-behavioral-change`;
- `migrate-functionality-without-redesign`;
- `test-at-the-behavioral-boundary`.

The original scope contract remains authoritative during validation.

---

## Required Validation Statement

Before running broad checks, restate:

```text
Intended change:
<one semantic purpose>

Primary evidence:
<test/example/check proving the change>

Behavior expected to remain unchanged:
- <relevant behavior>
- <relevant behavior>

Package validation required:
- <test suite>
- <documentation/build>
- <package check>
- <ecosystem-specific check>

Known pre-existing failures:
<none known, or list>

Non-goals:
- unrelated cleanup
- unrelated warning fixes
- unrelated dependency updates
```

---

# Validation Order

Use a widening sequence.

```text
1. direct reproduction / focused test
2. affected test file or subsystem
3. complete unit test suite
4. generated documentation or artifacts
5. package build/check
6. ecosystem-specific validation
7. diff and scope audit
```

Do not begin with the slowest check when a focused test can detect the primary
failure immediately.

---

## Step 1 — Re-run the Primary Evidence

The exact behavior that justified the change should pass first.

Examples:

### Bug fix

```text
regression test that failed before
    → now passes
```

### Refactor

```text
characterization tests
    → remain unchanged
```

### Migration

```text
source/destination parity comparison
    → passes
```

### API change

```text
new contract
    → works

compatibility path
    → behaves as planned
```

If the primary evidence fails, stop broad validation and correct the intended
change.

---

## Step 2 — Run the Nearest Relevant Tests

Run the smallest meaningful surrounding test group.

Examples:

- the affected test file;
- tests for the same public method;
- tests for the same object class;
- tests for alignment if alignment changed;
- tests for serialization if representation changed.

This catches collateral regressions while failure context is still narrow.

---

## Step 3 — Run the Complete Unit Test Suite

Once focused evidence passes, run the package's normal test suite.

Do not skip this because the targeted test passed.

Package interactions may reveal:

- changed defaults;
- namespace conflicts;
- hidden coupling;
- altered return classes;
- stale fixtures;
- accidental side effects.

---

# Failure Attribution

Every failure discovered during validation must be classified.

Use:

```text
A. introduced by this change
B. exposed by this change
C. pre-existing and unrelated
D. environmental or tooling failure
E. unclear
```

---

## A — Introduced by This Change

Fix it in the current PR if it falls inside the intended semantic boundary.

Example:

```text
renaming an exported function leaves its documentation stale
```

That is part of completing the rename.

---

## B — Exposed by This Change

Ask whether the newly exposed problem is required for the intended change to be
correct.

If yes:

```text
fix narrowly
explain why it is necessary
```

If no:

```text
record separately
do not absorb it into the PR
```

---

## C — Pre-existing and Unrelated

Do not fix it in the current PR.

Record:

```text
check/test
failure
evidence it predates the change
```

A PR should not grow merely because validation found unrelated debt.

---

## D — Environmental or Tooling Failure

Examples:

- unavailable external service;
- transient network problem;
- missing system dependency;
- broken CI runner;
- package repository outage.

Confirm that the failure is not caused by the package change.

Report it explicitly.

Do not alter package behavior to appease an unrelated environment failure.

---

## E — Unclear Failure

Investigate only enough to attribute it.

Use comparison with:

- base branch;
- previous commit;
- clean environment;
- known-good package version.

Do not immediately "fix" an unattributed failure.

---

# Validate Against the Current Base

Validation is only meaningful against the code the change will be merged into.

Before validating:

- fetch the base branch;
- start new work from the current base branch;
- merge the current base into an open PR that is behind it;
- use current development versions of direct dependencies.

When several PRs are open at once, merging the base into each of them often conflicts on version and changelog lines.

After resolving such conflicts:

- confirm that the diff against the base contains only the PR's own changes;
- re-run the tests for files changed by both the PR and the new base commits.

---

# Baseline Comparison

When a broad check fails unexpectedly, compare the same check against the base
revision when practical.

Conceptually:

```text
base branch:
check X → fails

change branch:
check X → fails identically
```

This is strong evidence that X is pre-existing.

If:

```text
base branch:
passes

change branch:
fails
```

the current change owns the regression unless evidence shows otherwise.

---

# Generated Artifacts

Some package changes require regenerated files.

Examples:

- documentation generated from source comments;
- namespace files;
- compiled registration files;
- README generated from source;
- website indexes when repository policy tracks them.

Regenerate only artifacts directly required by the intended change.

Do not use regeneration as permission to update unrelated generated content.

When the local generator version differs from the one the repository uses,
regeneration may rewrite unrelated files, formatting, or recorded tool versions.
Keep only the generated changes that the intended change requires.

Review generated diffs.

---

# Documentation Validation

If public behavior changed, verify that directly affected documentation is
consistent.

Check:

- function help;
- arguments;
- return value;
- examples;
- vignette calls;
- migration guidance;
- changelog/NEWS where project policy requires it.

A code change is incomplete when the public contract and documentation disagree.

---

# Examples and Vignettes

Run examples and vignettes when they are part of the package's normal validation
or when the change affects them.

Pay special attention to:

- renamed functions;
- changed object classes;
- changed dependencies;
- moved functionality;
- optional package use.

Do not disable examples merely to make validation pass unless there is a
documented package-policy reason.

---

# Package Build

Build from the package source, not only from an already loaded development
session.

A build catches issues such as:

- missing files;
- incorrect package metadata;
- generated documentation mismatch;
- undeclared dependencies;
- invalid file layout.

Use the project's standard build mechanism.

---

# Package Check

Run the ecosystem's standard full package check.

For R packages, this normally includes `R CMD check` or an established wrapper
that performs equivalent checks.

For other ecosystems, use the corresponding canonical package validator.

Do not substitute a linter for a package check.

---

# Ecosystem-Specific Checks

Some repositories have additional gates.

Examples:

```text
Bioconductor
    → BiocCheck

CRAN-oriented R package
    → CRAN-like incoming checks where appropriate

compiled package
    → sanitizer / valgrind / platform build when relevant

documentation-heavy package
    → vignette/site build

package with optional integrations
    → optional dependency matrix
```

Run the checks required by the target ecosystem and the nature of the change.

---

# Optional Dependencies

If the changed functionality uses an optional dependency, validate both relevant
states where practical:

```text
dependency available
    → feature works

dependency absent
    → clear expected failure or fallback
```

Do not accidentally convert an optional dependency into a mandatory one.

---

# Numerical Changes

For numerical software, validate both structure and values.

Check where relevant:

- result class;
- dimensions;
- names;
- ordering;
- numerical tolerance;
- convergence information;
- deterministic seed behavior;
- handling of missing values.

Do not rely only on visual similarity.

---

# Multi-Object Data

For changes involving multiple assays, experiments, tables, or datasets, validate:

- sample identifiers;
- feature identifiers;
- ordering;
- intersection rules;
- duplicate handling;
- alignment after subsetting.

A successful model fit does not prove correct alignment.

---

# API Changes

For public API evolution, validate:

```text
new API
    works as documented

preserved behavior
    remains preserved

deprecated/compatibility API
    signals and delegates as intended
```

Do not validate only the new name while ignoring the compatibility contract.

---

# Migration Validation

For migrated functionality:

```text
source behavior
    ≈
destination behavior
```

Compare meaningful contract elements, not volatile implementation attributes.

Then separately validate:

- destination package checks;
- source transition behavior;
- dependency direction.

---

# Refactor Validation

For a pure refactor:

```text
Intended behavioral change:
None.
```

Therefore:

- existing behavior tests should pass unchanged where practical;
- numerical outputs should remain equivalent;
- public docs should not need semantic edits;
- any user-visible difference should trigger reassessment.

---

# Dependency Change Validation

When dependencies change, verify:

- metadata declaration;
- namespace usage;
- optional versus required status;
- behavior with the new dependency path;
- no remaining references to removed dependencies;
- package build from a clean environment where practical.

A dependency change is not complete merely because local development succeeds.

---

# Validation Must Not Expand Scope

During validation, do not opportunistically fix:

- old style warnings;
- unrelated test failures;
- unrelated documentation problems;
- obsolete dependencies;
- neighboring bugs;
- formatting;
- naming inconsistencies.

Instead record them as follow-up work.

Use the question:

> Would this failure exist if the intended change had never been made?

If yes, it probably does not belong in this PR.

---

# Diff Audit

After all checks, inspect the final diff again.

Classify every changed file:

```text
implementation
test/evidence
documentation
generated consequence
package metadata required by change
unrelated
```

Remove the unrelated category.

Validation itself may have generated noise; clean it before review.

---

# Repository Hygiene

Before completion, ensure the change has not introduced:

- temporary debugging files;
- local check directories;
- build artifacts;
- logs;
- downloaded data;
- editor files;
- credentials;
- cache directories.

Do not modify `.gitignore` merely to hide accidental artifacts unless the
project genuinely needs a new ignore rule.

---

# Validation Report

Summarize validation concisely.

Recommended form:

```text
Primary evidence:
PASS — <focused test>

Relevant tests:
PASS — <scope>

Full tests:
PASS

Build:
PASS

Package check:
PASS / known pre-existing NOTE ...

Ecosystem check:
PASS / not applicable

Unrelated failures:
<none, or separately recorded>
```

Do not claim a check passed if it was not run.

---

# When Full Validation Cannot Be Run

State exactly what was and was not validated.

Example:

```text
Validated:
- targeted tests
- complete unit tests

Not validated:
- platform-specific compiled check because platform unavailable

Risk:
<what remains uncertain>
```

Do not replace missing validation with confidence language.

---

# Review Checklist

## Intended behavior

- [ ] Primary evidence passes.
- [ ] The test/check actually exercises the intended change.

## Preservation

- [ ] Relevant neighboring tests pass.
- [ ] Full unit tests pass or failures are attributed.

## Packaging

- [ ] Required generated artifacts are synchronized.
- [ ] Documentation matches public behavior.
- [ ] Package build succeeds.
- [ ] Standard package check succeeds or remaining issues are explained.

## Ecosystem

- [ ] Required ecosystem-specific checks were run.
- [ ] Optional dependency behavior was checked where relevant.

## Scope

- [ ] Failures were attributed before fixing.
- [ ] Unrelated failures remain outside the PR.
- [ ] Validation did not introduce cleanup scope.
- [ ] Final diff still has one semantic purpose.

---

# Completion Criteria

This skill is satisfied when:

- the promised primary evidence passes;
- relevant neighboring behavior is verified;
- the full test suite has been run where practical;
- required documentation/generated artifacts are current;
- package build/check has been performed;
- ecosystem-specific validation has been performed where applicable;
- every discovered failure is attributed;
- failures caused by the change are fixed;
- unrelated failures are documented rather than absorbed;
- the final diff still matches the original semantic scope;
- the validation report states exactly what was run.

---

# Final Rule

> Validation proves the change; it does not authorize a broader one.

Run the narrow evidence first.

Expand outward.

Attribute every failure.

Fix only what the intended change owns.
