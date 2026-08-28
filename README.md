# craftstep
Reusable Claude Code skills for individual steps of the software development lifecycle - spec writing, implementation and code review - each one on-demand expertise you invoke when you hit that step, not a one-size-fits-all agent.

## Project facts

Skills need a handful of repo-specific facts that they would otherwise have
to guess at or ask you about. Two files supply them: the repo's own
`CLAUDE.md`, and `.claude/project.md`.

Put a fact in `CLAUDE.md` when every session should have it in context anyway;
put it in `.claude/project.md` when only the skill that asks for it needs it — that
file is read on demand, so facts parked there cost no context in sessions where no
skill runs.

For any one fact, a skill takes the first of three sources that settles it,
in this order: the repo's own `CLAUDE.md`, then `.claude/project.md`, then
inference from the repo itself.

### `.claude/project.md`

Facts you don't keep in `CLAUDE.md` live in one shared file per repo:
`.claude/project.md`. Every skill in this repo reads from this file, so the
facts stay in one place as the repo evolves. It exists specifically to resolve
ambiguity the repo doesn't settle, so where it contradicts what a skill would
otherwise infer from the code, the file wins.

The schema is organized into sections, each covering one category of
repo-specific fact. **Skip any section whose content your `CLAUDE.md` already
describes** — skills look there first, and a second copy is one more thing to
keep current. Any section, or the file itself, may be absent; skills fall
back to inferring from the repo and say which facts were inferred:

````markdown

## plans

`SPEC.md`

<!-- Or `docs/specs/<feature-slug>.md` if you keep specs per-feature rather than
     overwriting a single scratch file. -->

## commands

```bash
npm test         # test
npm run lint     # lint
npm run build    # build
```

## tests

- `tests/unit` — pure logic, no I/O, no framework mounting
- `tests/integration` — crosses a real boundary (db, filesystem, HTTP client)
- `tests/e2e` — drives the app through the browser; expensive, keep the count low

## docs

`docs/user/` — user-facing. `docs/internal/` is not user-facing; changes there don't
count as documentation for this checklist.

<!-- Set to `none` for projects with no user-facing docs. -->

## reuse

- `src/lib/components/` — shared UI components
- `src/lib/hooks/` — shared stateful logic
- `src/lib/api/` — service clients; never call fetch directly from a component

## extra_checks

- Does this need a database migration? If so, is it reversible?
- Does this add user-visible strings? They must go through the i18n catalogue.
- Does this need a feature flag, and what's the removal condition?
- Does this change a public API surface? If so, note the version impact.
````

## Compatibility

Skills in this repo use the `argument-hint` frontmatter field to document
expected arguments for their `/craftstep:<name>` invocations. `argument-hint`
is a Claude Code extension, not part of the portable Agent Skills spec. This
means these skills are Claude Code-only: packaging or uploading them anywhere
else that validates against the spec (claude.ai, the Skills API) fails with
an unexpected-key error rather than silently ignoring the field.
