---
name: design-a-function-family
description: "Use when planning a set of exported functions (a new package, a new module, or a catalogue of plots or methods): decide for each candidate whether it is a new function or a parameter of an existing one, remove redundant functions without creating mega-functions, keep a coverage map as evidence, and implement the smallest useful subset first."
---

# Design a Function Family

## Purpose

Use this skill when deciding which functions a package or module should
export, for example:

- the catalogue of a new plotting package;
- a family of analysis methods;
- getters, setters and wrappers around one kind of result.

Its purpose is to avoid two failure modes:

1. **too many functions**: several exported functions that are the same
   operation with a different input or option;
2. **mega-functions**: one function whose `type =` argument switches between
   operations, so that half of its arguments are ignored depending on the
   switch.

---

## Core Principle

> One exported function per operation. Different inputs and optional layers
> are parameters; a different operation is a different function.

---

## Relationship to Other Skills

- New functions follow `evolve-public-apis` for naming and arguments.
- Reuse of existing functions comes first: `reuse-before-implementing`.
- Each function is added with `one-semantic-change`.

---

## The Three-Step Test

For each candidate function, compare it with the closest existing or planned
function:

1. Do the inputs and outputs live in the same domain? (For plots: the same
   axes, such as time × value or component × component. For methods: the same
   kind of input and the same kind of result.)
2. Is the core operation the same? (For plots: the same geometry. For
   methods: the same algorithm family.)
3. Is the only difference which input is supplied, which grouping is applied,
   or which optional layer is added?

- If 1, 2 and 3 are all true, the candidate is a **parameter**.
- If 1 or 2 is false, it is a **separate function**.

Optional layers on the same axes (individual lines versus summaries, an extra
reference line, a fitted curve) count as parameters, not as new geometry.

---

## Four Kinds of Redundancy

| Kind | Description | Becomes |
|------|-------------|---------|
| Input | Same operation, different stored result or column | an input argument |
| Grouping | Study design mistaken for a function (sites, pairs, arms) | grouping or faceting arguments |
| Layer | Same axes, optional extra layer | a layer argument |
| Synonym | Two fields' names for the same operation | one function |

---

## False Redundancy

Do not merge functions only because they read the same input. Keep them
separate when the domain or the operation differs:

- the same data drawn on different axes;
- the same result shown with a different encoding (height versus colour
  bands, points versus flows);
- many observations per subject connected in time versus one point per
  subject.

When applying the test, compare every mode of the existing function, not only
its main mode. A candidate that differs from the main mode may still be a
layer of another mode.

---

## Avoiding Mega-Functions

A merge is acceptable only if:

- every argument is meaningful for every input;
- no `type =` argument changes the axes, the geometry or the meaning of other
  arguments;
- the documentation of each argument does not need "only when type is ...".

If a merge needs such a switch, keep separate functions.

---

## Coverage Map

Keep a table of every method or use case the family is meant to support, and
map each one to the function and argument that covers it:

```text
method → function(argument = value)   status: covered / depends on decision /
                                       partial / reused from another package /
                                       out of scope
```

Use the map:

- as evidence for each merge (nothing was lost);
- to find gaps after a merge (a dropped argument value usually shows up here);
- to record decisions that are still open.

The map is a working document. Keep it where the project keeps design notes.

---

## Implement Small

Design the whole family, but implement the smallest useful subset first:

- shared internals the first functions need;
- one or two functions that exercise those internals;
- the core arguments of each, with optional layers added in later changes.

Open issues only for what will be worked on soon. Keep the rest of the
catalogue in the design notes until it is needed.

---

## Review Checklist

- [ ] Every candidate passed or failed the three-step test explicitly.
- [ ] All modes of existing functions were compared.
- [ ] No merge needs a `type =` switch that changes axes or meaning.
- [ ] Every argument is meaningful for every input it accepts.
- [ ] The coverage map lists every intended method and its status.
- [ ] Open decisions are recorded, not silently resolved.
- [ ] Only the first subset has issues and code.

---

## Completion Criteria

This skill is satisfied when:

- each exported function has one operation;
- variants are parameters;
- false redundancies are kept separate;
- the coverage map shows nothing was lost;
- implementation starts with a small, useful subset.

---

## Final Rule

> Split by operation, merge by input, and prove with a coverage map that
> nothing fell through.
