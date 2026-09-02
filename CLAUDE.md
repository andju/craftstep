# CLAUDE.md

`craftstep` is a Claude Code **plugin**. It ships no application code — the
deliverable is prose: skill instructions that other repos install and invoke.

## The contract skills must honour

Every skill in this plugin resolves its repo-specific facts from **three**
sources, checked in order, taking the first that settles each fact. A skill
never keeps its own config file — these three are all of them (precedence in
this order):

1. **The consuming repo's `CLAUDE.md`.** Usually already in context at launch;
   a skill reads it from disk only when it isn't — inside a subagent, or a
   non-root package in a monorepo.
2. **`.claude/project.md`** in the consuming repo, under a fixed section
   schema: `plans`, `decisions`, `reviews`, `commands`, `tests`, `docs`, `reuse`,
   `extra_checks`. A missing file, a missing section, and a missing key are
   all normal.
3. **Inference from the repo itself** — package manifest, CI config, existing
   test and docs layout — when neither source above settles the fact.

Rules that apply to every skill:

- **Every source, section, and key is optional.** When sources 1 and 2 are
  silent, infer from the repo and print every inferred fact in one block
  *before* acting on it, so the user can catch a bad guess early.
- **Never edit `.claude/project.md` or the consuming `CLAUDE.md` mid-run.** If
  a run turns up a fact worth recording, propose the addition at the end for
  the user to apply.
- **Need a fact the schema doesn't cover?** Propose a schema addition — and
  update the schema in `README.md` — rather than inventing a private key.
