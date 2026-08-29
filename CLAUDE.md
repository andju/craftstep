# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`craftstep` is a Claude Code **plugin**. It ships no application code — the
deliverable is prose: skill instructions that other repos install and invoke.

### Shipped vs. POC

`skills/` and `agents/` hold both at once — POC material is identified per
file, not per folder:

| Path | Status |
| --- | --- |
| [skills/plan/](skills/plan/) | **Shipped.** The reference implementation; a new skill copies its shape. |
| [skills/implement/](skills/implement/) | **Shipped.** Implements the spec `plan` writes, work item by work item. |
| [skills/review-code/](skills/review-code/) | **Shipped.** Reviews a named scope and writes one self-contained fix prompt per finding. |
| [agents/spec-step.md](agents/spec-step.md) | POC — single-step executor `implement` would delegate to. |
| [skills/review-to-prompts/](skills/review-to-prompts/) | POC — review step, superseded by `review-code`. |
| [skills/solution-design/](skills/solution-design/) | POC — design step. |

The POC entries are reference only and go away short- to mid-term. Don't extend
them and don't fix bugs in them.

## The contract skills must honour

Every skill in this plugin resolves its repo-specific facts from **three**
sources, checked in order, taking the first that settles each fact. A skill
never keeps its own config file — these three are all of them:

1. **The consuming repo's `CLAUDE.md`.** Usually already in context at launch;
   a skill reads it from disk only when it isn't — inside a subagent, or a
   non-root package in a monorepo.
2. **`.claude/project.md`** in the consuming repo, under a fixed section
   schema: `plans`, `reviews`, `commands`, `tests`, `docs`, `reuse`,
   `extra_checks`. A missing file, a missing section, and a missing key are
   all normal.
3. **Inference from the repo itself** — package manifest, CI config, existing
   test and docs layout — when neither source above settles the fact.

Rules that apply to every skill, and that a new skill must not quietly break:

- **Read `.claude/project.md` on demand**, at the point the fact is needed —
  never via `@import` from `CLAUDE.md`, which would load it at launch and cost
  context in sessions where no skill runs.
- **Every source, section, and key is optional.** When sources 1 and 2 are
  silent, infer from the repo and print every inferred fact in one block
  *before* acting on it, so the user can catch a bad guess early.
- **Precedence when sources disagree.** When `CLAUDE.md` and
- `.claude/project.md` give the *same* fact incompatible values, a skill stops
- and asks rather than picking one; a fact only one of them states is not a
- disagreement. Both beat inference from the code.
- **Never edit `.claude/project.md` or the consuming `CLAUDE.md` mid-run.** If
  a run turns up a fact worth recording, propose the addition at the end for
  the user to apply.
- **Need a fact the schema doesn't cover?** Propose a schema addition — and
  update the schema in `README.md` — rather than inventing a private key.

## Frontmatter

Skills use `argument-hint` to document their `/craftstep:<name>` arguments.
That field is a Claude Code extension, not part of the portable Agent Skills
spec — it makes these skills **Claude Code-only**.If it ever needs to change,
`README.md`'s Compatibility section changes with it.

## Keeping README and skills in sync

[README.md](README.md) is the public documentation of the
`.claude/project.md` schema and of how skills resolve project facts. A skill that
reads a section the README doesn't document, or a README section no skill
consumes, is a bug. Change both in the same commit.
