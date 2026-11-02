---
name: refactor-without-behavioral-change
description: "Use when changing code structure while intentionally preserving observable behavior: state that the intended behavioral change is none, establish a behavioral boundary, provide parity evidence, and keep refactoring separate from new behavior."
---

# Refactor Without Behavioral Change

## Purpose

Use this skill when changing code structure while intentionally preserving observable behavior.

Its purpose is to make refactoring reviewable, testable, and safe by separating structural improvement from functional change.

This skill applies to:

- simplifying internal logic;
- extracting helpers;
- removing duplication;
- reorganizing internal code;
- replacing internal implementation patterns;
- renaming private helpers;
- improving readability;
- consolidating repeated validation;
- simplifying control flow;
- reducing technical debt.

The governing principle is:

> A refactor changes how the code is organized, not what users observe.

If observable behavior changes, that work is not purely a refactor and should usually be scoped separately.

---

## Core Contract

Every refactor PR should state:

```text
Intended behavioral change:
None.
```

That statement is a commitment.

The refactor may change:

- file structure;
- helper boundaries;
- private names;
- control flow;
- internal representations;
- local duplication;
- implementation strategy.

It should preserve, where applicable:

- public function names;
- arguments;
- defaults;
- accepted inputs;
- return types;
- dimensions;
- names;
- ordering;
- metadata;
- warnings;
- errors;
- side effects;
- numerical behavior within appropriate tolerance.

---

## Relationship to Other Skills

This skill depends especially on:

- `one-semantic-change`;
- `characterize-before-changing`.

Before refactoring poorly tested code, establish the behavior that must remain stable.

Do not use refactoring as a vehicle for unrelated fixes, redesign, modernization, or API changes.

---

## When Refactoring Is Justified

A refactor is appropriate when there is a concrete maintenance problem such as:

- duplicated semantic logic;
- difficult-to-understand control flow;
- repeated bug-prone code;
- overly broad helper responsibilities;
- internal coupling blocking safe maintenance;
- unnecessary intermediate abstractions;
- dead indirection;
- private API dependence that can be removed without changing behavior;
- clear opportunities to reuse an existing internal or ecosystem abstraction.

Avoid refactoring solely because code style differs from personal preference.

---

## Required Refactor Statement

Before implementation, write:

```text
Maintenance problem:
<what makes the current implementation harder or riskier to maintain>

Intended structural change:
<what internal structure will change>

Intended behavioral change:
None.

Behavior to preserve:
- <public contract>
- <important edge behavior>

Evidence:
<tests or comparisons establishing equivalence>

Non-goals:
- <bug fix>
- <API change>
- <dependency change>
- <formatting cleanup>
```

---

## Procedure

### Step 1 — Define the Structural Problem

Describe the maintenance issue without proposing a broad rewrite.

Good:

```text
The same sample-alignment rule is implemented independently in three methods,
which risks inconsistent fixes.
```

Good:

```text
A private helper combines selection, validation, transformation, and fitting,
making method-specific changes difficult to reason about.
```

Poor:

```text
The code is messy.
```

Poor:

```text
We should modernize this file.
```

---

### Step 2 — Establish Behavioral Coverage

Before changing structure, verify that relevant behavior is protected.

Use:

- existing tests;
- focused characterization tests;
- parity comparisons;
- numerical assertions;
- class/dimension/name checks.

Do not refactor first and then invent tests around the new implementation.

---

### Step 3 — Identify the Refactor Boundary

Choose one coherent structural improvement.

Examples:

```text
extract one repeated alignment rule into one helper
```

```text
replace duplicated branching with one existing generic
```

```text
split one oversized helper into two helpers with distinct responsibilities
```

```text
remove one redundant wrapper layer
```

Avoid broad goals such as:

```text
clean all utilities
```

---

### Step 4 — Preserve the Public Surface

Unless changing the public API is the stated purpose of another PR, do not alter:

- exported names;
- signatures;
- defaults;
- documented return objects;
- public classes;
- documented side effects.

If public changes are discovered to be desirable, record them separately.

---

### Step 5 — Prefer Existing Abstractions

Before extracting a new helper or introducing a new structure, check whether:

- the language already provides the operation;
- the package already has a suitable helper;
- the ecosystem already has the correct generic or accessor.

Refactoring should not increase conceptual vocabulary unnecessarily.

---

### Step 6 — Make One Structural Change

Avoid combining several unrelated refactors.

For example, do not simultaneously:

```text
extract helpers
rename private functions
move files
replace loops
change validation
reformat everything
```

unless all are inseparable consequences of one structural goal.

Prefer multiple reviewable PRs.

---

### Step 7 — Keep Tests Behavior-Focused

Existing behavioral tests should ideally remain unchanged.

If a test must change only because it asserted a private helper or internal structure, improve the test boundary carefully.

Do not rewrite tests merely to make the new implementation pass.

A strong refactor leaves public tests untouched.

---

### Step 8 — Compare Before and After

Verify equivalence for relevant cases.

For deterministic outputs:

```text
old result ≈ new result
```

For objects:

```text
same class
same dimensions
same names
same relevant metadata
same values
```

For errors:

```text
same condition or compatible public behavior
```

For side effects:

```text
same externally visible modification
```

---

### Step 9 — Inspect for Accidental Improvements

Refactors often expose tempting opportunities.

Examples:

- correcting an inconsistent error message;
- changing a default;
- fixing an edge case;
- improving a return type;
- removing a dependency;
- renaming a public function.

Do not include them unless they are required for the refactor.

Record them as separate work.

---

### Step 10 — Review the Diff Semantically

For every change, ask:

```text
Is this line required to achieve the stated structural improvement
while preserving behavior?
```

If not, revert it.

---

## Refactor Categories

## Extract Repeated Semantic Logic

Good candidate:

```text
Three methods implement the same sample-alignment rule.
```

Refactor:

```text
one internal alignment helper
```

Requirements:

- rule is genuinely identical;
- helper has one clear purpose;
- callers become easier to read;
- no new general framework is introduced.

Do not extract merely similar syntax with different semantics.

---

## Remove Redundant Wrapper

Good candidate:

```text
helper A does nothing except call helper B with identical arguments.
```

If no contract depends on A, remove the indirection.

Do not keep wrappers solely for symmetry.

---

## Split Overloaded Helper

Good candidate:

```text
one helper selects data, validates samples, transforms values, and fits a model.
```

Potential refactor:

```text
prepare inputs
fit model
```

Only split where responsibilities are genuinely distinct.

Do not fragment code into many tiny helpers that make control flow harder to follow.

---

## Consolidate Validation

Consolidate validation only when several callers enforce the same semantic rule.

Good:

```text
all methods require unique sample identifiers
```

Weak:

```text
several methods happen to contain `if` statements
```

Do not build a generalized validation engine.

---

## Replace Custom Logic With Existing Functionality

A refactor may reduce code by delegating to an established public function.

Example:

```text
custom matrix-name matching
    ↓
existing ecosystem alignment mechanism
```

This is appropriate if behavior remains equivalent.

Use characterization tests to confirm parity.

---

## Move Code Between Files

File movement can be a refactor if:

```text
behavioral change:
none
```

Keep file-reorganization PRs narrow.

Avoid changing implementation, naming, and behavior at the same time.

Moving code creates noisy diffs, so minimizing unrelated edits is especially important.

---

## Rename Private Helpers

Private renames can be legitimate refactors when names materially obscure responsibility.

Keep them separate from behavioral changes where practical.

Do not rename public functions in a refactor PR unless public renaming is itself the semantic purpose.

---

## Simplify Control Flow

Examples:

- replace deeply nested branches with early returns;
- collapse equivalent branches;
- remove unreachable conditions;
- use an existing standard helper.

Tests should establish that supported cases behave identically.

Do not silently remove edge cases merely because they appear unused.

---

## Remove Internal Abstraction

Refactoring can simplify architecture by deleting unnecessary layers.

Example:

```text
AnalysisManager
    ↓
direct function call
```

when the manager adds no real semantics.

This is often a valuable simplification.

Preserve external behavior while reducing internal concepts.

---

## Refactoring Numerical Code

Use extra caution.

Characterize:

- key numerical values;
- tolerances;
- orientation;
- dimensions;
- names;
- convergence flags;
- randomness/seeds where relevant.

Do not assume algebraic equivalence implies identical numerical behavior.

If meaningful numerical behavior changes, treat that as a separate functional change.

---

## Refactoring S4 or Formal Dispatch

When changing method organization:

- preserve generic names;
- preserve signatures;
- preserve dispatch behavior;
- preserve inheritance assumptions;
- avoid creating new generics unless necessary.

Test at the generic interface, not only individual methods.

---

## Refactoring Around Dependencies

If refactoring changes how a dependency is called but preserves behavior:

- use only public APIs;
- preserve result class where relevant;
- avoid changing dependency versions in the same PR unless required.

If the purpose is dependency replacement, use a dedicated dependency-change scope.

---

## Refactoring Tests

Do not refactor production code and broadly refactor tests simultaneously.

Test-only cleanup can obscure whether behavioral coverage changed.

Acceptable test changes include:

- adding characterization coverage;
- updating tests that incorrectly relied on private structure;
- removing duplicated setup directly caused by the new stable helper boundary.

Keep unrelated test cleanup separate.

---

## Refactoring Documentation

Documentation normally should not change for a purely internal refactor.

Update documentation only if:

- internal documentation is now incorrect;
- developer-facing comments describe removed structure;
- generated documentation changes mechanically.

Public examples should remain unchanged if public behavior remains unchanged.

---

## Refactor Smells

Stop and reassess if the PR begins to include:

```text
"while here"
"also fix"
"also rename"
"also update dependency"
"also change default"
"also improve errors"
"also modernize"
```

These are likely separate semantic changes.

---

## Avoid Premature Generalization

A common refactor mistake is replacing concrete repeated code with an abstraction broader than current needs.

Prefer:

```text
align_common_samples(a, b)
```

over:

```text
UniversalMultiObjectAlignmentEngine(...)
```

unless repeated requirements justify the latter.

Refactoring should reduce concepts, not create speculative ones.

---

## Duplication Rule

Not all duplication is harmful.

Two code fragments may be duplicated because they represent different domain rules that happen to look similar.

Before deduplicating, ask:

```text
If one behavior changed in the future,
would the other necessarily need to change too?
```

If no, they may not share one abstraction.

Prefer clear duplication over false unification.

---

## Small Helper Rule

Create a helper when it captures a stable semantic operation.

Good:

```text
align samples shared by all joint methods
```

Weak:

```text
wrapper around `nrow(x) > 0`
```

Helpers should reduce conceptual repetition, not merely physical repetition.

---

## Refactor PR Template

```text
## Maintenance problem

<What structural issue exists?>

## Structural change

<What internal organization changes?>

## Intended behavioral change

None.

## Behavior preserved

- <public API>
- <result contract>
- <important edge behavior>

## Evidence

<tests or parity checks>

## Non-goals

- <bug fix>
- <API redesign>
- <dependency cleanup>
- <formatting>
```

---

## Examples

### Example 1 — Extract Shared Alignment

Current state:

```text
getMethodA() contains sample alignment
getMethodB() contains same sample alignment
getMethodC() contains same sample alignment
```

Refactor:

```text
extract one internal alignment helper
```

Evidence:

```text
all method tests unchanged
alignment tests preserved
```

Non-goals:

```text
no change to intersection policy
no new partial-overlap support
no public API changes
```

---

### Example 2 — Remove Unnecessary Wrapper

Current state:

```text
.internal_fit(x, ...)
    ↓
upstream::fit(x, ...)
```

and `.internal_fit()` adds no validation or semantics.

Refactor:

```text
call upstream::fit() directly
remove wrapper
```

Behavior remains unchanged.

---

### Example 3 — Split Overloaded Utility

Current helper:

```text
selects experiment
aligns samples
transforms values
fits model
formats result
```

Refactor only if the decomposition clarifies stable responsibilities.

Possible first PR:

```text
separate model fitting from input preparation
```

Do not simultaneously redesign the result format.

---

### Example 4 — Bad Refactor

```text
Refactor utilities, rename public functions, improve error messages,
remove dplyr, and fix a numeric-selection bug.
```

This is several semantic changes.

Split it.

---

## Review Checklist

### Scope

- [ ] The PR states `Intended behavioral change: None`.
- [ ] One structural goal is clear.
- [ ] Unrelated cleanup is absent.
- [ ] No public redesign is hidden inside the refactor.

### Safety

- [ ] Relevant behavior was characterized first.
- [ ] Public tests remain stable where possible.
- [ ] Numerical behavior is compared appropriately.
- [ ] Errors and side effects are preserved where relevant.

### Simplicity

- [ ] Existing abstractions were considered.
- [ ] New helpers have clear semantic purpose.
- [ ] False abstraction was avoided.
- [ ] Internal conceptual complexity is reduced or clarified.

### Diff Quality

- [ ] File movement is not mixed with unrelated edits.
- [ ] Formatting noise is minimized.
- [ ] Every line supports the structural goal.

---

## Completion Criteria

This skill is satisfied when:

- the maintenance problem is explicit;
- the structural change is narrow;
- intended behavioral change is none;
- relevant behavior is characterized;
- public contracts remain stable;
- tests protect observable behavior;
- internal structure is simpler or clearer;
- no unrelated bug fix, API change, dependency change, or cleanup is bundled;
- the final PR can be reviewed as one structural change.

---

## Final Rule

> Refactor structure, not semantics.

If behavior changes, name that change separately and give it its own scope.
