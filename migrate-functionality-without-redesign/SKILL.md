---
name: migrate-functionality-without-redesign
description: Use when moving functionality from one package, module, namespace, file, or subsystem to another while preserving existing behavior: migrate first, verify parity, then redesign later in separate changes; never combine move, rename, redesign, and optimization in one step.
---

# Migrate Functionality Without Redesign

## Purpose

Use this skill when moving functionality from one package, module, namespace, file, or subsystem to another while preserving its existing behavior.

Its purpose is to separate **migration** from **redesign** so that reviewers can verify location changes independently from semantic changes.

This skill applies to:

- moving methods between packages;
- splitting a monolithic package;
- reorganizing modules;
- extracting reusable functionality;
- moving analysis methods to a more appropriate package;
- consolidating duplicated implementations;
- retiring old locations after functionality has moved.

The governing principle is:

> Move first. Prove parity. Redesign later.

Migration should primarily change **where functionality lives**, not **what it does**.

---

## Core Contract

A migration PR should usually state:

```text
Intended behavioral change:
None.
```

The semantic purpose is:

```text
Make existing functionality available from the new location
with equivalent behavior.
```

Migration may legitimately require changes to:

- source code location;
- exports;
- namespace declarations;
- dependency declarations;
- tests;
- documentation;
- examples;
- package references.

It should not simultaneously change:

- algorithm;
- API shape;
- defaults;
- result representation;
- semantics;
- plotting system;
- validation policy;
- naming;

unless one of those changes is strictly required for the move.

---

## Relationship to Other Skills

This skill depends especially on:

- Skill 01 — One Semantic Change;
- Skill 03 — Characterize Before Changing;
- Skill 05 — Design With Existing Representations;
- Skill 07 — Introduce Dependencies Conservatively.

Migration is often safest as a sequence of PRs rather than one large cross-package rewrite.

---

## Required Migration Statement

Before implementation, write:

```text
Functionality being moved:
<function/method/subsystem>

Source:
<current package/module>

Destination:
<new package/module>

Reason for migration:
<architectural or ownership reason>

Behavior to preserve:
- <API>
- <result>
- <defaults>
- <errors>
- <important metadata>

Parity evidence:
<tests or comparisons>

Non-goals:
- <rename>
- <algorithm change>
- <result-class change>
- <optimization>
- <plotting additions>
```

---

## Procedure

### Step 1 — Establish Why the Destination Is Better

Migration should solve a real ownership or architecture problem.

Good reasons include:

- functionality belongs conceptually with another package;
- dependencies are better aligned at the destination;
- the source package should have narrower responsibility;
- multiple packages need the capability;
- functionality has evolved beyond the original package's scope.

Weak reasons include:

- preference for a different file layout;
- desire to "clean things up";
- opportunity to redesign several APIs at once.

---

### Step 2 — Characterize Existing Behavior

Before moving code, establish what the source implementation does.

Characterize as relevant:

- function signature;
- defaults;
- accepted inputs;
- result class;
- dimensions;
- names;
- ordering;
- metadata;
- warnings;
- errors;
- numerical values;
- dependency behavior;
- side effects.

Use the smallest representative inputs practical.

---

### Step 3 — Identify the True Dependencies

Determine what the migrated code actually needs.

Separate:

```text
dependencies genuinely required by the functionality
```

from:

```text
dependencies available merely because the source package already had them
```

At the destination, declare only what the migrated implementation actually requires.

Do not use migration as an excuse for broad dependency cleanup in the source package unless directly caused by the move.

---

### Step 4 — Preserve Existing Representations

Keep existing input and result representations where practical.

Do not introduce a new class merely because the function has moved.

Preferred:

```text
old location:
standard input → existing result

new location:
standard input → same existing result
```

Avoid:

```text
migration
    +
new result wrapper
```

unless the old representation cannot work at the destination.

---

### Step 5 — Port the Implementation With Minimal Change

Move or reproduce only what is required.

Prefer:

- same algorithm;
- same public argument names;
- same defaults;
- same upstream implementation;
- same result representation.

Adjust only what the destination requires, such as:

- namespace qualification;
- imports;
- package references;
- internal helper access.

Do not "improve" the implementation during the parity step.

---

### Step 6 — Replace Source-Private Coupling

If migrated code depended on private internals from the source package, migration may require resolving that coupling.

Use this order:

1. public API from source package;
2. shared public dependency;
3. minimal local implementation;
4. upstream extraction into an appropriate shared package.

Do not preserve `sourcepkg:::` coupling merely to make the move easier.

Any replacement should preserve observable behavior.

---

### Step 7 — Add Destination Tests

Run the same or equivalent behavioral tests at the destination.

Where practical, use identical fixtures.

Test:

- result class;
- dimensions;
- names;
- key values;
- defaults;
- relevant errors;
- method-specific metadata.

Do not redesign the test suite during migration.

---

### Step 8 — Compare Source and Destination Directly

Where both implementations can coexist temporarily:

```text
source_result <- sourcepkg::method(input)
destination_result <- destpkg::method(input)
```

Compare the relevant contract.

For numerical results, use appropriate tolerance.

For complex objects, compare stable meaningful fields rather than every incidental attribute.

---

### Step 9 — Expose the Destination API

Once parity is established:

- export the destination function if intended;
- document it;
- add minimal examples;
- update package metadata as required.

Keep documentation focused on the existing behavior.

Do not use this PR to rewrite the whole workflow tutorial.

---

### Step 10 — Update the Source Separately

After the destination implementation is established, decide what the source package should do.

Possible paths:

```text
delegate
deprecate
re-export
retain temporarily
remove later
```

This is often best handled in a separate PR or release step.

Do not delete the source implementation before destination parity is demonstrated.

---

## Recommended Migration Sequence

For cross-package migration, prefer:

### PR A — Destination parity

```text
Add existing functionality to destination package.
Preserve behavior.
Add parity tests.
```

### PR B — Source transition

```text
Delegate or deprecate the source API.
Update source documentation.
```

### PR C — Later redesign

```text
Rename, simplify, optimize, or change representation if justified.
```

### PR D — Eventual removal

```text
Remove deprecated source implementation according to policy.
```

This sequence makes each change independently reviewable.

---

## Migration Versus Rename

Do not usually combine:

```text
move `getRDA()`
+
rename to `getJointRDA()`
```

even if the new name is better.

Prefer:

```text
PR 1:
move with same name and behavior

PR 2:
rename with dedicated compatibility plan
```

An exception may be reasonable for unreleased code with no compatibility obligation, but the semantic scope should remain explicit.

---

## Migration Versus Algorithm Simplification

Do not combine:

```text
move method
+
replace algorithm implementation
```

Even when the replacement is objectively better.

Migration parity is easier to verify when the algorithm remains stable.

Simplify later with characterized behavior.

---

## Migration Versus New Result Class

Avoid:

```text
move method
+
introduce destination-specific result class
```

Return the same established result first.

If downstream needs later prove a new representation necessary, handle that separately.

---

## Migration Versus New Features

Do not add new method options while moving existing functionality.

Example:

```text
existing cross-association method
+
new mixed-effects support
```

should usually become:

```text
PR 1:
migrate existing method

PR 2:
add mixed-effects capability
```

Migration should not hide feature development.

---

## Migration Versus Refactor

Some mechanical refactoring may be necessary because source-specific helpers cannot move cleanly.

Keep it minimal.

If broad refactoring would improve the code but is not required:

```text
migrate first
refactor later
```

This preserves review clarity.

---

## Shared Helpers During Migration

If several packages need the same helper, do not immediately create a new shared utility package.

First determine whether the helper:

- already exists in a common dependency;
- can remain local at low cost;
- belongs conceptually in one package with a public API.

Create shared infrastructure only when repeated real use justifies it.

---

## Dependency Direction

Migration should improve, not worsen, dependency architecture.

Avoid circular relationships such as:

```text
package A depends on B
while B now depends on A for migrated internals
```

Prefer moving the functionality to a layer that can own its dependencies cleanly.

If migration creates a cycle, reconsider ownership before proceeding.

---

## Namespace and Generic Ownership

When migrating S3/S4 methods or generics, determine:

- which package should own the generic;
- whether an established generic already exists;
- whether the destination should import rather than redefine it;
- how dispatch will behave when both packages are loaded.

Do not create duplicate generics with the same conceptual role.

---

## Data Ownership

If the migrated method uses example datasets, test fixtures, or internal resources from the source package, decide whether to:

- construct minimal destination-local fixtures;
- depend on the source package for tests only;
- move the dataset if ownership genuinely moves.

Do not migrate large datasets automatically with the method.

---

## Documentation During Migration

Destination documentation should explain:

- what the method does;
- expected input;
- returned result;
- upstream implementation where relevant.

Source documentation should later explain:

- new location;
- deprecation or delegation behavior;
- migration path for users.

Avoid duplicate long-form documentation maintained independently in two places.

---

## Compatibility Strategies

### Delegation

Source implementation becomes:

```r
oldFunction <- function(...) {
    newpkg::newFunction(...)
}
```

Use when preserving API temporarily is valuable.

---

### Deprecation + Delegation

Source warns and forwards to destination.

Useful when ownership is changing but users need transition time.

---

### Re-export

May be appropriate when ecosystem conventions support it and maintaining one user-facing access point remains desirable.

Use cautiously; re-exports can blur package ownership.

---

### Hard Removal

Use only when compatibility policy allows it.

Usually not part of the initial migration.

---

## Parity Testing

A parity test should compare only meaningful contract elements.

Possible assertions:

```text
same class
same dimensions
same names
same coefficients
same statistic
same selected features
same metadata used downstream
```

Avoid exact comparison of:

- calls containing namespace differences;
- package-version metadata;
- environment references;
- volatile timing data;
- incidental attributes.

---

## Numerical Parity

For numerical methods, define tolerance deliberately.

Example concept:

```text
destination_result ≈ source_result
```

not necessarily byte-identical.

Document intentional numerical differences if migration forces them.

If numerical differences are significant, migration and algorithm change have become entangled and should be reconsidered.

---

## Migration of Tests

Prefer copying or adapting the source behavioral tests before rewriting them.

The initial destination tests should answer:

```text
Does the method still behave the same?
```

Later, test organization can improve separately.

---

## Migration of Bugs

If the source implementation contains a known bug, decide explicitly:

### Option A — Preserve then fix

Preferred when parity is important:

```text
migrate current behavior
then fix bug separately
```

### Option B — Fix during migration

Acceptable when:

- preserving the bug would create unusable functionality;
- the correction is tiny and unavoidable;
- parity is defined against intended documented behavior rather than the defective implementation.

If choosing B, make the deviation explicit.

Do not silently fix unrelated defects during migration.

---

## Source Cleanup

After callers have transitioned, source cleanup may include:

- removing implementation;
- removing imports;
- removing helpers used only by migrated code;
- removing dependencies used only by migrated code.

Do this only after evidence shows they are no longer required.

Use Skill 09 — Remove Proven Dead Code.

---

## Migration PR Template

```text
## Functionality

<what is moving>

## Source

<current owner>

## Destination

<new owner>

## Reason

<why ownership belongs at destination>

## Intended behavioral change

None.

## Behavior preserved

- <signature>
- <defaults>
- <result>
- <important edge behavior>

## Destination dependencies

<existing public implementations reused>

## Parity evidence

<source/destination comparisons>

## Non-goals

- no rename
- no algorithm redesign
- no new result class
- no optimization
- no unrelated cleanup
```

---

## Examples

### Example 1 — Move Joint RPCA

Preferred sequence:

```text
PR 1:
add joint RPCA to destination package
using the existing algorithm and result representation
with parity tests

PR 2:
update source package to delegate/deprecate

PR 3:
simplify implementation if justified
```

Avoid:

```text
move + rename + optimize + new class + new plot
```

---

### Example 2 — Move Cross-Association

If the source method currently supports basic association and future plans include mixed effects:

```text
migration PR:
move existing basic association behavior

later feature PR:
add mixed-effects support
```

Do not make future capability a prerequisite for correct migration.

---

### Example 3 — Internal Module Split

Current state:

```text
large module contains import and analysis logic
```

Desired destination:

```text
import module
analysis module
```

Migration PR can move the analysis function while preserving API.

Do not simultaneously redesign the parser.

---

### Example 4 — Bad Migration

```text
Move function to new package,
rename arguments,
switch upstream library,
change return type,
and rewrite documentation.
```

This cannot be reviewed as migration parity.

Split it.

---

## Review Checklist

### Ownership

- [ ] The reason for migration is clear.
- [ ] Destination package/module is a better conceptual owner.
- [ ] Dependency direction remains clean.

### Parity

- [ ] Existing behavior was characterized.
- [ ] Destination behavior matches the source where intended.
- [ ] Result representation is preserved.
- [ ] Defaults and important errors remain stable.

### Scope

- [ ] Migration is not combined with redesign.
- [ ] No unrelated refactor is present.
- [ ] No new feature is hidden in the move.
- [ ] No speculative class or framework is introduced.

### Transition

- [ ] Source transition strategy is explicit.
- [ ] Deprecation/removal is separated when appropriate.
- [ ] Users have a clear path to the destination.

---

## Completion Criteria

This skill is satisfied when:

- functionality has a clear destination owner;
- source behavior is characterized;
- destination implementation reproduces that behavior;
- standard representations are preserved;
- required dependencies are explicit and public;
- parity tests pass;
- source transition is planned separately where appropriate;
- no redesign is hidden inside the migration;
- the migration can be reviewed as one semantic change.

---

## Final Rule

> Change ownership first. Change design later.

A migration is successful when users can move to the new location without also having to learn a new behavior.
