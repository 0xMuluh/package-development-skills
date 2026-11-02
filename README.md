# Package Development Skills

Agent skills for developing and maintaining software packages, written for
R/Bioconductor work but mostly language-agnostic. Each skill is a folder with
a `SKILL.md` file in the [Agent Skills](https://agentskills.io) format, so it
works with Cursor, Antigravity, Claude Code, Codex and other agents that
support the standard.

## Skills

| Skill | When to use |
|-------|-------------|
| [`add-a-thin-adapter`](skills/add-a-thin-adapter/SKILL.md) | When adding package functionality that connects existing data structures to an established external implementation |
| [`characterize-before-changing`](skills/characterize-before-changing/SKILL.md) | Before changing existing behavior that is insufficiently understood or insufficiently tested |
| [`core-development-philosophy`](skills/core-development-philosophy/SKILL.md) | As the default engineering philosophy for developing, extending, updating, and maintaining software packages |
| [`delegate-with-a-review-gate`](skills/delegate-with-a-review-gate/SKILL.md) | When implementation work is handed to other agents, tools or contributors while one reviewer keeps commit and push rights |
| [`deprecate-and-replace`](skills/deprecate-and-replace/SKILL.md) | When retiring a public API, argument, behavior, representation, or package location and replacing it with a preferred alternative |
| [`design-a-function-family`](skills/design-a-function-family/SKILL.md) | When planning a set of exported functions (a new package, a new module, or a catalogue of plots or methods) |
| [`design-with-existing-representations`](skills/design-with-existing-representations/SKILL.md) | When deciding how data, results, metadata, configuration, or state should be represented in a package |
| [`document-a-real-workflow`](skills/document-a-real-workflow/SKILL.md) | When creating or revising package documentation that teaches users how to accomplish a real task |
| [`evolve-public-apis`](skills/evolve-public-apis/SKILL.md) | When changing a package's public interface |
| [`fix-one-bug`](skills/fix-one-bug/SKILL.md) | When correcting a specific defect in existing software |
| [`initialize-a-package`](skills/initialize-a-package/SKILL.md) | When creating a new package that will join an existing family of packages |
| [`introduce-dependencies-conservatively`](skills/introduce-dependencies-conservatively/SKILL.md) | When adding, removing, replacing, upgrading, or otherwise changing how a package depends on external software |
| [`migrate-functionality-without-redesign`](skills/migrate-functionality-without-redesign/SKILL.md) | When moving functionality from one package, module, namespace, file, or subsystem to another while preserving existing behavior |
| [`one-semantic-change`](skills/one-semantic-change/SKILL.md) | When planning, implementing, reviewing, or revising a pull request to enforce one semantic purpose per PR |
| [`prepare-a-bioconductor-ready-package`](skills/prepare-a-bioconductor-ready-package/SKILL.md) | When creating, maturing, or auditing an R package for Bioconductor submission or continued Bioconductor readiness |
| [`refactor-without-behavioral-change`](skills/refactor-without-behavioral-change/SKILL.md) | When changing code structure while intentionally preserving observable behavior |
| [`remove-proven-dead-code`](skills/remove-proven-dead-code/SKILL.md) | When removing code, helpers, branches, files, dependencies, exports, tests, or compatibility paths believed to be unused or obsolete |
| [`reuse-before-implementing`](skills/reuse-before-implementing/SKILL.md) | Before adding new code, new abstractions, new dependencies, new classes, new helpers, or new infrastructure |
| [`review-a-pull-request`](skills/review-a-pull-request/SKILL.md) | When reviewing a pull request or a diff written by someone else (a contributor, a co-maintainer or another agent) |
| [`test-at-the-behavioral-boundary`](skills/test-at-the-behavioral-boundary/SKILL.md) | When adding or revising tests for package behavior |
| [`validate-a-package-change`](skills/validate-a-package-change/SKILL.md) | As the final verification gate for a package change before review or merge |

## Installation

With Node.js:

```bash
npx skills add 0xMuluh/package-development-skills -g
```

This asks which agents to install the skills for. `-g` makes them available
in all projects. Update later with `npx skills update`.

Without Node.js, clone the repository and link the skills into your agent's
skills folder:

```bash
git clone https://github.com/0xMuluh/package-development-skills.git
```

| Agent | Global skills folder |
|-------|----------------------|
| Cursor | `~/.cursor/skills/` |
| Antigravity | `~/.gemini/config/skills/` |
| Claude Code | `~/.claude/skills/` |

For example, for Claude Code:

```bash
mkdir -p ~/.claude/skills && for d in package-development-skills/skills/*/; do ln -s "$(realpath "$d")" ~/.claude/skills/; done
```

Update with `git pull` in the cloned repository.

## License

MIT, see [LICENSE](LICENSE).
