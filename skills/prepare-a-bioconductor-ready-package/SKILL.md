---
name: prepare-a-bioconductor-ready-package
description: "Use when creating, maturing, or auditing an R package for Bioconductor submission or continued Bioconductor readiness: verify current official devel requirements, reuse Bioconductor classes and public APIs, audit package metadata and dependencies, documentation, vignettes, tests, data provenance, size/resource limits, versioning, and pass R CMD build/check plus BiocCheck without hiding unrelated package-design problems."
---

# Prepare a Bioconductor-Ready Package

## Purpose

Use this skill when developing or auditing an R package intended for:

- initial Bioconductor submission;
- re-submission;
- transfer into a Bioconductor-oriented ecosystem;
- substantial pre-submission maturation;
- continued Bioconductor readiness after acceptance.

Its purpose is to make the package conform to Bioconductor expectations without
turning "Bioconductor readiness" into a giant redesign PR.

The governing principle is:

> Make the package Bioconductor-native by reusing Bioconductor conventions,
> classes, generics, infrastructure, and review expectations rather than
> building parallel package-specific machinery.

---

# Source of Truth Rule

Bioconductor guidance changes over time.

Before auditing or modifying a package:

1. inspect the current official **Bioconductor Packages: Development,
   Maintenance, and Peer Review** documentation;
2. use the current Bioconductor **devel** branch and associated R version;
3. inspect the current `BiocCheck` behavior;
4. treat current official guidance and automated checks as authoritative if
   they differ from this skill.

Do not rely on remembered version numbers, thresholds, accepted dependencies,
or review rules.

This skill defines the workflow and philosophy.

Current official guidance defines the exact policy.

---

# Core Bioconductor Philosophy

A Bioconductor-ready package should favor:

```text
existing Bioconductor class
    over a package-specific container

existing generic/accessor
    over direct slot manipulation

existing package implementation
    over reimplementation

public dependency API
    over ::: internals

interoperable result
    over isolated wrapper

executable vignette
    over static documentation

reproducible data provenance
    over unexplained bundled data
```

Bioconductor readiness is not merely passing `BiocCheck`.

It is interoperability plus maintainability plus reproducibility.

---

# Do Not Make "Bioc Ready" One Giant PR

For an existing package, first audit and classify findings.

Use categories such as:

```text
A. submission blocker
B. correctness/interoperability issue
C. documentation/test gap
D. packaging/metadata issue
E. optional improvement
```

Then create small semantic PRs.

Examples:

```text
PR:
replace private mia API usage

PR:
add characterization tests for getLASSO()

PR:
add required biocViews metadata

PR:
add executable package vignette

PR:
remove oversized bundled example data
```

Do not combine them merely because all are required for eventual submission.

---

# Required Readiness Statement

Before making changes, record:

```text
Target:
<new Bioconductor software package / existing Bioconductor package / audit>

Current version:
<DESCRIPTION version>

Current Bioconductor environment:
<devel version and R version verified from official guidance>

Current blockers:
- <finding>
- <finding>

Existing Bioconductor infrastructure reused:
- <classes>
- <generics>
- <packages>

Validation target:
- R CMD build
- R CMD check
- BiocCheck
- package tests/vignettes
```

---

# Phase 1 — Verify Package Identity and Scope

## Package Name

Confirm that the proposed package name:

- matches the repository name where required by current guidance;
- is not already used on Bioconductor or CRAN;
- is not confusingly close to an existing package/function/class;
- is intended to remain stable.

Do this early.

Renaming after acceptance is expensive and disruptive.

---

## Package Scope

The package should have a clear scientific/software responsibility.

Ask:

```text
What does this package own?
What established packages does it compose?
What does it deliberately not own?
```

Avoid packages that duplicate an existing Bioconductor package with only minor
differences.

If functionality belongs in an existing package, consider contribution or
migration instead.

---

# Phase 2 — Audit Interoperability Before Packaging Details

This is the most important architectural phase.

## Reuse Existing Bioconductor Classes

For common biological data structures, search established classes before
creating new ones.

Examples may include:

- `SummarizedExperiment`;
- `SingleCellExperiment`;
- `MultiAssayExperiment`;
- `TreeSummarizedExperiment`;
- `GRanges`;
- `Rle`;
- `DNAStringSet`;
- other domain-standard containers.

Use the class that already preserves the required semantics.

---

## Reuse Existing Generics and Accessors

Before creating:

```text
getSamples()
getFeatures()
getAssay()
```

check whether the ecosystem already has:

```text
assay()
rowData()
colData()
reducedDim()
altExp()
metadata()
```

or another established generic.

Avoid parallel accessor vocabularies.

---

## Avoid Direct Slot Access

Prefer public accessors to `@` or direct slot manipulation.

If the ecosystem class does not expose the behavior you need, investigate the
public API or upstream contribution before relying on private representation.

---

## Reuse Existing Implementations

If a Bioconductor or CRAN package already provides the algorithm:

```text
container integration
    ↓
existing public method
    ↓
existing result
```

is generally preferable to reimplementation.

This is particularly important for:

- import/parsing;
- statistical methods;
- sequence operations;
- genomic ranges;
- annotation;
- standard transformations.

---

# Phase 3 — DESCRIPTION Audit

Audit every field deliberately.

## Package and Version

For a new Bioconductor submission, verify the **current official required
pre-release version** immediately before submission.

At the time this skill was authored, official guidance specifies:

```text
Version: 0.99.0
```

for a new package.

Do not hard-code that assumption forever; re-check official guidance.

For maintained packages, follow current Bioconductor release/devel versioning
rules and increment the appropriate component as required.

---

## Title and Description

Ensure:

- title is concise and informative;
- description explains the package's actual functionality;
- description is complete prose rather than marketing language;
- the package's distinction from related software is understandable.

---

## Authors and Maintainer

Verify:

- `Authors@R` is valid;
- maintainer is explicit;
- maintainer email is current and reachable;
- contributor roles are accurate.

---

## License

Confirm:

- package license is valid;
- bundled code/data can legally be redistributed;
- third-party code attribution is present where required;
- dependency licenses are compatible with intended distribution.

Do not treat publicly visible code as automatically redistributable.

---

## Dependencies

Classify each dependency correctly.

Current Bioconductor guidance distinguishes roles such as:

```text
Imports
    functions/classes used by package namespace

Depends
    essential functionality exposed to users; use sparingly

Suggests
    vignettes, examples, tests, conditional/optional functionality

Enhances
    optional performance/augmentation in appropriate cases
```

Verify current exact guidance before editing.

---

## Dependency Availability

Bioconductor packages should depend on packages available through supported
Bioconductor/CRAN repositories.

Do not rely on:

- GitHub-only runtime dependencies;
- unsupported `Remotes:` fields;
- arbitrary source URLs;
- private package repositories for required functionality.

If a required dependency is not eligible, resolve that architecture before
submission.

---

## Version Constraints

Do not add minimum versions casually.

Use them only when the package requires functionality introduced in that
version or current Bioconductor policy requires it.

---

## SystemRequirements

Declare required external software accurately.

Do not attempt to install system software from package R code, examples,
vignettes, or README execution.

If installation is non-trivial, follow current Bioconductor guidance for
`INSTALL` documentation.

---

## biocViews

Verify:

- `biocViews` exists;
- terms are current and case-sensitive;
- terms belong to the appropriate package type/trunk;
- sufficiently specific leaf terms are used.

At the time this skill was authored, current guidance requires at least two leaf
terms for ordinary package metadata.

Re-check the current hierarchy before submission.

---

## URLs and Bug Reports

Provide useful current repository/project URLs and an issue tracker where
appropriate.

---

# Phase 4 — NAMESPACE and Public API Audit

## Export Intentionally

Export user concepts, not all functions.

Avoid broad export patterns.

Internal helpers should remain internal.

---

## Imports

Prefer targeted imports where suitable.

When extending or heavily relying on a Bioconductor class, follow current
guidance for importing the relevant package/class infrastructure.

---

## Generic Ownership

Before defining a generic:

- search Bioconductor;
- search base R;
- search relevant ecosystem packages.

Reuse an established generic when semantics match.

Duplicate generics harm interoperability.

---

## Naming

Follow current Bioconductor naming guidance.

Avoid names that accidentally imply S3 dispatch or expose internals.

More importantly, names should align with established ecosystem concepts.

---

# Phase 5 — R Code Audit

Use `BiocCheck` and current code guidelines as feedback, not as an invitation to
rewrite everything.

Audit for:

- public/private API boundaries;
- direct slot access;
- `:::` usage;
- hidden global state;
- `<<-`;
- inappropriate `set.seed()` inside package internals;
- browser/debug statements;
- fragile iteration;
- unnecessary duplicated ecosystem functionality;
- excessive function complexity;
- undocumented assumptions.

Fix each substantive issue as its own semantic change where possible.

---

## Do Not Chase Style at the Expense of Semantics

Style findings may be valid.

Do not mix whole-package style reformats into behavioral PRs.

If a formatting pass is required for readiness, make it a dedicated mechanical
PR after functional work is stable.

---

# Phase 6 — Documentation Audit

## Package-Level Documentation

Provide a useful package overview discoverable through normal R help.

It should explain:

- purpose;
- primary data structures;
- main workflows;
- important functions.

---

## Exported Functions and Classes

Every exported user-facing object should have complete documentation.

For functions, document:

- purpose;
- arguments;
- return value;
- side effects;
- important assumptions;
- relevant upstream implementation.

For new classes, document structure and semantics thoroughly.

---

## Runnable Examples

Use runnable examples for documented functionality.

Do not hide ordinary examples inside `dontrun` or equivalent mechanisms merely
to pass checks.

If execution genuinely cannot occur in normal checks, document the reason and
follow current Bioconductor policy.

---

# Phase 7 — Vignette Readiness

A software package should have at least one executable vignette as required by
current Bioconductor guidance.

The vignette should be task-oriented.

Preferred structure:

```text
Introduction
    ↓
motivation and relationship to existing Bioconductor packages
    ↓
installation instructions in non-evaluated form
    ↓
standard input object
    ↓
real package workflow
    ↓
result interpretation
    ↓
session information
```

---

## Vignette Code Must Be Real

Do not create a mostly static vignette whose important code is disabled.

The vignette should demonstrate core package functionality through evaluated
code.

Keep computation small enough for the build system.

---

## Vignette Should Demonstrate Interoperability

Use standard Bioconductor objects and existing downstream tools.

Do not make the vignette a tour of package-specific abstractions.

A strong Bioconductor vignette should make the package feel like part of the
ecosystem.

---

## Vignette Builder

Use a supported vignette format and declare the required builder/system
requirements correctly.

Check current guidance before choosing Quarto or other tooling because system
requirements can affect build portability.

---

# Phase 8 — Unit Test Readiness

Tests should cover package-owned behavior.

At minimum, prioritize:

- public methods;
- adapters;
- alignment;
- names and dimensions;
- important validation;
- known regressions;
- return classes;
- migrations;
- optional dependency boundaries.

Do not duplicate mature upstream package test suites.

Use `test-at-the-behavioral-boundary`.

---

# Phase 9 — Data and Provenance Audit

Avoid bundling large data merely for convenience.

Prefer existing Bioconductor data infrastructure where appropriate.

For included data:

- document provenance;
- document structure;
- document transformation/preparation;
- make preparation reproducible where practical.

For `inst/extdata`, include the preparation/provenance script in the location
required or preferred by current Bioconductor guidance.

Do not include unexplained binary data.

---

## Large Data

If example or experiment data are substantial, consider:

- existing data packages;
- `ExperimentHub`;
- `AnnotationHub`;
- another Bioconductor-supported data mechanism.

Do not solve large-data problems by committing oversized files and hoping
submission checks allow them.

---

# Phase 10 — Repository Hygiene

Audit the source tree for:

- build products;
- check directories;
- rendered vignette outputs where not permitted;
- caches;
- editor files;
- temporary data;
- credentials;
- duplicate `.gitignore` files;
- filenames differing only by case;
- oversized files.

Keep source packages reproducible and portable.

---

# Phase 11 — Current Resource Limits

Before submission, read the current official limits.

At the time this skill was authored, official guidance includes constraints such
as:

```text
source package:
    less than 10 MB after R CMD build

individual software-package files:
    no more than 5 MB

R CMD check --no-build-vignettes:
    target under 10 minutes

vignettes/examples/tests:
    recommended to stay within 8 GB memory
```

These values can change.

Always verify them against the current official development guide rather than
treating this section as permanent policy.

---

# Phase 12 — Build and Check in the Correct Environment

Bioconductor submissions are evaluated against the current Bioconductor devel
environment associated with a specific R version.

Do not validate only against:

- an old release;
- system R with mixed package repositories;
- arbitrary CRAN latest packages;
- a developer workstation with undeclared dependencies.

Use the current devel environment.

---

# Required Validation Sequence

Run, using current official instructions:

```text
1. package unit tests
2. vignette build
3. R CMD build
4. R CMD check
5. BiocCheckGitClone
6. BiocCheck for new package when applicable
```

At the time this skill was authored, current guidance expects new packages to
pass `R CMD build/check` and `BiocCheck` without errors or warnings, and expects
remaining notes to be addressed or justified during review.

Always confirm the exact current expectations.

---

# Treat BiocCheck Findings Semantically

Do not mechanically rewrite code until every message disappears.

For each finding classify:

```text
A. actual correctness/interoperability problem
B. packaging/documentation requirement
C. style/complexity issue
D. justified exception
E. tool/environment artifact
```

Then address the finding at the smallest semantic scope.

If an exception is justified, document the reasoning.

---

# Do Not Hide Checks

Avoid "fixes" such as:

- disabling tests;
- wrapping executable examples to prevent execution;
- suppressing warnings globally;
- moving code out of checked paths;
- using undocumented exclusions;
- removing meaningful assertions;
- vendoring dependencies to bypass dependency rules.

The objective is a healthy package, not a quiet checker.

---

# Phase 13 — Reviewer Simulation

Before submission, review the package as someone unfamiliar with it.

Ask:

```text
Why should this package exist in Bioconductor?
What existing packages overlap with it?
Does it reuse ecosystem containers?
Can I run the main workflow from the vignette?
Are results interoperable?
Are dependencies justified?
Are data provenance and licensing clear?
Can I understand the public API quickly?
```

If the answer requires explaining hidden architecture, simplify before
submission.

---

# Phase 14 — Submission Readiness Report

Produce a concise audit:

```text
Environment:
<Bioconductor devel / R version>

Architecture:
PASS / blockers

DESCRIPTION:
PASS / blockers

NAMESPACE/API:
PASS / blockers

Documentation:
PASS / blockers

Vignette:
PASS / blockers

Tests:
PASS / blockers

Data/provenance:
PASS / blockers

Size/resources:
PASS / blockers

R CMD build:
PASS / output summary

R CMD check:
PASS / remaining NOTE justification

BiocCheck:
PASS / remaining NOTE justification

Submission blockers:
- <item>
- <item>

Follow-up improvements not required for submission:
- <item>
```

Do not conflate blockers with optional cleanup.

---

# Recommended Maturation Order for an Existing Package

A useful sequence is:

```text
1. fix correctness bugs
2. characterize public methods
3. remove private dependency APIs
4. align with standard Bioconductor containers/generics
5. migrate misplaced functionality without redesign
6. audit dependencies
7. establish documentation and executable workflow
8. audit data/provenance
9. resolve metadata/namespace packaging requirements
10. run build/check/BiocCheck and address remaining findings
```

This order prevents packaging work from masking architectural problems.

---

# Anti-Patterns

## "Pass BiocCheck at Any Cost"

Wrong objective.

The package should satisfy the reason behind the checks.

---

## New Bioconductor-Like Container

Avoid creating a package-specific container for data already represented by an
established Bioconductor class.

---

## Private API Coupling

Avoid depending on another package's `:::` internals.

---

## Static Vignette

Avoid a vignette that describes functionality but does not execute it.

---

## GitHub-Only Runtime Dependency

Avoid making submission depend on a package that Bioconductor cannot resolve
through supported repositories.

---

## Bundled Large Data Without Provenance

Avoid unexplained data files used only to make examples convenient.

---

## Giant Submission Cleanup PR

Avoid:

```text
rewrite utilities
+ add vignette
+ rename APIs
+ change result classes
+ fix tests
+ replace dependencies
+ reformat package
```

Split by semantic purpose.

---

# Review Checklist

## Architecture

- [ ] Package has a clear Bioconductor-relevant scope.
- [ ] Existing Bioconductor classes are reused.
- [ ] Existing generics/accessors are reused.
- [ ] Established implementations are reused.
- [ ] Private dependency APIs are avoided.

## Metadata

- [ ] Package name is valid and stable.
- [ ] Current submission version rule is satisfied.
- [ ] Maintainer information is current.
- [ ] License is valid.
- [ ] Dependencies are correctly classified.
- [ ] Runtime dependencies are available from supported repositories.
- [ ] `biocViews` is current and appropriate.

## API

- [ ] Exports are intentional.
- [ ] Imports are controlled.
- [ ] No duplicate generic is introduced unnecessarily.
- [ ] Public names align with ecosystem conventions.

## Documentation

- [ ] Package-level help exists.
- [ ] Exported functions/classes are documented.
- [ ] Examples are runnable.
- [ ] At least one executable vignette demonstrates core workflow.
- [ ] Vignette explains motivation and ecosystem relationship.
- [ ] Session information is included according to current guidance.

## Testing

- [ ] Public methods have behavioral coverage.
- [ ] Regression tests cover known bugs.
- [ ] Alignment/orientation are tested where relevant.
- [ ] Upstream algorithms are not redundantly retested.

## Data

- [ ] Included data have provenance.
- [ ] Preparation scripts exist where required.
- [ ] Large data use appropriate Bioconductor infrastructure where practical.

## Build

- [ ] Current Bioconductor devel environment is used.
- [ ] Current size/time/memory requirements were verified.
- [ ] `R CMD build` passes.
- [ ] `R CMD check` passes to current submission expectations.
- [ ] `BiocCheckGitClone()` passes to current expectations.
- [ ] `BiocCheck(..., new-package=TRUE)` is run when appropriate.
- [ ] Remaining notes are understood and justifiable.

## Scope

- [ ] Submission blockers are separated from optional improvements.
- [ ] Readiness work is split into reviewable semantic PRs.
- [ ] No checker-driven broad rewrite is hidden in one PR.

---

# Completion Criteria

A package is Bioconductor-ready under this skill when:

- current official Bioconductor guidance has been consulted;
- development/testing use the current devel environment;
- package scope is clear and non-duplicative;
- established Bioconductor representations and public APIs are reused;
- dependencies are eligible and correctly classified;
- package metadata satisfies current requirements;
- exports/imports are intentional;
- documentation and executable vignette cover the real workflow;
- behavioral tests cover package-owned functionality;
- data provenance is documented and reproducible where required;
- repository and package size/resource limits meet current guidance;
- `R CMD build`, `R CMD check`, and required `BiocCheck` modes meet current
  submission expectations;
- remaining exceptions are explicit and defensible;
- readiness was achieved through small semantic changes rather than a broad
  redesign.

---

# Current Official References

Before each audit, re-check these official sources because requirements evolve:

- https://contributions.bioconductor.org/
- https://contributions.bioconductor.org/general.html
- https://contributions.bioconductor.org/description.html
- https://contributions.bioconductor.org/namespace.html
- https://contributions.bioconductor.org/docs.html
- https://contributions.bioconductor.org/tests.html
- https://contributions.bioconductor.org/r-code.html
- https://contributions.bioconductor.org/build-check-bioccheck.html
- https://contributions.bioconductor.org/versionnum.html
- https://contributions.bioconductor.org/bioconductor-package-submissions.html

---

# Final Rule

> Bioconductor readiness is ecosystem readiness, not checker appeasement.

Reuse Bioconductor's structures.

Expose a small coherent API.

Document and test real workflows.

Then prove the package builds and checks cleanly in the current devel ecosystem.
