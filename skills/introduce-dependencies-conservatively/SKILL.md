---
name: introduce-dependencies-conservatively
description: "Use when adding, removing, replacing, upgrading, or otherwise changing how a package depends on external software: treat dependencies as architectural decisions, prefer the existing dependency graph, depend on public contracts, and keep dependency changes in their own semantic scope."
---

# Introduce Dependencies Conservatively

## Purpose

Use this skill when adding, removing, replacing, upgrading, or changing how a package depends on external software.

Its purpose is to ensure that dependencies are treated as architectural commitments rather than convenience imports.

This skill applies to:

- new package development;
- feature development;
- bug fixes caused by dependency behavior;
- dependency replacement;
- optional feature support;
- interoperability;
- migration work;
- compatibility maintenance;
- package slimming;
- build and installation reliability.

The governing principle is:

> Add a dependency only when the capability it provides is worth the maintenance, compatibility, and conceptual cost it introduces.

Dependencies are not free.

They become part of the package's long-term operating environment.

---

## Core Principle

Prefer:

```text
existing language capability
    ↓
existing package code
    ↓
existing dependency
    ↓
new dependency
```

in that order when the options are otherwise suitable.

A new dependency should solve a concrete current problem.

It should not be added merely because it reduces a few lines of code or offers stylistic convenience.

---

## What a Dependency Costs

Every dependency adds some combination of:

- installation time;
- transitive dependencies;
- version compatibility risk;
- release coordination;
- API coupling;
- platform-specific failures;
- build complexity;
- documentation burden;
- testing burden;
- maintenance work;
- user environment constraints.

For scientific software, it may also add:

- numerical behavior changes;
- compiled-code requirements;
- system libraries;
- reproducibility concerns;
- method-version sensitivity.

Therefore dependency decisions should be explicit.

---

## Required Dependency Statement

Before adding a dependency, write:

```text
Required capability:
<what the package needs>

Existing alternatives considered:
- language/base: <candidate>
- current package: <candidate>
- current dependencies: <candidate>
- ecosystem: <candidate>

Chosen dependency:
<package>

Public API used:
<functions/classes/generics>

Why this dependency is justified:
<concrete benefit>

Dependency role:
<required runtime / optional feature / testing / documentation / build>

Non-goals:
- <unrelated dependency cleanup>
- <broad package modernization>
```

This can be short.

It must show that the dependency was chosen deliberately.

---

## Procedure

### Step 1 — Define the Capability

Describe what the package actually needs.

Good:

```text
We need a maintained implementation of penalized regression with cross-validation.
```

Poor:

```text
We should import package X.
```

The first statement allows comparison.

The second assumes the answer.

---

### Step 2 — Check Existing Capability

Before adding anything new, inspect:

1. the language or standard library;
2. current package code;
3. existing dependencies;
4. the relevant ecosystem.

Do not add a package for behavior already available through a stable dependency you already have.

---

### Step 3 — Distinguish Complex Capability From Convenience

A dependency is more easily justified for:

- mature statistical methods;
- optimization algorithms;
- parsers;
- file formats;
- domain standards;
- complex numerical methods;
- specialized data structures;
- interoperability with major ecosystems.

A dependency is less easily justified for:

- one string helper;
- one list operation;
- one trivial matrix transform;
- one formatting convenience;
- syntax shortening;
- a tiny wrapper around base-language behavior.

---

### Step 4 — Evaluate Dependency Weight

Consider:

```text
What do we gain?
What do we inherit?
```

Inspect:

- size of the dependency tree;
- compiled requirements;
- system requirements;
- maintenance activity;
- API stability;
- licensing compatibility;
- platform support;
- release cadence;
- relevance to the package's core purpose.

A strong dependency should provide substantial durable value.

---

### Step 5 — Use the Narrowest Public API

Depend only on documented, exported interfaces.

Prefer:

```r
pkg::method(...)
```

over:

```r
pkg:::internal_helper(...)
```

Prefer documented accessors over manipulating object internals.

The smaller the dependency surface, the easier future maintenance becomes.

---

### Step 6 — Choose the Correct Dependency Role

Do not make every dependency mandatory.

Classify the dependency according to how the package uses it.

Conceptually:

```text
required for ordinary package functionality
    → required runtime dependency

required only for an optional feature
    → optional dependency

required only for tests
    → testing dependency

required only for vignettes/examples
    → documentation dependency
```

Use the package ecosystem's standard mechanism for expressing those roles.

---

## Required Versus Optional Dependencies

A dependency should usually be required when:

- core exported functionality cannot work without it;
- users reasonably expect the capability to be available after installation;
- nearly all normal workflows need it;
- optional checks would complicate the API more than they help.

A dependency should usually be optional when:

- only one specialized method needs it;
- plotting support is optional;
- a file format is uncommon;
- an integration targets a separate ecosystem;
- users can productively use the package without it.

Do not make a large dependency mandatory solely to support one peripheral feature.

---

## Optional Feature Pattern

When a feature depends on an optional package:

```text
user calls optional feature
        ↓
check dependency availability
        ↓
if unavailable:
    provide clear installation guidance
        ↓
if available:
    call public dependency API
```

Do not check for optional dependencies at package startup if they are only needed for one function.

---

## Avoid Dependency Leakage

An optional dependency should not become accidentally required because:

- a top-level import references it;
- examples run unconditionally;
- tests call it without skipping when it is unavailable;
- package initialization calls it;
- a helper used by core functions assumes it exists;
- documentation tooling loads it globally.

Keep optional integrations isolated.

---

## Public API Boundary

Do not bind package correctness to dependency internals.

Avoid:

- `:::`;
- undocumented slots;
- private helper functions;
- internal environments;
- implementation-specific attributes;
- source-file assumptions.

If the public API is insufficient, consider:

1. another public route;
2. minimal local implementation;
3. upstream contribution;
4. another dependency.

---

## Dependency Replacement

Replacing one dependency with another is a semantic change and should usually have its own PR.

Before replacement:

1. characterize the behavior currently relied upon;
2. identify equivalent public behavior in the replacement;
3. compare outputs and edge cases;
4. preserve user-facing semantics where possible;
5. document intentional differences.

Do not combine replacement with unrelated refactoring.

---

## Dependency Removal

Remove a dependency only when there is evidence that it is unnecessary.

Possible evidence:

- no runtime calls remain;
- functionality was migrated;
- a language-native implementation now covers the use case;
- a dependency became optional;
- dead code was removed;
- another existing dependency already provides the capability.

Do not delete a dependency from metadata merely because direct usage is not obvious.

Check:

- generated code;
- tests;
- vignettes;
- examples;
- namespace imports;
- conditional features;
- compiled code;
- scripts used in release workflows.

---

## Dependency Upgrades

Do not upgrade dependencies casually inside unrelated feature PRs.

Upgrade when:

- required for compatibility;
- needed for a new public API;
- addressing a bug or security issue;
- required by ecosystem release policy;
- intentionally modernizing the package.

Keep version changes minimal.

Do not raise minimum versions without evidence that older versions no longer support required behavior.

---

## Minimum Version Rule

A minimum version should correspond to a concrete required capability.

Good:

```text
Require package >= 2.3 because exported function X was introduced in 2.3.
```

Poor:

```text
Require the latest version because it is newer.
```

Avoid unnecessary exclusion of users and build environments.

---

## Transitive Dependency Rule

Do not rely on a dependency merely because another dependency happens to import it.

If your package directly calls its public API, declare it directly according to ecosystem conventions.

Transitive availability is not a stable contract.

---

## Dependency and Thin Adapters

Dependencies are especially appropriate when the package acts as a thin adapter around a mature implementation.

Preferred pattern:

```text
standard container
    ↓
local integration
    ↓
dependency::method()
    ↓
dependency result
```

This is a strong use of a dependency because the package gains substantial domain capability without recreating the algorithm.

---

## Dependency and Local Code

A few local lines may be preferable to adding a dependency when:

- the operation is trivial;
- behavior is stable;
- implementation is obvious;
- tests are simple;
- no domain expertise is being duplicated.

Example:

```r
log(x + pseudocount)
```

may be preferable to importing a large package solely for that expression.

---

## Dependency and Copying Code

Do not avoid a dependency by copying large chunks of its implementation.

This often creates a worse maintenance problem:

```text
no declared dependency
but hidden duplicated dependency logic
```

If the functionality is substantial, reuse the maintained implementation unless there is a strong reason not to.

---

## Dependency and Reproducibility

For numerical or scientific methods, record enough information to understand which upstream implementation is used.

Where relevant, consider:

- package version;
- method name;
- important defaults;
- algorithm options.

Do not build a provenance system solely for dependency versions unless the package already has one.

Use existing session or environment reporting mechanisms when sufficient.

---

## Dependency and Testing

Tests should verify the integration contract.

For required dependencies:

- test the package behavior directly.

For optional dependencies:

- skip or conditionally run tests according to ecosystem standards;
- test the missing-dependency error path;
- test the feature when the dependency is available.

Do not duplicate the dependency's full test suite.

---

## Dependency and Documentation

Documentation should make dependency boundaries clear when users need to know them.

For optional features, state:

```text
This feature requires package X.
```

For delegated algorithms, acknowledge the implementation:

```text
This function uses `pkg::method()` internally.
```

Do not obscure the origin of established algorithms.

---

## Dependency and Namespace Hygiene

Import only what is needed.

Prefer narrow imports or explicit namespace qualification when appropriate.

Avoid attaching large packages solely for a few functions.

Explicit calls such as:

```r
pkg::function()
```

can make dependency ownership obvious and reduce namespace ambiguity.

Follow the conventions of the package ecosystem and project.

---

## Dependency Name Conflicts

If two dependencies export conflicting names:

- prefer explicit namespace qualification;
- avoid creating wrapper names solely to hide the conflict;
- use established generics where appropriate.

Do not let namespace conflicts drive unnecessary architecture.

---

## Dependency Failure Modes

For important dependencies, consider likely failure modes:

- API removal;
- changed defaults;
- changed result class;
- changed numerical behavior;
- unavailable system library;
- package archival;
- incompatible major version.

Mitigate through:

- narrow public API usage;
- behavioral tests;
- explicit version bounds only when needed;
- optional isolation for peripheral integrations.

Do not over-engineer fallback systems for hypothetical failures.

---

## Optional Backend Warning

Avoid introducing a generic "backend" architecture merely because two dependencies can implement similar methods.

Prefer explicit functions or small dispatch until repeated current requirements justify a backend abstraction.

For example, do not create:

```text
ModelBackend
BackendRegistry
BackendConfig
```

only because users may someday choose between two algorithms.

---

## One Dependency Change Per PR

Dependency additions, removals, and replacements should generally be semantically isolated.

Good:

```text
PR:
Add dependency X to support method Y.
```

Good:

```text
PR:
Remove dependency Z after the final caller was removed.
```

Poor:

```text
PR:
Add new feature, remove two old dependencies, update all minimum versions, and reformat DESCRIPTION.
```

---

## PR Scope Template

```text
## Capability

<What capability requires dependency work?>

## Decision

<Add / remove / replace / make optional / update>

## Reuse analysis

- Language/base: <result>
- Current dependencies: <result>
- Ecosystem: <result>

## Public API used

<functions/classes/generics>

## Dependency role

<required / optional / test / documentation>

## Version requirement

<none or concrete reason>

## Behavior preserved

- <existing behavior>
- <existing behavior>

## Non-goals

- <unrelated dependency cleanup>
- <unrelated modernization>
```

---

## Anti-Patterns

### Dependency for One Trivial Helper

Avoid adding a package solely to shorten a few obvious lines.

---

### Private API Dependency

Avoid:

```r
pkg:::helper()
```

as part of production package logic.

---

### Mandatory Heavy Dependency for Peripheral Feature

Avoid making all users install a large stack for one optional plotting or import function.

---

### Latest-Version Inflation

Avoid raising minimum versions without a concrete required API.

---

### Unrelated Dependency Cleanup

Avoid removing or upgrading dependencies while implementing an unrelated feature.

---

### Dependency Avoidance Through Copy-Paste

Avoid copying substantial maintained algorithms into the package merely to avoid declaring a dependency.

---

### Framework for Multiple Backends Too Early

Avoid abstract backend systems before multiple real integrations require them.

---

## Examples

### Example 1 — Established Statistical Method

Requirement:

```text
Add penalized regression.
```

Evaluation:

```text
base language:
no equivalent mature implementation

current dependencies:
none suitable

established package:
glmnet provides maintained implementation
```

Decision:

```text
reuse glmnet
```

Local responsibility:

```text
extract data
align observations
prepare inputs
call public glmnet API
return established result
```

This is a strong dependency justification.

---

### Example 2 — Simple Transformation

Requirement:

```text
Add log(x + pseudocount).
```

Evaluation:

```text
base language already provides required behavior.
```

Decision:

```text
implement locally using base operations.
```

Do not add a dependency.

---

### Example 3 — Optional Plotting Package

Requirement:

```text
Provide an example visualization of an upstream ordination object.
```

If plotting is not core:

```text
keep plotting dependency optional
```

and document the workflow.

Do not make it a required runtime dependency solely for vignette aesthetics.

---

### Example 4 — Removing a Dependency

Evidence:

```text
all calls to package X were removed in earlier PRs
no tests or vignettes require it
namespace no longer imports it
```

Then a dedicated PR can remove X.

Do not combine this with unrelated refactoring.

---

### Example 5 — Private API Breakage

Current code:

```r
mia:::.internal_transform(...)
```

Preferred resolution:

```text
1. search for exported equivalent;
2. if unavailable, implement only the small required operation locally;
3. preserve behavior with tests;
4. remove private coupling.
```

Do not simply pin an old dependency version unless absolutely necessary.

---

## Review Checklist

### Need

- [ ] The required capability is explicit.
- [ ] Existing language and package options were considered.
- [ ] The dependency provides substantial enough value.

### Boundary

- [ ] Only public APIs are used.
- [ ] Dependency surface is narrow.
- [ ] Transitive dependencies are not relied upon implicitly.

### Role

- [ ] Required versus optional status is justified.
- [ ] Optional dependencies do not leak into core functionality.
- [ ] Test/documentation-only dependencies are classified appropriately.

### Versioning

- [ ] Minimum versions have concrete justification.
- [ ] Unnecessary upgrades are absent.
- [ ] Compatibility behavior is tested where relevant.

### Scope

- [ ] Dependency work has one semantic purpose.
- [ ] No unrelated cleanup is bundled.
- [ ] No speculative backend framework was introduced.

---

## Completion Criteria

This skill is satisfied when:

- the capability requiring dependency work is explicit;
- language-native and existing dependency options were evaluated;
- the chosen dependency is justified by concrete value;
- only public stable interfaces are used;
- dependency role is appropriate;
- optional integrations remain optional;
- version constraints are evidence-based;
- transitive assumptions are avoided;
- tests cover the package's integration boundary;
- unrelated dependency changes are excluded;
- the PR has one semantic purpose.

---

## Final Rule

> Dependencies should carry more complexity out of the package than they bring into it.

Add them for substantial capability.

Use them through stable public contracts.

Keep them only as long as they continue to earn their place.
