---
name: remove-proven-dead-code
description: Use when removing code, helpers, branches, files, dependencies, exports, tests, or compatibility paths believed to be unused or obsolete: delete only with evidence that the code is unused, replaced, unreachable, or unsupported by the public contract; delete one coherent group at a time.
---

# Remove Proven Dead Code

## Purpose

Use this skill when removing code, helpers, branches, files, dependencies, exports, tests, or compatibility paths that are believed to be unused or obsolete.

Its purpose is to make deletion safe, reviewable, and evidence-based.

This skill applies to:

- unused internal helpers;
- obsolete compatibility code;
- dead branches;
- duplicated legacy implementations;
- unused exports;
- superseded utility functions;
- stale files;
- obsolete feature flags;
- retired migration paths;
- unused dependencies after callers are removed.

The governing principle is:

> Delete code only after establishing that the supported package behavior no longer depends on it.

Deletion is valuable.

Speculative deletion is risky.

---

## Core Principle

Prefer:

```text
identify candidate
    ↓
prove non-use or obsolescence
    ↓
delete one coherent unit
    ↓
run relevant checks
```

over:

```text
large cleanup
    ↓
delete anything that looks unused
    ↓
discover hidden consumers later
```

Dead-code removal should reduce maintenance surface without altering supported behavior.

---

## Relationship to Other Skills

This skill depends especially on:

- Skill 01 — One Semantic Change;
- Skill 03 — Characterize Before Changing;
- Skill 08 — Refactor Without Behavioral Change.

A dead-code-removal PR should generally state:

```text
Intended behavioral change:
None.
```

unless removing support for an obsolete public interface is itself the intended change.

---

## What Counts as Dead Code

Code may be considered dead when it is demonstrably:

- unreachable;
- uncalled;
- unexported and unused internally;
- superseded by another implementation;
- part of a retired migration path;
- guarded by conditions that can no longer occur;
- tied to unsupported versions;
- associated with removed functionality;
- duplicated but no longer referenced.

Do not equate "I cannot find a call immediately" with "dead".

---

## Required Removal Statement

Before deleting, write:

```text
Candidate for removal:
<function/file/branch/export/dependency>

Why it appears dead:
<evidence>

Supported behavior affected:
None.
```

or, if intentionally removing support:

```text
Supported behavior intentionally removed:
<explicit contract being retired>
```

Also state:

```text
Evidence checked:
- <call sites>
- <exports>
- <tests>
- <documentation>
- <downstream use if relevant>

Non-goals:
- <adjacent cleanup>
- <refactor>
- <renaming>
```

---

## Procedure

### Step 1 — Identify One Coherent Removal Target

Choose a narrow unit.

Examples:

```text
one unused helper
```

```text
one obsolete compatibility branch
```

```text
one cluster of tree-validation helpers with no callers
```

```text
one dependency after its final use disappeared
```

Avoid:

```text
clean all utilities
```

or:

```text
remove old code across package
```

unless the entire group is clearly one obsolete subsystem.

---

### Step 2 — Search for Direct Callers

Check for references in:

- package source;
- tests;
- examples;
- vignettes;
- documentation;
- scripts;
- compiled code;
- configuration;
- generated files where relevant.

For functions, search:

- exact name;
- namespaced calls;
- indirect registration;
- string-based lookup where plausible.

Do not rely on one search pattern if dynamic use is possible.

---

### Step 3 — Check Public Exposure

Determine whether the candidate is:

- exported;
- documented;
- referenced in examples;
- part of a formal generic;
- registered as a method;
- referenced by external configuration;
- part of a user-facing file format or serialized object.

An uncalled exported function is not automatically dead.

It may still be public API.

---

### Step 4 — Check Indirect Use

Look for non-obvious use such as:

- S3 method registration;
- S4 method dispatch;
- callbacks;
- hooks;
- reflection;
- dynamic lookup;
- namespace registration;
- test fixtures;
- plugin registration;
- compiled symbol registration.

Code that appears unreferenced by ordinary text search may still be active through dispatch or registration.

---

### Step 5 — Check Historical or Compatibility Role

Some code exists only for older versions, migrations, or compatibility.

Determine:

- which versions it supports;
- whether those versions remain supported;
- whether users can still encounter the path;
- whether serialized objects depend on it;
- whether deprecation policy requires keeping it.

Do not remove compatibility logic merely because current tests do not exercise it.

---

### Step 6 — Establish Behavioral Coverage

Before removal, identify the behavior around the candidate.

If code is truly unused, deleting it should not require behavioral test changes.

If the code belongs to a previously supported path, characterize that path before deciding whether removal is intentional.

For internal dead helpers, existing package tests may be sufficient.

---

### Step 7 — Delete Only the Proven Unit

Remove the candidate and directly associated artifacts.

Possible associated removals include:

- its documentation;
- its tests if those tests exist only for the dead private code;
- its export if obsolete;
- its import if no longer needed;
- a dependency if this was the final caller.

Do not remove unrelated nearby helpers.

---

### Step 8 — Run Relevant Checks

Run:

- package tests;
- package checks;
- relevant examples;
- targeted tests for surrounding behavior.

If deletion causes failures, determine whether:

1. the code was not dead;
2. a stale reference remains;
3. a generated artifact needs updating;
4. an unsupported path was implicitly relied upon.

Do not patch around failures blindly.

---

### Step 9 — Inspect the Diff

Ask of each deletion:

```text
What evidence proves this is safe to remove?
```

Ask of each non-deletion edit:

```text
Why is this required by removing the dead code?
```

Remove opportunistic cleanup.

---

## Evidence Hierarchy

Evidence of deadness may include:

### Strong Evidence

- no callers in source, tests, docs, examples, or registration;
- unexported internal helper;
- static analysis confirms no reachable path;
- feature that used it was already removed;
- compatibility version is outside supported range;
- replacement implementation is already the sole active path.

### Moderate Evidence

- no text references found;
- no tests exercise it;
- maintainers believe it is obsolete.

Moderate evidence may be insufficient for public or dynamically dispatched code.

### Weak Evidence

- function looks old;
- name suggests legacy;
- no one remembers using it;
- file is large;
- code appears redundant.

Weak evidence is not enough.

---

## Internal Helper Removal

An internal helper is a good removal candidate when:

```text
not exported
not registered
no callers
no reflective lookup
no tests depend on it
```

A dedicated PR may simply delete it.

Do not replace it with a new helper unless another caller needs the same behavior.

---

## Dead Branch Removal

For a branch such as:

```r
if (old_condition) {
    ...
} else {
    ...
}
```

prove that `old_condition` cannot occur under supported inputs or versions.

Then remove:

- condition;
- dead branch;
- obsolete imports or comments tied only to it.

Preserve the active branch behavior.

---

## Compatibility-Code Removal

Compatibility code often requires a stronger contract.

Document:

```text
Supported versions before:
<range>

Supported versions after:
<range>

Reason old compatibility path is no longer needed:
<policy or dependency requirement>
```

Removing compatibility support is not behavior-preserving for unsupported environments, but may still be correct package maintenance.

Keep that semantic change explicit.

---

## Export Removal

Removing an exported function is API change, not ordinary dead-code cleanup.

Use a deprecation/evolution skill unless:

- the export was accidental and never released;
- package policy explicitly allows removal;
- the package is pre-release and no compatibility contract exists.

Do not classify public API as dead merely because internal code does not call it.

---

## Test Removal

Delete tests when:

- they test only removed private code;
- the behavior is no longer supported;
- equivalent behavioral coverage exists at a better boundary.

Do not delete failing tests merely to enable code removal.

A failing test may prove the code is not actually dead.

---

## Documentation Removal

Remove documentation tied exclusively to the removed code.

Do not rewrite unrelated docs during the same PR.

If deleting a public feature, update documentation according to the package's deprecation and release policy.

---

## Dependency Removal After Dead Code

A dependency may become unused after code deletion.

Prefer two possible patterns:

### Same PR

If the dependency existed solely for the removed code and its removal is a direct consequence:

```text
remove obsolete feature
    ↓
remove now-unused dependency
```

This can remain one semantic change.

### Follow-Up PR

If dependency usage is broader or uncertain:

```text
PR 1: remove dead code
PR 2: audit and remove dependency
```

Choose the clearer review boundary.

---

## Dead Code Versus Duplication

Duplicated code is not dead merely because another implementation exists.

Before deletion, verify:

- the duplicate has no caller;
- callers have already migrated;
- behavior is equivalent where required.

If callers still use both versions, first migrate them in a separate refactor or behavior-preserving change.

---

## Dead Code Versus Deprecated Code

Deprecated code is intentionally still alive.

Do not remove it until the deprecation policy allows removal.

Check:

- release timeline;
- documented migration path;
- compatibility guarantees.

Deprecation warnings are evidence of planned removal, not immediate deadness.

---

## Dead Code Versus Feature Flags

A disabled feature may still be supported.

Before deleting flagged code, determine whether:

- flag can still be enabled;
- configuration is public;
- external builds use it;
- test environments cover it.

Do not confuse default-off with dead.

---

## Dynamic Languages and Reflection

In dynamic systems, extra caution is required.

Check for:

- `get()`;
- `do.call()`;
- name-based dispatch;
- formula evaluation;
- environment lookup;
- plugin registries;
- method registration.

Static text search alone may miss real use.

---

## Package-Level Cleanup Strategy

For a large legacy utility file, do not attempt one giant cleanup.

Prefer:

```text
PR 1:
remove proven-unused validation helpers

PR 2:
remove obsolete tree helpers

PR 3:
remove final unused dependency

PR 4:
refactor remaining utilities
```

Each PR should have one semantic purpose and independent evidence.

---

## Dead-Code PR Template

```text
## Candidate

<what is being removed>

## Evidence of non-use

- <search/call evidence>
- <registration/export evidence>
- <compatibility evidence>

## Intended behavioral change

None.

## Associated removals

- <docs/tests/imports directly tied to candidate>

## Behavior preserved

- <surrounding public behavior>

## Non-goals

- <other cleanup>
- <refactor>
- <API redesign>
```

---

## Examples

### Example 1 — Unused Internal Helper

Evidence:

```text
not exported
no source caller
no test caller
no method registration
no documentation reference
```

Action:

```text
delete helper
```

Non-goals:

```text
do not reorganize utility file
do not rename neighboring helpers
```

---

### Example 2 — Obsolete Compatibility Branch

Evidence:

```text
branch supports dependency < 2.0
package now requires dependency >= 3.0
```

Action:

```text
remove obsolete branch
```

Also remove imports used only by that branch if clearly direct consequences.

---

### Example 3 — Large Utility File

Observation:

```text
utils.R has many helpers, some apparently unused
```

Wrong approach:

```text
rewrite and clean utils.R
```

Preferred approach:

```text
identify one coherent group with no callers
prove non-use
delete that group
```

Then repeat later if justified.

---

### Example 4 — Public Function With No Internal Callers

Observation:

```text
exported function has no package-internal callers
```

Conclusion:

```text
not dead
```

It may exist for users.

Use API-evolution/deprecation process if removal is desired.

---

## Review Checklist

### Evidence

- [ ] Direct callers were searched.
- [ ] Tests and documentation were checked.
- [ ] Export and registration status were checked.
- [ ] Dynamic/indirect use was considered.
- [ ] Compatibility role was considered.

### Scope

- [ ] One coherent dead-code unit is removed.
- [ ] Unrelated cleanup is absent.
- [ ] Neighboring code is not rewritten unnecessarily.

### Behavior

- [ ] Supported behavior remains unchanged.
- [ ] Public API is not removed accidentally.
- [ ] Deprecation policy is respected.
- [ ] Relevant tests still pass.

### Consequences

- [ ] Associated docs/imports are removed only when directly tied.
- [ ] Dependency removal is justified.
- [ ] Generated artifacts are updated only as required.

---

## Completion Criteria

This skill is satisfied when:

- the removal target is explicit;
- evidence demonstrates non-use or obsolescence;
- public exposure and dynamic registration are checked;
- compatibility obligations are considered;
- one coherent unit is removed;
- supported behavior remains unchanged unless explicit removal is intended;
- tests and checks pass;
- unrelated cleanup is excluded;
- the codebase becomes smaller without gaining new concepts.

---

## Final Rule

> Delete with evidence, not intuition.

Remove what the package no longer needs.

Leave everything else for another change.
