---
name: evolve-public-apis
description: "Use when changing a package's public interface: renaming functions, adding or removing arguments, changing defaults or return types, or otherwise altering exported contracts; preserve compatibility, update documentation and tests, and keep API changes to their own semantic scope."
---

# Evolve Public APIs

## Purpose

Use this skill when changing a package's public interface.

Its purpose is to make API evolution deliberate, reviewable, and compatible with the package's maturity and user obligations.

This skill applies to:

- renaming exported functions;
- renaming arguments;
- changing defaults;
- changing return values;
- adding or removing arguments;
- changing accepted input types;
- changing result classes;
- changing side effects;
- introducing replacement interfaces;
- consolidating overlapping functions;
- reducing conceptual collisions;
- preparing deprecations.

The governing principle is:

> Change public contracts only to solve a concrete user or maintenance problem, and change one contract dimension at a time where practical.

Public APIs are long-lived commitments.

Internal inconvenience alone is not sufficient reason to change them.

---

## Core Principle

A public API change should answer:

```text
What concrete problem does the current interface create?
```

and:

```text
Why is changing the public contract better than adapting internally?
```

Prefer preserving stable APIs when internal change can solve the problem.

When public change is justified, make the smallest coherent contract change.

---

## What Counts as Public API

Public API includes more than exported function names.

It may include:

- exported functions;
- exported classes;
- generics and methods;
- argument names;
- defaults;
- accepted input types;
- return classes;
- result structure;
- names and ordering;
- warnings and errors when relied upon;
- side effects;
- documented metadata placement;
- file formats;
- serialization behavior;
- re-exported interfaces;
- command-line options;
- configuration keys.

Treat documented and widely used behavior as part of the contract even when not formally typed.

---

## Required API Change Statement

Before implementation, write:

```text
Current contract:
<what users call or receive today>

Problem:
<concrete issue with current contract>

Proposed contract:
<what changes>

Why internal adaptation is insufficient:
<reason>

Compatibility plan:
<none / alias / delegation / deprecation / transition>

Behavior preserved:
- <what does not change>

Non-goals:
- <implementation refactor>
- <new feature>
- <result redesign>
```

---

## Procedure

### Step 1 — Identify the Contract Dimension

Determine exactly what is changing.

Examples:

```text
function name
argument name
default value
accepted input class
return class
return field name
side effect
```

Avoid changing multiple dimensions at once unless they are inseparable.

---

### Step 2 — State the User Problem

Good reasons include:

- name collides with an established ecosystem concept;
- current name is misleading;
- argument semantics are ambiguous;
- default is unsafe or consistently inappropriate;
- return type prevents interoperability;
- interface duplicates another established API;
- function location changed and ownership needs clarification.

Weak reasons include:

- personal naming preference;
- code looks cleaner;
- new style guide;
- internal implementation changed.

---

### Step 3 — Check Existing Ecosystem Conventions

Before inventing a new public name or pattern, inspect:

- established generics;
- neighboring packages;
- domain conventions;
- standard argument names;
- common result types.

Prefer familiar vocabulary over package-specific terminology.

---

### Step 4 — Characterize Existing Behavior

Before changing the contract, establish:

- current arguments;
- defaults;
- result class;
- dimensions;
- names;
- errors;
- important examples;
- downstream use where known.

This provides the preservation boundary.

---

### Step 5 — Choose the Smallest Contract Change

If the problem is naming, change the name.

Do not also change:

- defaults;
- result type;
- algorithm;
- plotting behavior.

If the problem is a default, change the default.

Do not also rename the function.

Keep semantic scope narrow.

---

### Step 6 — Decide Compatibility Strategy

Choose deliberately among:

```text
hard change
alias
delegation
deprecation
re-export
versioned transition
```

The correct strategy depends on:

- package maturity;
- release status;
- user base;
- ecosystem policy;
- severity of the current problem.

---

## Rename Strategy

For a function rename, prefer:

```text
oldName()
    ↓
newName()
```

with identical behavior during transition when compatibility matters.

Do not duplicate implementation.

The old name should delegate to the new implementation or vice versa.

---

## Argument Rename Strategy

For argument renaming:

- support the old name temporarily when policy requires;
- detect conflicting use of old and new names;
- document the replacement;
- keep semantics identical during the transition.

Avoid silently interpreting two names differently.

---

## Default Change Strategy

Changing a default is a behavioral API change.

Treat it independently.

Document:

```text
old default
new default
reason
behavioral consequence
```

Add tests for:

```text
implicit call
explicit old value
explicit new value
```

Do not hide a default change inside a refactor.

---

## Return-Type Change Strategy

Changing a result representation is significant.

Before doing so, ask:

```text
Can the existing result be preserved while adding the needed capability elsewhere?
```

If a new return type is necessary:

- document conversion or compatibility;
- test downstream behavior;
- avoid simultaneous algorithm change;
- preserve names and semantics where possible.

---

## Adding Arguments

Add an argument only when current behavior cannot be expressed cleanly without it.

Prefer arguments that:

- represent one clear choice;
- match ecosystem terminology;
- have stable semantics;
- avoid overlapping with `...`.

Do not accumulate many switches that turn one function into a workflow engine.

---

## Removing Arguments

Removing a public argument usually requires deprecation unless compatibility obligations are minimal.

Before removal, determine:

- whether it is used;
- whether it is documented;
- whether another argument replaces it;
- whether it has been ineffective or ignored.

An ignored argument may still require a transition if users rely on its existence.

---

## Broadening Accepted Inputs

Supporting additional standard input classes can be a backward-compatible API evolution.

Prefer existing dispatch mechanisms.

Do not create a universal coercion layer unless required.

Test that existing inputs remain unchanged.

---

## Narrowing Accepted Inputs

Rejecting previously accepted inputs is breaking behavior.

Only do so when:

- old behavior was incorrect or unsafe;
- support is impossible to maintain;
- ecosystem standards changed;
- deprecation has occurred.

Make the compatibility impact explicit.

---

## Generic Naming

Before creating or renaming a generic:

- check for established generics;
- avoid semantic collisions;
- reuse existing names when contracts match;
- avoid duplicate generics with subtly different meanings.

Generic ownership should be clear.

---

## Name Clarity Rule

A public name should communicate the package-level distinction users need.

For example, if:

```text
getRDA()
```

is already an established single-dataset concept, a multi-dataset version may need a clearer name such as:

```text
getJointRDA()
```

if that distinction is genuinely meaningful.

The rename itself should not change the method.

---

## Compatibility Levels

### Pre-Release or Experimental API

Hard changes may be acceptable when:

- package has no stable release obligation;
- change prevents future confusion;
- migration cost is low.

Still document the change.

---

### Released but Young API

Prefer:

- alias;
- warning;
- staged transition.

---

### Mature Widely Used API

Prefer deliberate deprecation cycles and strong compatibility.

Avoid churn for cosmetic improvements.

---

## Deprecation Boundary

This skill covers deciding that an API should change and introducing the replacement.

Detailed deprecation mechanics belong in `deprecate-and-replace`.

---

## API Change and Documentation

A public contract change should update:

- reference docs;
- examples;
- vignettes that directly use the interface;
- NEWS/changelog where project policy expects it.

Do not broadly rewrite unrelated documentation.

---

## API Change and Tests

Tests should cover:

```text
new contract
preserved behavior
compatibility path if present
```

For pure rename:

```text
newName(input) ≡ oldName(input)
```

during the compatibility period.

---

## API Change and Implementation

Do not combine public API change with deep implementation rewrite unless necessary.

Preferred:

```text
PR 1:
change public name with same implementation

PR 2:
refactor implementation
```

This lets reviewers separate compatibility from correctness.

---

## API Change and Migration

When functionality is moving packages:

```text
migrate first
evolve API second
```

unless the old name is impossible or harmful at the destination.

Keeping migration and API redesign separate simplifies parity testing.

---

## API Change and New Features

Do not use an API rename as an opportunity to add new options.

Example:

```text
rename `getRDA()` → `getJointRDA()`
```

should not simultaneously add:

```text
new scaling modes
new plotting result
new preprocessing switches
```

---

## API Churn Warning

A package becomes difficult to trust when names and contracts change frequently.

Before changing public API, ask:

```text
Is this problem severe enough to justify user migration cost?
```

Prefer internal adaptation for minor inconsistencies.

---

## Avoid Synonym Proliferation

Do not keep many permanent aliases for the same concept.

Temporary compatibility aliases are useful.

Permanent synonym sets create:

- documentation duplication;
- user confusion;
- maintenance cost.

Choose one canonical API.

---

## API Surface Reduction

Sometimes simplification means removing redundant public functions.

Do this carefully.

Prefer consolidation when:

- two functions have the same semantics;
- one is a historical alias;
- one can delegate cleanly.

Use deprecation policy for released APIs.

---

## Public Versus Internal Helpers

Do not export a helper merely because another internal function needs it.

Public API should represent user concepts, not implementation convenience.

Once exported, removal becomes harder.

---

## API PR Template

```text
## Current contract

<current public behavior>

## Problem

<why it should change>

## Proposed contract

<new behavior>

## Compatibility

<transition plan>

## Behavior preserved

- <algorithm>
- <result>
- <defaults not changing>

## Evidence

<tests demonstrating new and preserved behavior>

## Non-goals

- <refactor>
- <new feature>
- <migration>
```

---

## Examples

### Example 1 — Function Rename

Problem:

```text
`getRDA()` is ambiguous because the ecosystem already uses that name for a different scope.
```

Change:

```text
rename to `getJointRDA()`
```

Preserve:

```text
arguments
defaults
algorithm
result class
errors
```

Compatibility:

```text
old name delegates during transition if required
```

---

### Example 2 — Argument Rename

Problem:

```text
argument `exp` is ambiguous
```

Change:

```text
rename to `experiment`
```

Do not also change accepted values.

---

### Example 3 — Default Change

Problem:

```text
current default selects a behavior known to be unsafe for ordinary use
```

Change:

```text
change default only
```

Keep explicit old behavior available if appropriate.

---

### Example 4 — Return-Type Change

Problem:

```text
current custom wrapper prevents use of established upstream plotting and summary methods
```

Potential change:

```text
return upstream result directly
```

This should be its own API PR, with compatibility and downstream tests.

---

### Example 5 — Bad API PR

```text
Rename function,
change defaults,
replace algorithm,
introduce new class,
and rewrite plotting.
```

This is several semantic changes.

Split it.

---

## Review Checklist

### Necessity

- [ ] The current API problem is concrete.
- [ ] Internal adaptation was considered.
- [ ] Ecosystem conventions were checked.

### Scope

- [ ] One contract dimension is changing where practical.
- [ ] No unrelated refactor is bundled.
- [ ] No new feature is hidden in the change.

### Compatibility

- [ ] Package maturity informs transition strategy.
- [ ] Aliases/deprecation are used appropriately.
- [ ] One canonical replacement API is clear.

### Behavior

- [ ] Unchanged semantics are characterized.
- [ ] Tests cover the new contract.
- [ ] Return values/defaults remain stable unless they are the intended change.

### Documentation

- [ ] Directly affected docs are updated.
- [ ] Migration path is clear.
- [ ] Unrelated docs remain untouched.

---

## Completion Criteria

This skill is satisfied when:

- the current contract and its problem are explicit;
- the proposed change is narrowly scoped;
- ecosystem conventions have been considered;
- compatibility strategy matches package maturity;
- unchanged behavior is preserved;
- tests demonstrate the new contract;
- documentation reflects the change;
- no unrelated implementation redesign or feature work is bundled;
- users have one clear canonical API after the transition.

---

## Final Rule

> Public APIs should change more slowly than implementations.

Change the contract only when the user-facing benefit justifies the migration cost.
