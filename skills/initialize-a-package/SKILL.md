---
name: initialize-a-package
description: "Use when creating a new package that will join an existing family of packages: check the name, initialize with the ecosystem's standard tooling, then align every file with the sibling packages, remove template placeholders, add infrastructure and dependencies only together with real content, and run the package check before the first commit."
---

# Initialize a Package

## Purpose

Use this skill when starting a new package, especially one that will sit next
to existing packages maintained by the same group.

Its purpose is to produce a first commit that is:

- created with the ecosystem's standard tooling;
- consistent with the sibling packages;
- free of template placeholders;
- free of infrastructure that has nothing to run yet;
- passing the package check.

The goal is not a feature-complete package.

The goal is a clean, honest skeleton that later changes can build on one
semantic change at a time.

---

## Core Principle

> Use the standard initializer, then make every file look like it belongs to
> the family, and commit nothing that only exists because a template put it
> there.

---

## Relationship to Other Skills

- Dependencies follow `introduce-dependencies-conservatively`.
- Each later addition follows `one-semantic-change`.
- Readiness for submission is `prepare-a-bioconductor-ready-package`.
- The first check follows `validate-a-package-change`.

---

## Procedure

### Step 1 — Check the Name

Before creating anything, check that the name is free:

- in the package repositories users install from (for R: CRAN and
  Bioconductor, compared case-insensitively);
- on the code-hosting platform;
- as an exported function name in widely used packages, where a clash would
  confuse users or mask functions.

Record the result. The name is the maintainer's decision.

---

### Step 2 — Initialize with the Standard Tooling

Use the initializer the ecosystem expects, not hand-written files.

For a Bioconductor package in R:

```r
usethis::create_package("<pkg>")
biocthis::use_bioc_pkg_templates()
biocthis::use_bioc_description()
# then the core-file steps of biocthis' templates, in their order
```

Follow the tool's own template order. Where a step cannot run (for example it
needs an interactive editor or a browser), skip it and note why.

Known pitfalls of running such tools non-interactively:

- helpers that write over existing files may silently do nothing without
  confirmation; check the result of each step instead of assuming it ran;
- template steps may install hooks or add dependencies that later changes
  make obsolete (for example a pre-commit hook that requires a README source
  file you later remove).

---

### Step 3 — Align with the Sibling Packages

List the files the sibling packages track, and align with them:

```text
metadata     → DESCRIPTION fields, license form, version scheme
changelog    → file name and entry format
README       → layout and sections
CI           → the workflow the family uses
build ignore → same patterns
```

Where the standard tooling and the siblings disagree, the maintainer decides.
Default to the siblings for anything users or CI see, and to the tooling for
anything the ecosystem's checks require.

Do not track files the siblings do not track (design notes, personal
scripts, extra community templates) unless the maintainer asks for them.

---

### Step 4 — Remove Placeholders

Templates contain text meant to be replaced. Before the first commit, every
placeholder must be filled with real content or removed:

- "The goal of ... is to ...";
- example sections with generic data;
- citations with empty titles or TODO identifiers;
- changelog entries describing template features;
- "edit this as you see fit" notes.

Search for them explicitly. A placeholder that reaches the first commit
usually stays for years.

---

### Step 5 — Add Infrastructure Only with Content

Infrastructure that has nothing to run either fails or misleads:

- no test runner until there is a real test (an empty test directory fails
  the check; a placeholder test only inflates coverage);
- no coverage step until there are tests;
- no dependencies until code uses them;
- a vignette only with real content, unless the ecosystem requires one for
  submission, in which case keep it minimal and factual.

Switch each piece on in the same change that adds its first real content.

---

### Step 6 — Check Before the First Commit

Run the package check and the ecosystem's checker on the skeleton.

Expected findings on a skeleton (record them, do not fix them artificially):

- "new submission" notes;
- links to a repository that does not exist yet;
- warnings that disappear with the first real dependency or test.

Any other finding is fixed before the first commit.

---

### Step 7 — First Commit and Repository

- Use the commit identity and license the maintainer chose.
- Create the repository only with the maintainer's approval.
- Let the first CI run finish and record its result.

---

## Review Checklist

- [ ] Name checked in package repositories, on the hosting platform and against
      common function names.
- [ ] Created with the standard initializer, steps in template order.
- [ ] Each tooling step verified, not assumed.
- [ ] Files, formats and CI aligned with the siblings.
- [ ] No tracked files the siblings do not have, unless requested.
- [ ] No template placeholders.
- [ ] No test, coverage or dependency infrastructure without content.
- [ ] Package check and ecosystem check run; only expected skeleton findings.
- [ ] Commit identity, license and repository approved by the maintainer.

---

## Completion Criteria

This skill is satisfied when:

- the name is free and decided;
- the skeleton was produced by the standard tooling;
- it looks like a member of its package family;
- it contains no placeholders and no empty infrastructure;
- the checks show only the expected skeleton findings;
- the first CI run passed.

---

## Final Rule

> Start from the standard, finish in the family style, and commit only what
> already has a reason to exist.
