---
name: document-a-real-workflow
description: Use when creating or revising package documentation that teaches users how to accomplish a real task: organize examples as a scientific or user question, small setup, a few package functions, a standard result, and interpretation; make the package look as simple as it is intended to be.
---

# Document a Real Workflow

## Purpose

Use this skill when creating or revising package documentation that teaches users how to accomplish a real task.

Its purpose is to make documentation reflect the package's intended software philosophy:

- use existing representations;
- compose existing tools;
- expose a small conceptual vocabulary;
- keep package-specific ceremony minimal;
- show one clear workflow at a time.

This skill applies to:

- vignettes;
- tutorials;
- README examples;
- package websites;
- function examples;
- migration guides;
- workflow articles;
- method demonstrations;
- integration examples.

The governing principle is:

> Documentation should show the simplest realistic path from a user question to a useful result.

Documentation is not only explanatory material.

It is also a test of API design.

If ordinary use requires extensive package-specific machinery, the documentation should reveal that rather than conceal it.

---

## Core Principle

Prefer documentation organized as:

```text
real question
    ↓
small amount of setup
    ↓
standard data representation
    ↓
one or a few package operations
    ↓
standard result
    ↓
interpretation
```

Avoid documentation organized primarily around:

```text
package architecture
internal abstractions
configuration layers
helper classes
framework concepts
```

unless users genuinely need those concepts to perform the task.

---

## Documentation as Design Feedback

A workflow that requires many steps may indicate:

- too much API ceremony;
- poor reuse of existing containers;
- unnecessary conversions;
- hidden preprocessing;
- overly broad functions;
- unclear names;
- package-specific result wrappers;
- missing interoperability.

Do not automatically solve documentation complexity with more helper code.

First ask:

> Is the underlying API more complicated than the user problem requires?

Documentation should expose friction honestly.

---

## Required Workflow Statement

Before writing, define:

```text
User question:
<what the user is trying to learn or accomplish>

Starting object:
<standard input/container>

Package contribution:
<what this package uniquely adds>

Existing tools reused:
<upstream methods, containers, plotting packages, etc.>

Final result:
<standard result or interpretable output>

Non-goals:
- <unrelated methods>
- <full package tour>
- <internal architecture explanation>
```

---

## Procedure

### Step 1 — Start From a Real User Question

Good:

```text
How do I test whether two omics layers share multivariate structure?
```

Good:

```text
How do I fit a penalized model using one assay to predict a phenotype?
```

Good:

```text
How do I migrate from the old function name to the new one?
```

Poor:

```text
This vignette demonstrates all package functions.
```

Poor:

```text
This article explains the internal object model.
```

A focused question produces focused documentation.

---

### Step 2 — Use the Standard Starting Representation

Begin with the object users are expected to already have.

Prefer:

- established ecosystem container;
- matrix;
- data frame;
- standard upstream result.

Do not begin by coercing into a package-specific wrapper unless that wrapper is truly necessary.

If import or preprocessing belongs to another package, say so and reuse it.

---

### Step 3 — Keep Setup Minimal but Realistic

Use enough setup to make the example scientifically or operationally meaningful.

Avoid:

- dozens of unrelated preprocessing steps;
- large data-loading sections;
- extensive object construction unrelated to the method;
- custom helper functions created solely for the vignette.

When setup is inherently substantial, separate it clearly from the package-specific step.

---

### Step 4 — Make Package Responsibility Visible

The user should be able to see what the package contributes.

For example:

```r
result <- getJointRDA(x, ...)
```

should be visually distinct from:

```r
x <- preprocess_with_existing_tool(x)
```

This helps users understand package scope.

---

### Step 5 — Reuse Existing Workflow Components

If another package already provides:

- preprocessing;
- normalization;
- plotting;
- summarization;
- import;
- model diagnostics;

reuse it.

Do not reproduce neighboring ecosystem functionality merely to make the vignette self-contained.

A good workflow can span packages.

---

### Step 6 — Prefer Explicit Composition

Show ordinary function composition.

Prefer:

```r
x <- preprocess(x)
result <- analyse(x)
plot(result)
```

over:

```r
workflow <- NewWorkflow(...)
workflow$configure(...)
workflow$run(...)
```

unless the workflow system is genuinely part of the package's supported API.

---

### Step 7 — Show the Result in Its Native Form

If the package returns an established result class, show users how to use that class directly.

For example:

```r
summary(result)
plot(result)
```

or an existing ecosystem plotting package.

Do not immediately convert every result into a package-specific table or class.

---

### Step 8 — Explain Only the Important Choices

Focus explanation on decisions that materially affect interpretation.

Examples:

- which assay was selected;
- how samples were aligned;
- what model formula means;
- which transformation was used;
- what the returned statistic represents.

Do not narrate every line of routine code.

---

### Step 9 — Interpret the Output

A workflow is incomplete if it only produces an object.

Explain:

```text
What should the user inspect?
What does the result mean?
What should they do next?
```

Keep interpretation within the method's actual scope.

Do not overstate statistical conclusions.

---

### Step 10 — Keep One Workflow Focus

Do not turn one vignette into a catalog of all package functions.

If several workflows answer different questions, separate them.

A useful documentation structure may be:

```text
Article 1 — joint ordination
Article 2 — association testing
Article 3 — penalized prediction
Article 4 — migration guide
```

rather than one giant article.

---

## Realistic Simplicity

Do not confuse "simple" with "toy".

A good example should be:

- small enough to understand;
- realistic enough to transfer;
- complete enough to run;
- explicit about assumptions.

Use small real or synthetic data when possible.

Avoid examples that hide important structure.

---

## Documentation Should Reflect Package Boundaries

If the package does not own preprocessing, do not imply that it does.

Example:

```text
Preprocess with mia.
Integrate with multiomics.
Visualize using an existing plotting tool.
```

This is preferable to reproducing all three layers in one package.

Clear package boundaries improve ecosystem understanding.

---

## Reuse Upstream Documentation Instead of Rewriting It

If an adapter delegates to:

```r
upstream::method(...)
```

do not reproduce the upstream package's complete method documentation.

Explain:

- what data the adapter supplies;
- what package-specific behavior occurs;
- which arguments are forwarded;
- where users can learn advanced upstream options.

Keep local docs focused on integration.

---

## Show Argument Forwarding Clearly

If `...` is passed upstream, say so.

Example:

```text
Additional arguments are forwarded to `vegan::rda()`.
```

Then demonstrate only one or two relevant options.

Do not copy the upstream argument reference.

---

## Show Alignment When It Matters

For multi-input analyses, users need to know how observations are matched.

Document:

- identifier used;
- whether order is changed;
- what happens to unmatched samples;
- whether exact overlap is required.

A concise alignment explanation may prevent major scientific errors.

---

## Show Orientation When It Matters

If containers store:

```text
features × samples
```

but upstream methods consume:

```text
samples × features
```

users may not need to perform the transpose themselves.

Still document the semantic orientation of inputs and outputs when relevant.

Do not expose internal conversion unnecessarily.

---

## Avoid Helper-Heavy Examples

A warning sign is a vignette containing many custom functions such as:

```r
prepare_data_for_package()
extract_package_result()
reshape_package_output()
fix_names_for_plotting()
```

If these are necessary for ordinary use, consider whether they should:

- use existing ecosystem tools;
- become a small public adapter;
- disappear through API simplification.

Do not normalize cumbersome examples by hiding complexity in vignette-only helpers.

---

## Avoid Artificial Self-Containment

Documentation does not need to prove that the package can do everything alone.

It is acceptable and often desirable to show:

```text
Package A for import
Package B for analysis
Package C for plotting
```

when those packages are established tools.

This demonstrates interoperability rather than weakness.

---

## Function Examples

Reference examples should be shorter than full workflows.

They should demonstrate:

- minimal valid input;
- main intended call;
- shape of result;
- perhaps one important option.

Do not turn every function example into a tutorial.

Use vignettes for complete workflows.

---

## README Examples

README examples should communicate the package's main conceptual value quickly.

Prefer:

```text
standard object
→ one meaningful package call
→ familiar result
```

Avoid long installation-to-publication workflows.

A reader should understand what the package contributes within a few examples.

---

## Vignette Structure

A strong workflow vignette often follows:

```text
1. Question
2. Data
3. Minimal preparation
4. Analysis
5. Result
6. Interpretation
7. Optional next step
```

This structure keeps the user problem central.

---

## One Method Per Section

Within a larger vignette, give each method its own clear purpose.

Avoid showing several methods back-to-back without explaining when to choose among them.

Prefer:

```text
Use RDA when ...
Use Mantel when ...
Use penalized regression when ...
```

based on scientific or analytical questions.

---

## Documentation and Optional Dependencies

If an example requires an optional package:

- state that clearly;
- keep the optional dependency local to that example;
- avoid making the main workflow depend on it unless appropriate.

Do not force all users to install large optional stacks for one visualization.

---

## Documentation and New Classes

If explaining a workflow requires a long section on a package-specific result class, reconsider whether that class is necessary.

Prefer established result types that users can inspect with existing tools.

New classes should earn their documentation burden.

---

## Documentation and Error Prevention

Use short notes for important mistakes users might make.

Examples:

```text
Samples are aligned by name, not position.
```

```text
The method expects transformed abundance values rather than raw counts.
```

Do not bury critical assumptions in long prose.

---

## Documentation and Defaults

Show default behavior when it is the recommended path.

Use explicit arguments when:

- the choice matters scientifically;
- reproducibility benefits;
- the default is not obvious.

Do not specify every default argument mechanically.

That makes examples noisy and fragile.

---

## Documentation and Version-Specific Behavior

Avoid examples tightly coupled to unstable dependency internals.

Use public APIs.

If behavior differs by dependency version, document only supported relevant distinctions.

Do not make the vignette a compatibility history unless it is a migration guide.

---

## Documentation and Reproducibility

Examples should be reproducible.

Where relevant:

- set seeds;
- avoid network dependence;
- use stable package data;
- use deterministic options;
- state required optional packages.

Do not compromise conceptual clarity with excessive environment boilerplate.

---

## Documentation and Performance

Examples should run in reasonable time.

Prefer small datasets and reduced iterations for demonstration.

If full analysis is expensive, explain that the example uses a reduced configuration.

Do not alter the method so much that the example teaches unrealistic behavior.

---

## Documentation and Tests

Executable documentation is valuable because it validates:

- public APIs;
- examples;
- interoperability;
- package integration.

Treat failing vignettes as possible API signals, not merely documentation problems.

---

## Workflow as API Audit

After writing the workflow, ask:

```text
How many package-specific concepts did the user have to learn?
```

and:

```text
How much code was integration plumbing rather than analysis?
```

If the answer is "too much", inspect the API.

Possible improvements may belong in later PRs.

Do not redesign the package inside the documentation PR.

---

## Documentation PR Scope

A workflow documentation PR should have one clear purpose.

Good:

```text
Add a vignette showing joint RDA using a MultiAssayExperiment.
```

Good:

```text
Update migration guide for `getRDA()` → `getJointRDA()`.
```

Avoid:

```text
rewrite all vignettes
+
rename functions
+
change defaults
+
add plotting dependency
```

Documentation may reveal problems, but fixes should be separate unless strictly necessary.

---

## Workflow Documentation Template

```text
# <Question-oriented title>

## Question

<What does the user want to learn?>

## Data

<What standard object is used?>

## Preparation

<Only required preprocessing, using existing tools>

## Analysis

<One or a few package functions>

## Result

<Show native result structure>

## Interpretation

<What should the user inspect or conclude?>

## Next steps

<Optional established downstream tools>
```

---

## Example 1 — Joint Ordination

Question:

```text
Do two omics layers share structured multivariate variation?
```

Workflow:

```text
MultiAssayExperiment
    ↓
select/preprocess assays with existing tools
    ↓
getJointRDA()
    ↓
rda result
    ↓
summary/established visualization
```

Documentation should focus on:

- assay selection;
- sample alignment;
- interpretation of ordination.

Do not teach a new result class if none is needed.

---

## Example 2 — Penalized Prediction

Question:

```text
Which features are useful for predicting a phenotype?
```

Workflow:

```text
standard container
    ↓
extract/select predictors
    ↓
getLASSO()
    ↓
glmnet-compatible result or clear coefficient result
    ↓
inspect selected features
```

Reuse upstream modeling concepts.

Do not recreate a machine-learning framework in the vignette.

---

## Example 3 — Migration Guide

Question:

```text
How do I update code from `oldFunction()` to `newFunction()`?
```

Show:

```r
# old
oldFunction(x)

# new
newFunction(x)
```

Then state:

```text
analysis behavior is unchanged
```

if true.

Do not turn the migration guide into a full method tutorial.

---

## Example 4 — Bad Workflow Documentation

```text
1. convert standard object to package object
2. create analysis config
3. register backend
4. create workflow
5. create result wrapper
6. convert result to data frame
7. use package-specific plotting object
```

If the user problem is simple, this workflow signals excessive conceptual machinery.

---

## Review Checklist

### User Focus

- [ ] The documentation starts from a real question.
- [ ] One workflow has one clear purpose.
- [ ] The package's contribution is visible.

### Simplicity

- [ ] Standard representations are used.
- [ ] Existing ecosystem tools are reused.
- [ ] Package-specific setup is minimal.
- [ ] No vignette-only framework hides complexity.

### Result

- [ ] The native result is shown.
- [ ] Existing summary/plotting tools are reused where appropriate.
- [ ] Interpretation is included.

### Correctness

- [ ] Important assumptions are explicit.
- [ ] Alignment/orientation is explained where relevant.
- [ ] Examples are reproducible.
- [ ] Optional dependencies are clearly identified.

### Scope

- [ ] Documentation changes do not hide API redesign.
- [ ] Unrelated tutorials are not rewritten.
- [ ] Problems discovered while documenting are recorded separately when appropriate.

---

## Completion Criteria

This skill is satisfied when:

- the workflow answers one real user question;
- it begins from a standard representation;
- setup is minimal and realistic;
- package-specific operations are easy to identify;
- existing preprocessing, analysis, and plotting tools are reused where appropriate;
- the result remains in an established representation where possible;
- interpretation is provided;
- important assumptions are explicit;
- documentation reveals rather than hides API complexity;
- the PR has one documentation purpose.

---

## Final Rule

> Teach the task, not the machinery.

Good documentation makes a capable package feel smaller because users only need to learn the concepts the package truly owns.
