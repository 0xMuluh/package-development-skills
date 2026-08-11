---
name: core-development-philosophy
description: Use as the default engineering philosophy for developing, extending, updating, and maintaining software packages: smallest conceptual change, reuse before implementing, one semantic change per PR, preserve unrelated behavior, avoid speculative abstractions, prefer explicit behavior. Applies to new package development, feature work, bug fixing, refactoring, API evolution, dependency changes, migration, and documentation.
---

# Core Development Philosophy

## Purpose

Use this skill as the default engineering philosophy for developing, extending, updating, and maintaining software packages.

It is intended to guide:

- new package development;
- incremental feature development;
- bug fixing;
- maintenance of established packages;
- refactoring;
- API evolution;
- migration of functionality between packages;
- dependency changes;
- documentation changes;
- compatibility work.

This skill is not tied to any one repository or package.

Its purpose is to keep software conceptually small as it grows.

---

## Core Principle

> Solve the stated problem with the smallest conceptual change, preferably by composing existing software, existing representations, and existing public interfaces.

"Small" refers to **semantic scope**, not the number of changed lines.

A change may legitimately touch many files or contain a substantial amount of code when all of those edits are necessary to solve one well-defined problem.

A short diff is not automatically simple.

A large diff is not automatically complex.

The relevant question is:

> Does every changed line contribute to the same intended change?

---

## Primary Engineering Goal

Prefer software that gains capabilities faster than it gains concepts.

A mature package may contain many functions while still exposing a small conceptual vocabulary.

Prefer:

```text
existing container
    +
existing implementation
    +
small domain-specific adapter
    =
new capability
```

over:

```text
new container
    +
new framework
    +
new abstraction layer
    +
new result class
    +
new configuration system
    =
new capability
```

New abstractions are not prohibited.

They must be justified by a concrete problem that existing abstractions cannot solve adequately.

---

## Development Invariants

The following rules apply unless a specific task provides a compelling reason to deviate.

### 1. Reuse before implementing

Before introducing new code, determine whether the required behavior already exists in:

1. the language or standard library;
2. the package's existing public API;
3. a dependency already used by the package;
4. the surrounding software ecosystem;
5. an established domain package or standard.

Prefer delegation over reimplementation when the existing behavior fits the requirement.

Do not create package-local machinery merely to make an existing implementation look package-specific.

---

### 2. Prefer existing representations

Before creating a new class, object type, schema, wrapper, or container, ask whether the information can be represented adequately using an existing structure.

Prefer, where appropriate:

- vectors;
- matrices;
- data frames;
- lists;
- established domain containers;
- existing result classes;
- existing metadata mechanisms;
- existing generic interfaces.

A new representation should solve a demonstrated problem, not anticipate a hypothetical future one.

---

### 3. One intended change per PR

Every pull request should have one semantic purpose.

A useful test is:

> Can the intended change be described accurately in one sentence?

Examples:

- Fix numeric selection of alternative experiments.
- Add regression tests for `getJointRDA()`.
- Rename `getRDA()` to `getJointRDA()` without changing behavior.
- Stop depending on a non-exported function from another package.
- Port joint RPCA while preserving its existing API and behavior.

A PR may modify implementation, tests, documentation, exports, examples, and multiple files if those changes are required by the same purpose.

Do not combine independent improvements merely because they are nearby.

---

### 4. Preserve unrelated behavior

When solving one problem, avoid changing behavior outside that problem's boundary.

Do not opportunistically:

- rename unrelated variables or functions;
- reformat unrelated code;
- reorganize files;
- rewrite adjacent helpers;
- change error messages;
- alter defaults;
- modify return values;
- change dependencies;
- update unrelated documentation;
- modernize syntax;
- remove unrelated code.

These may be worthwhile changes.

They belong in separate PRs.

---

### 5. Prefer explicit behavior over hidden machinery

Code should make important behavior visible.

Prefer:

```text
select input
align observations
call established implementation
return established result
```

over behavior hidden behind multiple internal layers unless those layers are necessary.

Avoid introducing:

- managers;
- engines;
- registries;
- factories;
- plugin systems;
- internal DSLs;
- configuration frameworks;
- speculative extension points;

unless the package has a demonstrated requirement for them.

---

### 6. Use the programming language as the workflow language

Do not invent a workflow abstraction when ordinary function composition already expresses the analysis clearly.

Prefer:

```r
x <- step_one(x)
x <- step_two(x)
result <- analyse(x)
```

over introducing a pipeline language, execution engine, or configuration format without a concrete need.

---

### 7. Depend on public contracts

When using another package, prefer its exported, documented interfaces.

Avoid relying on:

- non-exported functions;
- undocumented object internals;
- private fields;
- implementation-specific side effects.

If a dependency does not expose the required operation publicly, choose deliberately among:

1. using another public interface;
2. implementing the minimal required behavior locally;
3. proposing an upstream API;
4. introducing a dependency that provides the needed public contract.

Do not quietly bind package correctness to another package's internals.

---

### 8. Let established implementations own domain complexity

If a well-established package already implements a statistical method, algorithm, parser, model, or transformation correctly, the local package should usually provide only the integration needed for its users.

Prefer:

```text
domain container
    ↓
small adapter
    ↓
upstream implementation
    ↓
upstream result
```

The package should not reimplement an algorithm merely to make the code appear self-contained.

---

### 9. New classes must be earned

Do not introduce a new class because a result may eventually need methods.

First consider:

1. the upstream result class;
2. an existing domain class;
3. a matrix or data frame;
4. a named list;
5. attributes or metadata attached through an existing mechanism.

Create a new class only when it solves a current, concrete problem that these representations do not solve adequately.

---

### 10. Dependencies are architectural decisions

Every dependency adds:

- installation cost;
- compatibility surface;
- maintenance work;
- potential failure modes;
- API coupling.

Before adding one, determine whether the required functionality already exists elsewhere in the dependency graph or can be expressed clearly with existing tools.

Do not remove or replace dependencies opportunistically in unrelated PRs.

Dependency changes deserve their own semantic scope.

---

### 11. Tests protect behavior, not implementation style

Tests should primarily establish externally meaningful behavior.

Prefer tests for:

- accepted inputs;
- outputs;
- dimensions;
- names;
- classes;
- alignment;
- important metadata;
- error conditions;
- argument forwarding;
- known regressions.

Avoid tests that unnecessarily freeze private helper structure or incidental implementation details.

A refactor should be possible without rewriting the entire test suite if public behavior is unchanged.

---

### 12. Characterize before changing mature behavior

When modifying existing functionality that lacks adequate tests:

1. establish what it currently does;
2. add focused characterization tests where appropriate;
3. make the intended change;
4. add or update the test that demonstrates the new behavior.

Do not redesign poorly understood code and tests simultaneously unless the task specifically requires it.

---

### 13. Migrate before redesigning

When moving functionality between packages or modules, separate migration from improvement.

Preferred sequence:

```text
1. establish existing behavior;
2. reproduce it at the destination;
3. verify parity;
4. update the source package;
5. redesign later, in separate changes.
```

Do not combine:

```text
move + rename + redesign + optimize + change result type
```

unless those changes are inseparable.

---

### 14. Delete only with evidence

Removing code is valuable when the code is demonstrably unnecessary.

Before deleting, establish that it is:

- unused;
- obsolete;
- replaced;
- unreachable;
- unsupported by the current public contract.

Delete one coherent group at a time.

Do not use dead-code removal as justification for a broad cleanup.

---

### 15. Documentation should demonstrate ordinary use

Documentation should make the package look as simple as it is intended to be.

Prefer examples organized as:

```text
scientific or user question
    ↓
small amount of setup
    ↓
one or a few package functions
    ↓
standard result
    ↓
interpretation
```

If routine use requires extensive package-specific machinery, treat that as possible API feedback rather than hiding it in helper-heavy examples.

---

## Required Pre-Change Questions

Before implementing a change, answer these questions.

### Problem

What specific problem is being solved?

Write it without describing the proposed implementation.

### Intended change

What observable behavior should change?

If behavior should not change, state that explicitly.

### Non-goals

What related improvements are deliberately excluded from this change?

Include tempting adjacent work.

### Reuse check

What existing implementation, representation, class, generic, dependency, or ecosystem tool could solve all or part of this problem?

### Behavioral boundary

What must remain unchanged?

### Evidence

What test, example, check, or reproducible case will demonstrate that the problem is solved?

---

## PR Scope Template

Use the following structure when planning a pull request.

```text
Problem:
<one concrete problem>

Intended change:
<one observable change>

Reuse:
<existing code, API, class, dependency, or implementation being reused>

Non-goals:
- <explicitly excluded change>
- <explicitly excluded change>

Behavior that must remain unchanged:
- <existing behavior>
- <existing behavior>

Evidence:
<test or reproducible example that proves the intended change>
```

---

## Review Heuristics

A reviewer should be able to answer "yes" to most of the following.

### Scope

- Is there one clear purpose?
- Does every changed line serve that purpose?
- Are unrelated cleanups absent?
- Are unrelated behavior changes absent?

### Simplicity

- Does the change reuse existing abstractions where practical?
- Does it avoid introducing speculative machinery?
- Is the new conceptual surface smaller than or proportional to the capability added?
- Could a contributor understand the change without learning a new framework?

### Compatibility

- Are existing public interfaces preserved unless changing them is the purpose?
- Are return types preserved unless changing them is the purpose?
- Are defaults preserved unless changing them is the purpose?
- Are external APIs used through public interfaces?

### Evidence

- Is the intended behavior demonstrated by a focused test or example?
- Does the test protect the user-visible contract rather than private structure?
- Is the regression case represented when fixing a bug?

---

## Signals That a PR Should Be Split

Split the work when the PR contains multiple independent verbs such as:

```text
fix + rename
move + redesign
add + refactor
deprecate + optimize
document + alter behavior
remove dependency + rewrite implementation
add feature + clean utilities
```

Other warning signs include:

- the PR description requires several unrelated bullet points;
- tests cover behavior unrelated to the stated problem;
- many changes are explained as "while here";
- reviewers need to understand unrelated subsystems;
- rollback of one part would not require rollback of another;
- one portion could be merged safely without the rest.

---

## Acceptable Complexity

Simplicity does not require artificially short code.

The following can still be a simple change:

- a regression test longer than the bug fix;
- a rename touching implementation, exports, documentation, tests, and examples;
- a compatibility fix fragmented across several methods;
- a new adapter requiring explicit input checks and sample alignment;
- a migration touching two packages in separate PRs;
- a longer implementation chosen because it avoids reliance on unstable private APIs.

Prefer explicit, local complexity over hidden architectural complexity when the former keeps the system easier to reason about.

---

## Unacceptable "Simplification"

Do not simplify code by:

- silently dropping supported behavior;
- weakening validation without evidence;
- changing numerical behavior unintentionally;
- replacing a stable public dependency with copied code without justification;
- merging distinct concepts into one ambiguous function;
- hiding complexity behind a broad helper with unclear semantics;
- introducing a generic abstraction solely to reduce duplicated lines;
- removing tests because they make refactoring inconvenient.

Reducing line count is not the objective.

Reducing unnecessary concepts and unintended coupling is.

---

## Decision Rule for New Abstractions

Introduce a new abstraction only when all of the following are true:

1. a concrete current requirement exists;
2. existing language, ecosystem, or package abstractions are inadequate;
3. the abstraction removes repeated conceptual complexity rather than merely repeated syntax;
4. at least one current caller benefits from it;
5. its public or internal contract can be stated clearly;
6. the resulting system is easier to reason about than the alternatives.

If these conditions are not met, prefer local explicit code.

---

## Completion Criteria

A change guided by this skill is complete when:

- the stated problem is solved;
- the intended behavioral change is demonstrated;
- unrelated behavior remains unchanged;
- existing suitable software or representations have been reused;
- no unnecessary abstraction has been introduced;
- the PR contains no opportunistic cleanup;
- documentation is updated if the intended public behavior requires it;
- the diff can be explained by one semantic purpose.

---

## Default Principle

When uncertain between a broader improvement and a narrower sufficient change, choose the narrower sufficient change.

When uncertain between inventing and reusing, investigate reuse first.

When uncertain whether two improvements belong together, separate them.

When uncertain whether an abstraction is needed, wait until a concrete requirement proves that it is.
