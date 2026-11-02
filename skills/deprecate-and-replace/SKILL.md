---
name: deprecate-and-replace
description: "Use when retiring a public API, argument, behavior, representation, or package location and replacing it with a preferred alternative: plan the deprecation path, warn users, keep compatibility during transition, and avoid breaking existing callers."
---

# Deprecate and Replace

## Purpose

Use this skill when retiring a public API, argument, behavior, representation, or package location and replacing it with a preferred alternative.

Its purpose is to provide a clear transition path for users while keeping deprecation work separate from unrelated redesign.

This skill applies to:

- renamed functions;
- renamed arguments;
- relocated functionality;
- superseded interfaces;
- obsolete return formats;
- retired compatibility modes;
- redundant exported functions;
- replacement of legacy APIs;
- migration between packages.

The governing principle is:

> Deprecation is a transition contract, not a cleanup shortcut.

A deprecation should tell users:

```text
what is old
what replaces it
how to migrate
when removal may occur
```

and should preserve behavior during the transition where practical.

---

## Core Principle

Prefer:

```text
old API
    ↓
clear deprecation signal
    ↓
canonical replacement
    ↓
delegation to one implementation
    ↓
eventual removal
```

over:

```text
old API and new API both evolve independently
```

There should be one canonical implementation whenever possible.

The deprecated interface should usually become a compatibility layer, not a second code path.

---

## Relationship to Other Skills

This skill follows naturally from:

- `one-semantic-change`;
- `migrate-functionality-without-redesign`;
- `evolve-public-apis`.

Use this skill after a replacement interface has been chosen.

Do not begin deprecation until there is a clear destination.

---

## Required Deprecation Statement

Before implementation, write:

```text
Deprecated interface:
<function/argument/behavior>

Canonical replacement:
<new interface>

Reason:
<concrete user or architecture problem>

Behavior during transition:
<delegation/alias/warning>

Migration path:
<what users should change>

Removal policy:
<release/version/timeline rule if known>

Non-goals:
- <algorithm redesign>
- <new feature>
- <unrelated cleanup>
```

---

## Procedure

### Step 1 — Ensure the Replacement Is Real

Do not deprecate an interface in favor of a vague future design.

The replacement should already be:

- implemented;
- documented;
- tested;
- usable for the supported old use case.

If the replacement is not ready, finish it first.

---

### Step 2 — Define One Canonical Replacement

Avoid ambiguous migrations such as:

```text
oldFunction() is deprecated;
use functionA(), functionB(), or functionC() depending on preference.
```

Prefer one canonical successor when semantics are equivalent.

If old behavior genuinely splits into multiple new concepts, document the mapping explicitly.

---

### Step 3 — Preserve Behavior Through Delegation

When possible:

```r
oldFunction <- function(...) {
    deprecation_signal(...)
    newFunction(...)
}
```

This keeps:

- one implementation;
- one bug-fix path;
- one result contract;
- lower maintenance.

Do not copy the implementation into both old and new interfaces.

---

### Step 4 — Emit a Useful Signal

A deprecation signal should communicate:

- what is deprecated;
- what to use instead;
- optionally where the replacement lives.

Avoid vague messages such as:

```text
This function is deprecated.
```

Prefer:

```text
`oldFunction()` is deprecated; use `newFunction()` instead.
```

Use the ecosystem's standard deprecation mechanism where available.

---

### Step 5 — Keep the Deprecated Path Thin

The deprecated function or argument should contain only:

```text
compatibility handling
deprecation signal
delegation
```

Do not continue adding features to it.

New development belongs in the canonical replacement.

---

### Step 6 — Test the Transition

Tests should establish:

```text
replacement works
deprecated path signals appropriately
deprecated path preserves behavior where intended
```

For a pure rename:

```text
oldFunction(x) ≡ newFunction(x)
```

during the transition period.

---

### Step 7 — Update Direct Documentation

Update:

- reference documentation;
- examples using the old interface;
- vignettes where the old call is central;
- changelog/NEWS if project policy requires it.

Prefer new documentation to teach the replacement.

Do not keep expanding examples around the deprecated interface.

---

### Step 8 — Stop Creating New Internal Callers

Once deprecated, package internals should use the replacement.

Search for calls to the old interface and migrate internal callers.

The deprecated path should exist primarily for external compatibility.

---

### Step 9 — Observe the Removal Policy

Do not remove deprecated APIs prematurely.

Use:

- project policy;
- semantic versioning rules;
- ecosystem release cadence;
- documented deprecation period;
- package maturity.

Removal should be a separate semantic change.

---

### Step 10 — Remove in a Later PR

When policy allows:

```text
remove deprecated interface
remove deprecation tests
remove obsolete docs
remove compatibility code
remove dependency only if now proven unused
```

Use `remove-proven-dead-code`.

---

## Function Rename Pattern

Preferred transition:

```text
new canonical implementation:
newFunction()

deprecated compatibility wrapper:
oldFunction() → warning → newFunction()
```

Do not maintain two implementations.

---

## Argument Rename Pattern

Suppose:

```text
old argument: exp
new argument: experiment
```

During transition:

1. accept `experiment`;
2. accept `exp` with deprecation signal;
3. reject ambiguous simultaneous conflicting values;
4. map old argument to new semantics;
5. document only the new name in primary examples.

Do not let old and new names drift into different behavior.

---

## Moved Functionality Pattern

If functionality moves between packages:

```text
sourcepkg::oldFunction()
    ↓
deprecation signal
    ↓
destpkg::newFunction()
```

when dependency direction permits.

If direct delegation would create an undesirable dependency cycle, use a transition strategy compatible with package architecture.

Do not create circular dependencies solely for deprecation convenience.

---

## Return Representation Deprecation

If an old result format is being replaced:

- provide a clear conversion path when practical;
- avoid changing algorithm and representation simultaneously;
- document which downstream operations change;
- consider temporary compatibility extractors.

Result-format changes are often more disruptive than function renames.

Use a longer transition when users may serialize or persist results.

---

## Deprecated Behavior Flags

If an argument value or mode is being retired:

```text
method = "legacy"
```

deprecate the specific value if the language/framework supports it clearly.

Do not necessarily deprecate the whole function.

Keep the signal as close as possible to the obsolete behavior.

---

## Alias Versus Deprecation

A permanent alias may be appropriate only when:

- both names are established standards;
- interoperability requires both;
- maintenance cost is negligible and intentional.

Otherwise, prefer one canonical name and a temporary deprecated alias.

Permanent synonyms expand conceptual surface.

---

## Deprecation Warnings Should Not Be Noisy

Avoid repeated warnings deep inside loops or internal calls.

Signal at the public boundary.

If the ecosystem provides once-per-session or lifecycle-aware mechanisms, use them appropriately.

The warning should inform, not overwhelm.

---

## Error After Removal

After a deprecated API is removed, consider whether a targeted error or migration hint is appropriate if the ecosystem supports it.

Do not keep large compatibility layers indefinitely merely to produce a nicer error.

---

## Documentation Strategy

During deprecation:

### Primary docs

Teach the new interface.

### Deprecated docs

State:

```text
Deprecated.
Use X instead.
```

Keep enough information for migration.

Do not duplicate full tutorials for both old and new APIs.

---

## NEWS / Changelog

A useful entry states:

```text
`oldFunction()` is deprecated in favor of `newFunction()`.
The replacement preserves the same analysis behavior.
```

If there are differences, state them explicitly.

Avoid vague statements such as:

```text
API cleanup.
```

---

## Versioning and Removal

Do not invent a universal removal schedule.

Use project and ecosystem policy.

Possible strategies:

```text
deprecate in one release
remove after one or more later releases
```

or:

```text
retain throughout current major version
remove in next major version
```

The key requirement is consistency and predictability.

---

## Pre-Release Packages

For unreleased or experimental APIs, full deprecation may be unnecessary.

A direct rename or removal may be appropriate when:

- no stable users are expected;
- package policy permits breakage;
- the change prevents future confusion.

Still record the change and update all references.

Do not add compatibility machinery that will never be needed.

---

## Mature Packages

For mature packages:

- prefer staged deprecation;
- provide clear migration;
- preserve behavior during transition;
- avoid simultaneous redesign;
- allow enough release time for downstream packages to adapt.

Downstream breakage should be treated as part of the migration cost.

---

## Downstream Package Coordination

If other packages import or call the deprecated API:

1. identify known downstream callers when practical;
2. provide replacement guidance;
3. update sibling packages in separate PRs;
4. avoid removing the old API before ecosystem transition is reasonable.

Do not combine all downstream updates into one giant cross-repository PR unless project tooling requires it.

---

## Generic and Method Deprecation

For S3/S4 APIs, determine whether deprecation applies to:

- generic;
- specific method;
- signature;
- package-owned wrapper.

Avoid creating duplicate generics for replacement when an established generic already exists.

Test dispatch during the transition.

---

## Deprecation and Dependency Direction

A migration may create this temptation:

```text
source package must depend on destination package
only to delegate deprecated calls
```

Evaluate whether that dependency is architecturally acceptable.

Alternatives may include:

- re-export during a transition;
- lightweight local compatibility;
- coordinated release;
- removal at a major version boundary.

Do not introduce long-term dependency cycles for temporary compatibility.

---

## Deprecation and Bugs

A deprecated API should still receive necessary bug fixes while supported, preferably automatically through delegation.

This is another reason to maintain one canonical implementation.

Do not let deprecated interfaces become known-broken before removal unless policy explicitly says they are unsupported.

---

## Deprecation and New Features

Do not add new capabilities only to the deprecated API.

If a feature is needed:

```text
implement in canonical replacement
deprecated wrapper inherits it only when behavior is compatible
```

The old API should not become a competing branch.

---

## Deprecation PR Scope

A good deprecation PR may include:

- compatibility wrapper;
- warning;
- documentation;
- tests;
- NEWS entry;
- internal caller updates.

All of these support one semantic purpose:

```text
begin transition from old API to canonical replacement
```

Do not combine with:

```text
algorithm rewrite
result-class redesign
dependency cleanup
broad refactor
```

---

## Deprecation PR Template

```text
## Deprecated interface

<old API>

## Replacement

<new canonical API>

## Reason

<why transition is needed>

## Transition behavior

<warning + delegation>

## Preserved behavior

- <results>
- <defaults>
- <important semantics>

## Migration

<what users should change>

## Removal

<policy/timeline if known>

## Non-goals

- <redesign>
- <new feature>
- <cleanup>
```

---

## Examples

### Example 1 — Pure Function Rename

```text
Deprecated:
getRDA()

Replacement:
getJointRDA()

Transition:
getRDA() warns and delegates to getJointRDA()

Preserved:
arguments
defaults
result class
algorithm
```

This is a clean deprecation.

---

### Example 2 — Package Migration

```text
Deprecated:
mia::getJointRPCA()

Replacement:
multiomics::getJointRPCA()

Transition:
source package signals migration and delegates if dependency architecture permits
```

Do not redesign joint RPCA in the deprecation PR.

---

### Example 3 — Argument Rename

```text
Deprecated:
exp =

Replacement:
experiment =

Transition:
old argument maps to new argument with warning
conflicting simultaneous use errors clearly
```

---

### Example 4 — Bad Deprecation

```text
Deprecate old function,
change defaults in new function,
switch algorithm,
introduce new result class,
and remove old function immediately.
```

This hides several changes inside one transition.

Split it.

---

## Review Checklist

### Replacement

- [ ] A real canonical replacement exists.
- [ ] Replacement covers the supported old use case.
- [ ] There is one canonical implementation where practical.

### Signal

- [ ] Deprecation message names the replacement.
- [ ] Signal occurs at the public boundary.
- [ ] Warnings are not excessively repeated.

### Compatibility

- [ ] Deprecated path preserves behavior where intended.
- [ ] Internal callers use the replacement.
- [ ] Downstream impact was considered.
- [ ] Dependency direction remains acceptable.

### Scope

- [ ] No algorithm redesign is bundled.
- [ ] No unrelated refactor is bundled.
- [ ] No new feature is specific to the deprecated path.

### Removal

- [ ] Removal policy is explicit or governed by project policy.
- [ ] Immediate deletion is avoided when compatibility requires transition.
- [ ] Eventual removal will be a separate change.

---

## Completion Criteria

This skill is satisfied when:

- the deprecated interface is clearly identified;
- a canonical replacement already exists;
- users receive a clear migration signal;
- the deprecated interface delegates where practical;
- behavior remains stable during transition;
- internal code has moved to the replacement;
- documentation teaches the new API;
- removal timing follows project policy;
- no unrelated redesign is hidden inside the deprecation;
- one implementation remains authoritative.

---

## Final Rule

> Deprecate toward one clear destination.

Warn clearly.

Delegate when possible.

Remove only after the transition has done its job.
