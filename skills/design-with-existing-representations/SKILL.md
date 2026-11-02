---
name: design-with-existing-representations
description: "Use when deciding how data, results, metadata, configuration, or state should be represented in a package: prefer vectors, matrices, data frames, lists, established domain containers, and existing result classes; a new class or representation must be earned by a concrete current problem."
---

# Design with Existing Representations

## Purpose

Use this skill when deciding how data, results, metadata, configuration, or state should be represented in a package.

Its purpose is to prevent unnecessary package-specific classes, schemas, wrappers, and containers when established representations already solve the problem adequately.

This skill applies to:

- new package development;
- feature development;
- result design;
- data import and export;
- multi-assay or multi-modal data;
- metadata storage;
- model outputs;
- transformations;
- plotting integration;
- serialization;
- API evolution;
- migrations from older object systems.

The governing principle is:

> Prefer the simplest established representation that preserves the information and behavior users actually need.

---

## Core Principle

Do not introduce a new representation merely because the package has a new concept.

Ask first whether the concept can be expressed clearly using:

- vectors;
- matrices;
- arrays;
- data frames;
- lists;
- named lists;
- existing domain containers;
- existing result classes;
- established metadata mechanisms;
- existing generic interfaces.

A new representation should solve a demonstrated current problem.

It should not exist only to make the package feel self-contained or extensible.

---

## Why Representation Choices Matter

Representations create long-term obligations.

A new class or schema usually requires:

- constructors;
- validation;
- accessors;
- documentation;
- printing;
- subsetting;
- serialization;
- compatibility handling;
- testing;
- migration support;
- downstream methods;
- deprecation policy.

Therefore:

> A representation is an architectural commitment, not a convenience wrapper.

The cost is often much larger than the initial class definition.

---

## Representation Preference Order

Consider options in roughly this order.

### 1. Primitive or Base-Language Structure

Use when sufficient:

- vector;
- factor;
- matrix;
- array;
- data frame;
- list;
- named list.

These are broadly understood and compose well with existing tooling.

---

### 2. Existing Ecosystem Container

Use an established domain object when it already represents:

- assays;
- samples;
- features;
- metadata;
- experiments;
- trees;
- ranges;
- reduced dimensions;
- model results;
- genomic or biological structure.

Prefer ecosystem interoperability over package-specific encapsulation.

---

### 3. Existing Upstream Result Class

If an established method returns a useful class, preserve it when practical.

This often provides:

- `print`;
- `summary`;
- `plot`;
- `predict`;
- `coef`;
- extractors;
- downstream compatibility.

Do not wrap it merely to attach the package name.

---

### 4. Existing Metadata Mechanism

If auxiliary information can live naturally in:

- attributes;
- metadata slots;
- row metadata;
- column metadata;
- assay names;
- reduced dimensions;
- named list elements;

prefer that over a new wrapper class.

---

### 5. New Representation

Create a new class, schema, or container only after existing options are shown to be inadequate.

---

## Required Representation Check

Before creating a new representation, document:

```text
Information to represent:
<what must be stored>

Operations required:
<what users or package code must do with it>

Existing representations considered:
- base structure: <candidate>
- ecosystem class: <candidate>
- upstream result: <candidate>
- metadata mechanism: <candidate>

Why they are insufficient:
<concrete limitation>

Proposed new representation:
<only if necessary>

New responsibilities introduced:
<constructors, accessors, methods, validation, etc.>
```

If the "why insufficient" section is speculative, do not create the new representation.

---

## Procedure

### Step 1 — Identify Information, Not Objects

Describe the information that needs representation before naming a class.

Good:

```text
We need:
- a feature-by-sample abundance matrix;
- sample metadata;
- feature taxonomy;
- optional phylogenetic tree.
```

Poor:

```text
We need a `MicrobiomeDataSet` class.
```

The first statement leaves representation choices open.

---

### Step 2 — Identify Required Operations

Ask what users and package functions must actually do.

Examples:

- subset samples;
- subset features;
- store several assays;
- attach sample annotations;
- attach feature annotations;
- store a tree;
- fit a model;
- retrieve coefficients;
- plot ordination;
- serialize the result.

Representation should be chosen based on operations, not aesthetics.

---

### Step 3 — Search Existing Structures

Check:

1. language-native structures;
2. current package structures;
3. ecosystem containers;
4. upstream result classes;
5. established metadata mechanisms.

Prefer structures users already know.

---

### Step 4 — Test Fit With Real Workflows

Try expressing a small real workflow using the candidate existing representation.

Ask:

```text
Can users:
- construct it;
- inspect it;
- subset it;
- pass it to existing tools;
- retrieve needed information;
without package-specific ceremony?
```

If yes, the representation is probably sufficient.

---

### Step 5 — Identify the Actual Gap

If an existing representation is insufficient, state the exact gap.

Examples of concrete gaps:

```text
The existing class cannot preserve required relationships during subsetting.

The existing result loses feature identifiers required for downstream methods.

The current metadata mechanism cannot distinguish two semantically different result types needed for dispatch.
```

Non-gaps:

```text
A custom class would look cleaner.

We may need more methods later.

It would be nice to have package-specific printing.

This feels more object-oriented.
```

---

### Step 6 — Add the Minimum New Structure

If a new representation is justified, introduce only the structure required for the demonstrated gap.

Do not immediately add:

- broad inheritance hierarchies;
- factories;
- registries;
- adapters for hypothetical future backends;
- large method families;
- convenience wrappers unrelated to the current need.

---

## Standard Scientific Data Pattern

For many scientific packages, a preferred pattern is:

```text
established container
├── numeric assays
├── row/feature metadata
├── column/sample metadata
├── optional structured components
└── derived representations
```

Use existing container mechanisms for derived data where appropriate.

For example:

```text
raw counts
relative abundance
CLR values
```

may coexist as named assays instead of requiring separate package-specific dataset classes.

---

## Results Pattern

When wrapping an established method, prefer:

```text
input container
    ↓
adapter
    ↓
upstream method
    ↓
upstream result class
```

Avoid:

```text
upstream result
    ↓
package-specific result wrapper
```

unless the wrapper solves a real current problem.

---

## When a Plain Matrix Is Enough

Use a matrix when the essential contract is:

- rectangular numeric data;
- stable row and column meaning;
- names;
- standard matrix operations.

Do not create a class solely to say:

```text
rows are features
columns are samples
```

if that convention is already clear from API context and names.

Add explicit validation or documentation instead when necessary.

---

## When a Data Frame Is Enough

Use a data frame when information is tabular and heterogeneous.

Examples:

- sample metadata;
- feature annotations;
- coefficients;
- test statistics;
- tidy result summaries.

Do not create one class per table-shaped result unless dispatch or invariants require it.

---

## When a Named List Is Enough

Use a named list when:

- components differ in type or shape;
- component names communicate meaning;
- no specialized subsetting semantics are required;
- no downstream generic dispatch depends on a class.

A clear named list is often better than a custom container with several one-line accessors.

---

## When Attributes Are Appropriate

Attributes may be suitable for small auxiliary metadata that:

- belongs tightly to the result;
- does not require complex subsetting semantics;
- is unlikely to be manipulated independently;
- does not justify a new container.

Use attributes conservatively.

Do not hide core user data in obscure attributes.

---

## Existing Domain Containers

If the ecosystem already has a standard container, prefer using it directly.

Benefits include:

- familiar accessors;
- stable subsetting semantics;
- compatibility with many packages;
- existing validation;
- reduced learning burden;
- less package code.

Do not duplicate a domain container simply to avoid importing or depending on its ecosystem.

---

## Reuse of Upstream Result Objects

Preserve upstream result classes when they are useful.

Examples may include:

- model-fit objects;
- ordination objects;
- clustering results;
- statistical test objects.

This allows users to reuse existing methods without package-specific translation.

Before wrapping an upstream result, ask:

```text
What concrete downstream operation fails without the wrapper?
```

If none, return the original result.

---

## New Classes Must Be Earned

A new class is justified when at least one concrete requirement cannot be met adequately with existing structures.

Examples:

### Invariant Preservation

The object must enforce relationships between components that ordinary lists cannot safely maintain.

### Specialized Subsetting

Subsetting must update several linked components together.

### Dispatch

Current downstream behavior genuinely requires method dispatch on a distinct semantic result type.

### Validation

Strong structural invariants are essential and repeatedly relied upon.

### Serialization Contract

The object requires a stable portable representation beyond ordinary structures.

### Interoperability Standard

The class itself is part of an accepted external standard.

Even then, prefer extending or composing an existing class when appropriate.

---

## Weak Reasons for a New Class

The following alone are usually insufficient:

- custom printing;
- aesthetic preference;
- reducing a few repeated field names;
- anticipated future methods;
- package branding;
- desire for encapsulation;
- wanting everything to "belong" to the package;
- hiding an upstream result;
- wrapping a list with no additional invariants.

---

## Result-Class Decision Ladder

Use this sequence:

```text
1. Can the upstream result be returned directly?
   → yes: return it.

2. Does an established ecosystem class fit?
   → yes: use it.

3. Is a matrix/data.frame/list sufficient?
   → yes: use it.

4. Can small auxiliary metadata use an existing metadata mechanism?
   → yes: use that.

5. Is distinct dispatch or invariant enforcement genuinely required?
   → only then consider a new class.
```

---

## Wrapper-Class Warning

Wrapper classes often begin as:

```text
class containing:
- upstream result
- some metadata
```

They then accumulate:

- accessors;
- converters;
- plotting methods;
- summaries;
- compatibility code.

Before adding a wrapper, determine whether the same metadata could be attached or returned alongside the established result more simply.

---

## Avoid Parallel Object Universes

A package should not require users to constantly convert between:

```text
ecosystem object
↔ package object
↔ upstream object
↔ plotting object
```

Prefer one standard object flowing through as much of the workflow as possible.

Conversion should exist only where representations genuinely differ.

---

## Representation and API Simplicity

The fewer object types users must understand, the simpler the package API can remain.

Prefer:

```r
result <- getJointRDA(x)
plot(result)
summary(result)
```

over:

```r
result <- JointRDAResult(getJointRDA(x))
result <- asMultiomicsResult(result)
plotJointRDA(result)
summaryJointRDA(result)
```

unless the extra layers provide necessary behavior.

---

## Representation and Migration

When migrating functionality between packages, preserve the existing result representation first.

Do not use migration as an opportunity to introduce a new class.

Preferred sequence:

```text
1. migrate behavior with same representation;
2. verify parity;
3. evaluate representation separately;
4. change only if a concrete need exists.
```

---

## Representation and Plotting

Do not create a new result class merely to support plotting.

First check whether:

- the upstream class already has plotting methods;
- an existing plotting package supports it;
- a small extractor is sufficient;
- a data frame conversion is enough.

A plotting requirement alone does not automatically justify a new analysis-result class.

---

## Representation and Serialization

If objects must be saved or exchanged, prefer established serialization methods for the chosen representation.

Do not invent a new file format unless:

- existing formats cannot preserve required information;
- interoperability needs justify it;
- long-term maintenance responsibility is accepted.

New file formats create enduring compatibility obligations.

---

## Representation and Validation

Do not over-validate standard structures merely because they are not custom classes.

Validation should protect real assumptions.

For example, if a method requires:

```text
rows = features
columns = samples
unique sample names
```

validate those assumptions directly.

A new class is not required solely to move those checks into a constructor.

---

## Representation and Naming

Use clear names to communicate semantics when a simple structure is otherwise adequate.

For example:

```r
list(
    coefficients = ...,
    selected_features = ...,
    lambda = ...
)
```

may be clearer than a new class whose fields require custom accessors.

Names are a lightweight semantic tool.

Use them before adding machinery.

---

## Representation and Metadata Placement

Store information where ecosystem users expect it.

Examples:

```text
feature metadata → row metadata
sample metadata → column metadata
alternative assays → assay container
reduced representations → reduced-dimension mechanism
global provenance → object metadata
```

Avoid package-specific side channels when standard locations exist.

---

## Minimal Conversion Principle

If conversion is necessary, make it:

- explicit;
- local;
- reversible when practical;
- loss-aware;
- tested.

Do not scatter conversions throughout many methods.

Prefer a small boundary adapter.

---

## Anti-Patterns

### Brand-New Dataset Class for Standard Data

Avoid creating a package-specific dataset class when a standard ecosystem container already supports the required components.

---

### Wrapper Around Every Upstream Result

Avoid:

```text
MethodAResult
MethodBResult
MethodCResult
```

when upstream classes already provide suitable behavior.

---

### Class for a Single Data Frame

Avoid a custom class whose only purpose is to contain one table with named columns.

---

### Metadata Hidden in Slots

Avoid new slots for information already supported by standard metadata mechanisms.

---

### Parallel Accessor Vocabulary

Avoid creating:

```text
getSamples()
getFeatures()
getAssayMatrix()
```

when established ecosystem accessors already exist and are suitable.

---

### Conversion as Routine Workflow

Avoid making normal users repeatedly call:

```text
asPackageX()
asPackageY()
toModelInput()
fromModelOutput()
```

when adapters can remain internal.

---

## PR Scope for Representation Changes

A representation change is usually significant and should be isolated.

A PR should not combine:

```text
new class
+
new feature
+
migration
+
plotting API
```

unless inseparable.

Prefer:

```text
PR 1:
establish feature using existing representation

PR 2:
demonstrate concrete limitation

PR 3:
introduce representation change if still justified
```

---

## Review Checklist

### Need

- [ ] The information and required operations are clearly stated.
- [ ] Existing structures were evaluated first.
- [ ] A new representation solves a concrete current problem.

### Simplicity

- [ ] The representation introduces no unnecessary concepts.
- [ ] A matrix/data frame/list was considered.
- [ ] An ecosystem container was considered.
- [ ] An upstream result class was considered.
- [ ] Existing metadata mechanisms were considered.

### Interoperability

- [ ] Users can continue using established tools where practical.
- [ ] Existing accessors and generics are reused.
- [ ] Conversion layers are minimized.

### Maintenance

- [ ] New constructors/accessors/methods are justified.
- [ ] Validation corresponds to real invariants.
- [ ] Serialization and subsetting obligations are understood.
- [ ] The class is not speculative.

---

## Examples

### Example 1 — Multi-Assay Data

Requirement:

```text
Store several assays measured on overlapping sample sets with metadata.
```

If an established multi-assay container already supports:

- several experiments;
- sample maps;
- metadata;
- subsetting;

reuse it.

Do not create:

```text
PackageMultiOmicsData
```

without a demonstrated missing capability.

---

### Example 2 — Ordination Result

Requirement:

```text
Return the result of an established ordination method.
```

If the upstream package returns an `rda` object with existing summary and plotting support:

```text
return the `rda` object
```

unless package-specific metadata or behavior cannot be represented otherwise.

---

### Example 3 — Coefficient Table

Requirement:

```text
Return selected features and coefficients.
```

A data frame such as:

```text
feature
coefficient
selected
```

may be sufficient.

Do not create `FeatureSelectionResult` unless distinct behavior or invariants require it.

---

### Example 4 — Derived Assay

Requirement:

```text
Store normalized values alongside raw counts.
```

If the existing container supports multiple named assays:

```text
add another assay
```

rather than creating a new normalized-data object.

---

### Example 5 — Small Auxiliary Metadata

Requirement:

```text
Record the transformation method used to create a result.
```

If the result/container has a standard metadata mechanism, use it.

Do not wrap the result solely to store one method name.

---

## When a New Representation Is Correct

A new representation may be the right choice when:

- existing containers cannot preserve essential relationships;
- downstream dispatch needs a distinct semantic type;
- the object has meaningful invariants that must be enforced centrally;
- the package defines a genuinely new domain concept;
- repeated workflows reveal that existing structures create persistent ambiguity or unsafe manipulation;
- interoperability requirements specify the representation.

When introducing one, document why established alternatives were insufficient.

---

## Completion Criteria

This skill is satisfied when:

- information and required operations are identified first;
- existing language and ecosystem representations are evaluated;
- upstream result classes are preserved when suitable;
- standard metadata mechanisms are reused;
- conversion layers are minimized;
- no new class is introduced without a demonstrated need;
- any new representation has explicit invariants and responsibilities;
- the chosen representation keeps the package interoperable and conceptually small.

---

## Final Rule

> Use a new representation only when an existing one fails a concrete requirement.

Represent information simply.

Let existing containers carry complexity when they already know how.
