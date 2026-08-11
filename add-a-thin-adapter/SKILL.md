---
name: add-a-thin-adapter
description: Use when adding package functionality that connects existing data structures to an established external implementation: build a thin domain-specific adapter, perform explicit input checks and sample alignment, delegate to the upstream implementation, and return established result types.
---

# Add a Thin Adapter

## Purpose

Use this skill when adding package functionality that connects existing data structures to an established external implementation.

Its purpose is to keep integration code small in conceptual scope while allowing the package to expose useful domain workflows.

This skill applies to:

- statistical methods;
- machine-learning methods;
- ordination;
- clustering;
- transformations;
- distance calculations;
- enrichment;
- visualization adapters;
- parsers;
- import/export bridges;
- domain-specific algorithms already implemented elsewhere.

The governing principle is:

> Let the package own integration. Let the established implementation own the algorithm.

A thin adapter should make an existing capability natural to use with the package's standard containers without creating a parallel framework.

---

## Core Pattern

Prefer:

```text
standard package/ecosystem input
        ↓
select relevant data
        ↓
validate method-specific requirements
        ↓
align or reshape only as necessary
        ↓
call established implementation
        ↓
preserve established result
```

Avoid:

```text
standard input
        ↓
convert into package-specific intermediate framework
        ↓
reimplement upstream preparation
        ↓
reimplement algorithm
        ↓
wrap upstream-style result in a package-specific class
```

The adapter should be a bridge, not a replacement ecosystem.

---

## What a Thin Adapter Owns

A thin adapter may legitimately own:

- selecting assays or components;
- extracting matrices or tables;
- aligning samples or features;
- validating package-specific assumptions;
- converting orientation;
- forwarding user arguments;
- restoring names;
- attaching small amounts of relevant metadata;
- selecting an appropriate upstream method;
- translating between established representations where necessary.

These are integration responsibilities.

---

## What a Thin Adapter Usually Does Not Own

A thin adapter should usually not own:

- the statistical algorithm;
- numerical optimization;
- model-fitting internals;
- general-purpose preprocessing already available elsewhere;
- a package-specific workflow engine;
- a new result class without demonstrated need;
- a generalized validation framework;
- a private copy of upstream implementation code.

---

## Required Adapter Statement

Before implementing an adapter, write:

```text
User problem:
<what users need to accomplish>

Standard input:
<existing container or representation>

Established implementation:
<package/function being reused>

Package-specific integration required:
- <selection>
- <alignment>
- <validation>
- <conversion>

Result:
<existing result representation where possible>

Non-goals:
- <algorithm reimplementation>
- <new framework>
- <unrelated preprocessing>
```

This forces the design to distinguish integration from scientific or algorithmic complexity.

---

## Procedure

### Step 1 — Identify the Upstream Contract

Inspect the established implementation's public interface.

Determine:

- required input type;
- required orientation;
- assumptions about names;
- handling of missing values;
- accepted arguments;
- return class;
- relevant downstream methods.

Use the public documented interface only.

Do not begin by reading or copying upstream internals unless needed to understand a documented behavior.

---

### Step 2 — Identify the Package Input Contract

Determine what your users already hold.

Examples:

- matrix;
- data frame;
- `SummarizedExperiment`;
- `SingleCellExperiment`;
- `MultiAssayExperiment`;
- another established domain container.

Do not introduce an intermediate package-specific input object unless existing structures cannot express the required information.

---

### Step 3 — Find the Smallest Translation Boundary

Determine the minimal transformation needed to satisfy the upstream function.

For example:

```text
MultiAssayExperiment
    ↓
select two experiments
    ↓
extract assays
    ↓
align samples
    ↓
matrix + matrix
    ↓
upstream::method()
```

Do not normalize, aggregate, transform, or filter unless the method specifically requires it.

Preprocessing choices should remain explicit and composable whenever possible.

---

### Step 4 — Validate Only Method Requirements

Validation should answer:

```text
Can this method run correctly on this input?
```

Examples:

- requested assay exists;
- required experiments exist;
- samples can be aligned;
- matrix is numeric;
- minimum number of observations is satisfied;
- response has valid type;
- no unsupported missing values are present.

Avoid broad unrelated validation.

Do not validate every possible property of the container.

---

### Step 5 — Align Explicitly When Needed

For multi-object methods, alignment is often the most important package-specific responsibility.

Make the alignment rule clear.

Possible rules include:

- intersection of sample identifiers;
- exact same sample set required;
- preserve first-input order;
- reorder all inputs to common identifiers.

Do not rely on accidental positional alignment.

Test names and order explicitly.

---

### Step 6 — Convert Orientation Only at the Boundary

If upstream expects:

```text
observations × variables
```

but the ecosystem stores:

```text
features × samples
```

transpose at the adapter boundary.

Do not change the package-wide storage convention.

Make the orientation change local and testable.

---

### Step 7 — Forward Upstream Arguments Transparently

If users need access to upstream options, prefer forwarding `...` or a small explicit subset.

Do not mirror every upstream argument in a new package-specific configuration object.

Use explicit arguments when:

- the adapter changes their semantics;
- validation is package-specific;
- defaults need to differ intentionally;
- discoverability materially benefits users.

Otherwise, preserve upstream flexibility.

---

### Step 8 — Call the Established Implementation Directly

Prefer one clear delegation point.

Example:

```r
result <- upstream::method(x, y, ...)
```

Avoid multiple internal wrapper layers around the same upstream call unless each layer has a distinct, reusable semantic purpose.

---

### Step 9 — Return the Established Result

Return the upstream result class where practical.

Benefits include:

- compatibility with upstream summaries;
- existing plotting methods;
- existing extractors;
- user familiarity;
- lower maintenance.

Only convert or wrap the result when a concrete package requirement demands it.

---

### Step 10 — Attach Only Necessary Metadata

If package-specific metadata is required, use existing metadata mechanisms where possible.

Examples:

- names;
- attributes;
- container metadata;
- standard slots;
- named list elements.

Do not create a result class just to record:

- source assay name;
- method label;
- call metadata;
- one extra table.

---

## Thin Adapter Shape

A healthy adapter often looks conceptually like:

```r
getJointMethod <- function(x, experiment_a, experiment_b, ...) {
    a <- extract_requested_data(x, experiment_a)
    b <- extract_requested_data(x, experiment_b)

    aligned <- align_common_samples(a, b)

    upstream::method(
        t(aligned$a),
        t(aligned$b),
        ...
    )
}
```

The actual code may be longer because validation, naming, or S4 dispatch is required.

That does not make it conceptually thick.

The question is whether the extra code serves integration rather than recreating the upstream method.

---

## Adapter Complexity Budget

A thin adapter may be mechanically long when it must handle:

- multiple established container classes;
- method dispatch;
- assay selection;
- alignment;
- informative errors;
- name restoration;
- documentation;
- tests.

This is acceptable if the conceptual flow remains:

```text
prepare → delegate → return
```

Do not judge thinness by line count.

Judge it by ownership of concepts.

---

## Adapter Versus Wrapper Framework

A warning sign is when adding one method requires introducing:

```text
MethodConfig
MethodRunner
MethodResult
MethodRegistry
MethodBackend
```

Before doing so, ask whether the function itself plus existing R dispatch already solves the problem.

Prefer ordinary functions until repeated concrete needs justify more structure.

---

## Adapter Versus Reimplementation

If the package starts duplicating formulas or algorithm steps from the upstream implementation, stop and reassess.

Ask:

```text
Why are we not calling the upstream function directly?
```

Possible valid reasons:

- upstream API lacks required functionality;
- performance constraints require specialization;
- package's scientific contribution is a new method;
- upstream behavior is incompatible with the required semantics.

If none applies, delegate.

---

## Adapter Versus Preprocessing

Do not hide substantial preprocessing inside an analysis adapter unless it is mathematically intrinsic to the method.

Prefer:

```r
x <- transformAssay(x, ...)
result <- getJointMethod(x, ...)
```

over:

```r
result <- getJointMethod(
    x,
    normalize = TRUE,
    pseudocount = 1,
    aggregate = "Genus",
    filter_prevalence = 0.1,
    ...
)
```

The adapter should not become a pipeline.

---

## Adapter Versus Convenience

Convenience is useful when it removes repeated domain-specific plumbing.

Good convenience:

```text
select requested assays from a standard container
align samples correctly
call upstream method
```

Weak convenience:

```text
rename an upstream function without changing inputs or outputs
```

A package-level adapter should add domain integration value.

---

## Multiple Input Containers

If the method supports several established containers, use existing dispatch mechanisms where appropriate.

Prefer:

```text
generic
├── method for container A
└── method for container B
```

when the ecosystems already use formal dispatch.

Avoid manually switching on class names inside one large function unless that is simpler and consistent with the language/package conventions.

Do not invent a new dispatch system.

---

## Alignment Contract

Every multi-input adapter should make its alignment rule explicit.

Document:

```text
- identifier used;
- whether unmatched observations are dropped or rejected;
- order after alignment;
- duplicate handling;
- empty-intersection behavior.
```

Tests should cover at least one non-trivial ordering case.

---

## Missing Data Contract

Do not silently alter missing-value behavior unless required.

Determine whether:

- upstream handles missing values;
- the adapter should reject them;
- rows should be removed;
- users should preprocess first.

Prefer explicit behavior over hidden cleaning.

---

## Argument Forwarding Contract

When using `...`, document what it is forwarded to.

Example:

```text
Additional arguments are passed to `vegan::rda()`.
```

Do not consume or reinterpret arbitrary arguments invisibly.

If the adapter needs its own arguments, keep their names distinct from upstream options where possible.

---

## Result Contract

Document the result in terms of the established representation.

Prefer:

```text
Returns the object produced by `upstream::method()`.
```

plus any package-specific additions.

Avoid pretending an upstream result is a package-specific invention.

---

## Testing a Thin Adapter

Tests should focus on integration responsibilities.

Test:

- correct data selection;
- correct alignment;
- correct orientation;
- correct argument forwarding where important;
- preservation of names;
- expected upstream result class;
- method-specific input errors.

Do not duplicate the upstream package's algorithm tests.

---

## Argument Forwarding Test

When forwarding matters, use an upstream argument with an observable effect.

Do not inspect call internals unnecessarily.

Example conceptual test:

```text
call adapter with upstream option A
verify result reflects option A
```

---

## Orientation Test

Use asymmetric dimensions or deliberately structured values so a mistaken transpose is detectable.

Avoid square matrices for orientation tests.

---

## Alignment Test

Construct two inputs with the same identifiers in different orders.

Verify that the adapter produces the same result as manually aligned inputs.

This is often more valuable than testing a large real dataset.

---

## Result-Class Test

If interoperability depends on the upstream result, assert its class.

Example:

```r
expect_s3_class(result, "rda")
```

This protects against unnecessary future wrapping.

---

## Upstream Dependency Boundary

The adapter should depend only on the public API required for the method.

Avoid:

- private helpers;
- undocumented object slots;
- copying internal preprocessing;
- relying on incidental attributes.

If upstream changes its private internals, a well-designed adapter should usually remain unaffected.

---

## Error Design

Adapter errors should focus on integration failures the package can explain clearly.

Examples:

- experiment not found;
- assay not found;
- no common samples;
- non-numeric assay;
- incompatible dimensions.

Let upstream handle errors belonging to its own method where reasonable.

Do not duplicate upstream validation unless needed to provide a materially clearer domain-specific error.

---

## Naming

Name the adapter for the domain operation, not the upstream package.

Prefer:

```text
getJointRDA()
```

over:

```text
runVeganRDA()
```

when the method itself is the user concept.

However, documentation should acknowledge the upstream implementation.

---

## Provenance

Where scientifically important, document or store enough information to identify:

- upstream method;
- relevant input assay names;
- important adapter-level choices.

Do not build a provenance framework unless repeated workflows require it.

---

## Thin Visualization Adapter

The same pattern applies to plotting.

Prefer:

```text
existing result
    ↓
small extractor or fortify step
    ↓
established plotting framework
```

Avoid creating a plotting object hierarchy.

If the upstream result already plots well, documentation may be sufficient and no adapter is needed.

---

## Thin Import Adapter

For import:

```text
external format
    ↓
established parser
    ↓
map parsed components
    ↓
standard ecosystem container
```

Do not write a parser when a trusted parser already exists.

The package-specific responsibility is the mapping into the ecosystem representation.

---

## Thin Export Adapter

For export:

```text
standard container
    ↓
extract required components
    ↓
established writer
```

Do not invent a new file format without demonstrated need.

---

## PR Scope

One adapter should normally be one semantic PR.

A good PR may include:

- generic/method definition;
- extraction code;
- alignment helper if truly needed;
- upstream dependency call;
- tests;
- documentation;
- export changes;
- example.

Do not combine:

```text
new adapter
+
general utility refactor
+
new result class
+
plotting framework
+
dependency cleanup
```

unless inseparable.

---

## Common Anti-Patterns

### Reimplemented Algorithm

```text
The upstream package already provides the method,
but the adapter reproduces the equations locally.
```

Avoid.

---

### Hidden Preprocessing Pipeline

```text
Adapter silently normalizes, filters, aggregates, imputes, and fits.
```

Avoid unless those steps are intrinsic and documented as part of the method.

---

### Result Wrapper Without Need

```text
upstream result → package-specific wrapper
```

Avoid unless the wrapper solves a current concrete problem.

---

### Generic Framework for One Method

```text
BackendRegistry + MethodEngine + ResultFactory
```

for a single adapter is usually unnecessary.

---

### Private Upstream Calls

```r
upstream:::internal_helper(...)
```

Avoid.

---

### Positional Alignment

Assuming two matrices are already in the same sample order is unsafe unless the container contract guarantees it.

---

## Review Checklist

### Design

- [ ] The adapter solves a clear user problem.
- [ ] An established implementation is reused.
- [ ] Standard input representations are preserved.
- [ ] The adapter contains integration logic rather than algorithm duplication.

### Input Handling

- [ ] Data selection is explicit.
- [ ] Alignment rules are explicit.
- [ ] Orientation conversion is local.
- [ ] Validation is method-specific and minimal.

### Upstream Boundary

- [ ] Only public upstream APIs are used.
- [ ] Upstream arguments are forwarded clearly.
- [ ] Upstream behavior is not unnecessarily duplicated.

### Result

- [ ] The established result representation is preserved where suitable.
- [ ] New classes are avoided unless justified.
- [ ] Metadata additions use existing mechanisms.

### Tests

- [ ] Selection is tested.
- [ ] Alignment is tested where relevant.
- [ ] Orientation is tested where relevant.
- [ ] Result class is tested.
- [ ] Upstream algorithms are not redundantly retested.

### Scope

- [ ] The PR contains one adapter or one coherent integration change.
- [ ] No unrelated cleanup is present.
- [ ] No speculative framework was introduced.

---

## Example 1 — Joint Ordination

User problem:

```text
Run constrained ordination between two assays stored in a multi-assay container.
```

Preferred design:

```text
MultiAssayExperiment
    ↓
select assay A and assay B
    ↓
align samples
    ↓
convert orientation
    ↓
vegan::rda()
    ↓
return `rda`
```

Package responsibilities:

```text
selection
alignment
orientation
errors
```

Upstream responsibilities:

```text
RDA mathematics
model fitting
ordination result structure
```

---

## Example 2 — Kernel Association

User problem:

```text
Run an established kernel association method using data from standard experiments.
```

Preferred design:

```text
container
    ↓
extract response/covariates/features
    ↓
align samples
    ↓
build only required inputs
    ↓
MiRKAT::...
    ↓
return established result
```

Do not create a package-specific kernel-testing engine.

---

## Example 3 — LASSO Integration

User problem:

```text
Fit LASSO using selected omics data.
```

Preferred design:

```text
container
    ↓
extract predictor matrix and response
    ↓
align
    ↓
perform only required method-specific preparation
    ↓
glmnet::...
    ↓
return standard result or small standard summary
```

Avoid copying `glmnet` fitting logic or relying on unrelated private helpers from another package.

---

## Example 4 — Plotting Existing Result

User problem:

```text
Plot an ordination result.
```

If an established plotting package already supports the returned class:

```text
document that route
```

No new adapter may be needed.

If package-specific extraction is required:

```text
existing result
    ↓
small extractor
    ↓
data frame
    ↓
ggplot
```

Keep the plotting bridge thin.

---

## When a Thin Adapter Is Not Enough

A larger implementation may be justified when:

- the package contributes a genuinely new algorithm;
- existing implementations do not support required semantics;
- performance requirements demand specialized code;
- upstream APIs are fundamentally incompatible;
- the package must maintain a stable cross-backend abstraction already required by multiple implementations.

Even then, introduce only the additional architecture proven necessary by current requirements.

Do not extrapolate from one complex method into a package-wide framework automatically.

---

## Completion Criteria

This skill is satisfied when:

- the user problem is explicit;
- an established implementation is identified and reused;
- standard input representations are retained;
- package-specific code is limited to necessary integration;
- alignment and orientation are explicit;
- validation is narrow and method-specific;
- only public upstream APIs are used;
- the established result is preserved where practical;
- tests cover the adapter boundary rather than duplicating upstream theory;
- no unnecessary framework, wrapper, or class has been introduced;
- the PR has one semantic purpose.

---

## Final Rule

> Prepare the input, call the established implementation, preserve the result.

The package should make existing expertise easier to use, not recreate it.
