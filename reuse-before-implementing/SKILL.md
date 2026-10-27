---
name: reuse-before-implementing
description: "Use before adding new code, new abstractions, new dependencies, new classes, new helpers, or new infrastructure: determine whether the behavior already exists in the language, standard library, package's public API, existing dependencies, or ecosystem; prefer delegation over reimplementation."
---

# Reuse Before Implementing

## Purpose

Use this skill before adding new code, new abstractions, new dependencies, new classes, new helpers, or new infrastructure.

Its purpose is to ensure that existing software, representations, and public interfaces are considered before implementing package-local machinery.

This skill applies to:

- new features;
- bug fixes;
- refactoring;
- algorithm integration;
- data transformation;
- visualization support;
- interoperability work;
- package migrations;
- compatibility updates;
- dependency decisions;
- API design.

The governing principle is:

> Reuse an existing, suitable solution before creating a new one.

Reuse is not an absolute rule.

Existing software should be reused when it adequately solves the problem and does not introduce disproportionate cost, instability, or conceptual complexity.

---

## Core Principle

Before implementing something new, investigate whether it already exists in:

1. the language or standard library;
2. the current package;
3. the package's dependencies;
4. the relevant domain ecosystem;
5. an established upstream implementation;
6. a standard representation or generic interface.

Prefer composition over duplication.

Prefer adaptation over reimplementation.

Prefer established contracts over package-specific ones.

---

## What Counts as Reuse

Reuse includes more than calling an existing function.

It may mean reusing:

- a function;
- a class;
- a generic;
- a data container;
- a result object;
- an algorithm implementation;
- a parser;
- an accessor;
- a metadata convention;
- a plotting system;
- a validation mechanism;
- a serialization format;
- an established workflow;
- a package already in the dependency graph.

A package can remain simple even while supporting sophisticated behavior if domain complexity is delegated to established software.

---

## Reuse Hierarchy

Investigate reuse in this order unless the task provides a reason to do otherwise.

### 1. Language and Standard Library

Ask first whether the programming language already provides the needed capability.

Examples include:

- matrix operations;
- list manipulation;
- file handling;
- string operations;
- mathematical functions;
- object inspection;
- serialization;
- iteration;
- error handling.

Do not add a dependency for a trivial operation already expressed clearly by the language.

Do not build a helper around a single standard operation merely to make it appear package-specific.

---

### 2. Existing Package Code

Search the current package for:

- public functions;
- internal helpers;
- existing validation;
- existing accessors;
- existing transformations;
- existing result handling;
- existing tests demonstrating related behavior.

Prefer using an existing coherent implementation over creating a parallel path.

However, do not force reuse of an overly broad helper when a small local operation would be clearer.

Reuse should reduce conceptual duplication, not merely line duplication.

---

### 3. Existing Dependencies

Inspect dependencies already used by the package.

Ask:

```text
Does one of our current dependencies already implement this correctly?
```

If yes, prefer delegating to it.

This is especially important for:

- statistical methods;
- numerical algorithms;
- parsers;
- optimization;
- specialized transformations;
- domain-specific calculations.

An established implementation usually carries more testing, validation, and domain expertise than a new local implementation.

---

### 4. Domain Ecosystem

Investigate the surrounding software ecosystem.

For scientific packages, this may include:

- standard containers;
- established generics;
- domain-specific packages;
- common result classes;
- shared conventions;
- interoperability standards.

The goal is not merely to avoid writing code.

The goal is to keep the package compatible with tools users already understand.

---

### 5. Upstream Package

If functionality conceptually belongs elsewhere, consider whether the correct solution is:

- use an upstream public API;
- request or contribute an upstream API;
- move functionality upstream;
- depend on an existing implementation.

Do not reproduce substantial upstream behavior locally solely to avoid coordination.

---

## Required Reuse Check

Before implementation, document the result of the following check.

```text
Language:
<existing capability or "none suitable">

Current package:
<existing capability or "none suitable">

Existing dependencies:
<existing capability or "none suitable">

Domain ecosystem:
<existing capability or "none suitable">

Upstream implementation:
<existing capability or "none suitable">

Existing work:
<open PR, branch, issue, or sibling package covering it, or "none found">

Decision:
<reuse / adapt / implement locally / add dependency / propose upstream change>
```

This need not be long.

It must be deliberate.

---

## Check Existing Work First

Reuse also applies to work that is not yet released.

Before starting, search:

- open and recently closed PRs in the package;
- remote branches;
- related issues and the PRs linked to them;
- sibling packages in the same ecosystem.

If the behavior already exists or is in progress elsewhere, build on it or point to it.

Do not start a parallel implementation.

---

## Reuse Decision Procedure

### Step 1 — Define the Required Behavior

Describe the problem without naming an implementation.

Good:

```text
We need to compute constrained ordination for two aligned assays.
```

Poor:

```text
We need to write our own RDA engine.
```

The first leaves room for reuse.

The second assumes the implementation before investigation.

---

### Step 2 — Separate Domain Logic From Integration Logic

Ask which part of the task is truly package-specific.

For example:

```text
package-specific:
- select assays;
- align samples;
- extract matrices;
- preserve names;
- validate container structure.

not package-specific:
- fit the statistical model.
```

The package should usually implement the first group and delegate the second.

---

### Step 3 — Search Existing Capabilities

Search in the reuse hierarchy.

For each candidate, determine:

- whether it solves the required behavior;
- whether it is public and documented;
- whether it is stable enough to depend on;
- whether its inputs and outputs are compatible;
- whether adaptation is smaller than reimplementation.

---

### Step 4 — Compare Conceptual Cost

Do not select reuse solely because code already exists.

Compare the conceptual cost of:

```text
A. reuse existing implementation
B. implement locally
```

Consider:

- new concepts introduced;
- dependency burden;
- maintenance surface;
- API stability;
- testing burden;
- performance;
- interoperability;
- numerical reliability;
- user familiarity.

Choose the option that keeps the system easier to reason about over time.

---

### Step 5 — Reuse the Narrowest Sufficient Interface

When reusing another package, prefer its smallest stable public interface.

Prefer:

```r
upstream::method(x, ...)
```

over:

```r
upstream:::internal_prepare(...)
upstream:::internal_fit(...)
upstream:::internal_finalize(...)
```

Prefer documented accessors over inspecting object internals.

Prefer public generics over manipulating slots directly.

---

### Step 6 — Add Only Necessary Glue

The local package should generally contain only the integration logic that belongs to its domain.

A thin adapter may need to:

```text
validate input
select data
align observations
convert representation
call upstream implementation
restore names or metadata
return result
```

This is appropriate local code.

Do not reproduce the upstream algorithm.

---

## Public API Rule

Reuse another package through its public contract.

Avoid:

- `:::` access;
- undocumented slots;
- hidden fields;
- internal helper names;
- assumptions about private object structure.

Private APIs can change without notice.

Depending on them transfers another package's implementation instability into your package.

---

## If Only a Private API Exists

When the needed functionality is available only through a non-exported interface, evaluate these options in order.

### Option 1 — Use a Different Public Interface

A broader public function may already provide the required behavior.

Use it if the adaptation remains clear.

---

### Option 2 — Implement the Minimal Operation Locally

If the hidden function performs a small, stable operation, local explicit code may be safer.

For example:

```text
private helper:
adds a pseudocount to a matrix
```

It may be better to write that small operation locally than depend on:

```r
dependency:::apply_pseudocount()
```

The local implementation should cover only the required behavior.

Do not copy an entire hidden framework.

---

### Option 3 — Request or Contribute an Upstream API

If the functionality is substantial and broadly useful, the appropriate solution may be upstream.

This is especially suitable when multiple packages need the same capability.

---

### Option 4 — Introduce Another Dependency

Use another package if it exposes a stable public implementation and the dependency cost is justified.

Treat this as a separate dependency decision.

---

## Reuse of Representations

Before introducing a new object representation, check whether existing structures already express the needed information.

Prefer, where appropriate:

```text
vector
matrix
data.frame
list
existing domain container
existing result class
existing metadata field
existing reduced-dimension storage
existing experiment slot
```

Ask:

> What problem does a new class solve that the existing representation cannot?

If the answer is speculative, do not introduce the class.

---

## Reuse of Result Classes

When wrapping an established method, return the established result class unless the package has a concrete reason not to.

Benefits include:

- compatibility with existing methods;
- existing print methods;
- existing summary methods;
- existing plotting tools;
- familiarity for users;
- reduced maintenance.

Avoid:

```text
upstream result
    ↓
package-specific wrapper class
```

unless the wrapper adds behavior that users currently need.

---

## Reuse of Generics

Before creating a new generic, determine whether a relevant generic already exists.

Examples:

- `show`;
- `plot`;
- `predict`;
- `coef`;
- `summary`;
- established ecosystem generics;
- Bioconductor accessors.

Do not create a package-specific generic solely to attach the package name to an established concept.

New generics increase conceptual and dispatch surface.

---

## Reuse of Existing Workflows

Do not create workflow machinery when normal composition is sufficient.

Prefer:

```r
x <- preprocess(x)
fit <- analyse(x)
plot(fit)
```

over:

```r
workflow <- PackageWorkflow$new(...)
workflow$add_step(...)
workflow$run(...)
```

unless users have a demonstrated need for workflow orchestration.

---

## Reuse of Visualization

Before adding package-specific plotting functions, ask:

```text
Can the existing result already be plotted by:
- base graphics?
- ggplot-compatible tools?
- a domain plotting package?
- the upstream method's plotting API?
```

If yes, documentation may be enough.

Add a plotting adapter only when users otherwise need package-specific extraction or interpretation logic.

---

## Reuse of Validation

Validation deserves careful judgment.

Do not create a generalized validation framework solely because several functions validate inputs.

Prefer local explicit checks when they are short and function-specific.

Create or reuse shared validation only when:

- the same semantic rule appears repeatedly;
- the shared function has a clear contract;
- callers genuinely become simpler;
- error behavior should be consistent.

Avoid generic validators that accumulate unrelated checks.

---

## Reuse of Data Alignment

For multi-object scientific data, sample or feature alignment is often package-specific integration logic.

Before implementing custom alignment, investigate:

- container-provided alignment;
- standard accessors;
- ecosystem helpers;
- common naming conventions.

If the container already guarantees alignment, do not reimplement it.

If alignment is required locally, implement only the rule needed by the method.

Do not build a general alignment subsystem unless repeated requirements prove it necessary.

---

## Reuse Versus Dependency Cost

Existing functionality is not automatically worth a new dependency.

Before adding one, ask:

```text
How much behavior are we gaining?
How much dependency surface are we adding?
```

A new dependency may be justified for:

- complex statistics;
- specialized algorithms;
- difficult parsers;
- domain standards;
- high-quality established methods.

A new dependency is usually not justified for:

- one trivial string operation;
- a single convenience helper;
- a simple matrix transformation;
- minor syntax reduction.

---

## Reuse Versus Local Explicit Code

Sometimes a few local lines are simpler than reuse.

Example:

```text
Option A:
depend on a large package for one trivial operation

Option B:
use five explicit lines of base-language code
```

Option B may be preferable.

The objective is not maximum reuse.

The objective is minimum unnecessary conceptual and maintenance burden.

---

## Reuse Versus Abstraction

Do not create a reusable abstraction before repeated requirements exist.

Two similar code fragments do not automatically justify a helper.

Ask:

```text
Are these fragments conceptually the same operation,
or do they merely look similar?
```

Prefer duplicated explicit code over a misleading abstraction when the semantics differ.

Refactor after the shared concept becomes clear.

---

## Thin Adapter Pattern

A preferred package architecture is:

```text
standard container
        ↓
extract required representation
        ↓
minimal validation/alignment
        ↓
established implementation
        ↓
established result
```

Example:

```r
getJointMethod <- function(x, assay_a, assay_b, ...) {
    a <- extract_assay(x, assay_a)
    b <- extract_assay(x, assay_b)

    aligned <- align_samples(a, b)

    upstream::method(
        aligned$a,
        aligned$b,
        ...
    )
}
```

The exact implementation may differ.

The principle is that the package owns integration, not reinvented domain machinery.

---

## Anti-Patterns

### Package-Specific Wrapper Without Need

Avoid:

```text
vegan::rda()
    ↓
MultiomicsRDAResult
```

if users can already work directly with the established RDA result.

---

### Reimplemented Established Algorithm

Avoid implementing a local version of an algorithm already maintained by a mature dependency unless there is a concrete incompatibility.

---

### Hidden Dependency Coupling

Avoid:

```r
dependency:::private_function()
```

especially in public methods.

---

### Dependency for Trivial Syntax

Avoid adding a package merely to replace a short, clear language-native operation.

---

### Premature Helper Framework

Avoid creating:

```text
TransformationManager
AnalysisRegistry
ResultFactory
ValidationEngine
```

because future methods may eventually need them.

Wait for demonstrated repeated requirements.

---

### Local Copy of Upstream Internals

Avoid copying hundreds of lines from another package to avoid a dependency.

If the behavior belongs upstream, use the upstream implementation or coordinate an API.

---

## PR Planning Template

For a change involving new functionality, include:

```text
Required behavior:
<what needs to happen>

Existing solutions considered:
- Language: <candidate>
- Current package: <candidate>
- Existing dependency: <candidate>
- Ecosystem/upstream: <candidate>

Chosen approach:
<what will be reused>

Local code required:
<only the integration logic that remains>

Why new abstraction is unnecessary:
<brief explanation>

Why a new dependency is or is not needed:
<brief explanation>
```

---

## Review Questions

A reviewer should ask:

### Existing capability

- Was the language checked first?
- Was the current package searched?
- Were existing dependencies considered?
- Was the domain ecosystem considered?

### Interface stability

- Is reuse through a public API?
- Are undocumented internals avoided?
- Is the selected representation established and stable?

### Local implementation

- Does local code contain only necessary integration logic?
- Is domain complexity delegated appropriately?
- Is copied or duplicated upstream logic avoided?

### Abstraction

- Is every new class justified?
- Is every new generic justified?
- Is every new helper conceptually meaningful?
- Is every new dependency justified?

### Simplicity

- Does the chosen solution reduce long-term conceptual burden?
- Would a future maintainer understand why this code exists?
- Could the same capability be achieved with fewer new concepts?

---

## Examples

### Example 1 — Statistical Method

Requirement:

```text
Support constrained ordination on assays stored in a standard container.
```

Preferred approach:

```text
container handling
    ↓
extract matrices
    ↓
align samples
    ↓
vegan::rda()
    ↓
return `rda` object
```

Avoid implementing a local RDA algorithm or custom result class without demonstrated need.

---

### Example 2 — Transformation

Requirement:

```text
Apply a simple log transform with a pseudocount.
```

If this is equivalent to a clear local expression:

```r
log(x + pseudocount)
```

do not add a large dependency solely for that operation.

---

### Example 3 — Private Helper

Current code:

```r
otherpkg:::.apply_pseudocount(x)
```

Preferred investigation:

```text
1. Is there a public exported equivalent?
2. Can a public transformation function be used?
3. Is the operation small enough to express locally?
4. Does this behavior belong upstream?
```

Do not continue depending on the private helper merely because it already works.

---

### Example 4 — Plotting

Requirement:

```text
Users want to visualize the result.
```

If the returned upstream class already works with an established plotting package:

```text
document that workflow first.
```

Do not create a package-specific plotting class and API without a concrete gap.

---

### Example 5 — Container

Requirement:

```text
Store several assays with sample metadata.
```

If an established ecosystem container already supports this:

```text
reuse it.
```

Do not introduce:

```text
NewPackageDataset
NewPackageAssay
NewPackageMetadata
```

unless existing containers are demonstrably inadequate.

---

## When Implementing Locally Is Correct

Local implementation is appropriate when:

- the operation is small and explicit;
- existing options introduce disproportionate dependencies;
- public APIs do not expose the required behavior;
- local semantics differ materially from upstream behavior;
- performance requirements justify specialization;
- interoperability constraints require custom logic;
- licensing or distribution constraints prevent reuse;
- the package's core scientific contribution is the implementation itself.

When implementing locally, document why reuse was not suitable.

---

## Completion Criteria

This skill is satisfied when:

- required behavior is stated independently of implementation;
- existing capabilities have been investigated;
- reuse candidates have been evaluated;
- public interfaces are preferred over private internals;
- existing representations are preferred where adequate;
- dependencies are added only when justified;
- local code is limited to necessary package-specific behavior;
- no speculative abstraction has been introduced;
- the chosen approach reduces long-term conceptual and maintenance burden.

---

## Final Rule

> Before writing a new mechanism, prove that an existing one is insufficient.

Reuse first.

Adapt second.

Implement only what remains.
