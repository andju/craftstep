# craftstep
Reusable Claude Code skills for individual steps of the software development
lifecycle - planning, implementation and review - each one on-demand expertise
you invoke when you hit that step, not a one-size-fits-all agent.

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

<!-- A single file: skills work one spec at a time.
     Defaults to `SPEC.md` at the repo root when unset. -->

## decisions

`docs/architecture/decisions/`

<!-- Where design and decision records live.
     Defaults to `docs/architecture/decisions/` when unset. -->

## reviews

`docs/code-review/`

<!-- The folder review findings are written to, one file per finding. Code and
     test review share it, prefixed `code-` and `test-` respectively.
     Defaults to `docs/code-review/` when unset. -->

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
- Named `*.test.ts`, wherever they sit — unit tests are co-located beside the code they cover

<!-- Two halves: the tiers, and how test files are named. The latter is important
     to identify co-locates tests. Information is maintained to avoid expensive
     discovery. -->

## docs

- `docs/user/` — user-facing.
- `docs/internal/` is not user-facing; changes there don't count as
  documentation for this checklist.

<!-- Set to `none` for projects with no user-facing docs. -->

## reuse

- `src/lib/components/` — shared UI components
- `src/lib/hooks/` — shared stateful logic
- `src/lib/api/` — service clients; never call fetch directly from a component
- `tests/fixtures/` — shared fixtures and factories; new test setup builds on these

<!-- Shared test fixtures, factories and helpers belong here too. -->

## extra_checks

- Does this need a database migration? If so, is it reversible?
- Does this add user-visible strings? They must go through the i18n catalogue.
- Does this need a feature flag, and what's the removal condition?
- Does this change a public API surface? If so, note the version impact.
````

## Related frameworks
craftstep is one of several attempts to bring discipline to AI-assisted development, but it's deliberately narrower than most: it supplies expertise for individual steps you invoke, not a methodology that runs your process for you.

[Superpowers](https://github.com/obra/superpowers)' skills fire automatically the moment the agent notices you're building something, and they carry a specific methodology with them: a design doc gets approved before any plan exists, and implementation is strict TDD (red-green-refactor; code written before its test gets deleted).

> craftstep never triggers itself and isn't bound to a methodology: It only acts on the one step you called it for, and doesn't require a design doc or test-first discipline to use it.


[GSD Core](https://github.com/open-gsd/gsd-core) is a methodology with its own execution model: a phase loop it drives across sessions, coordinating work over parallel subagents and carrying project state forward itself.

> craftstep skills own no loop and no state: Each does its one step and hands control straight back — there's nothing for it to resume or carry between invocations, because it isn't running your process, you are.

[gstack](https://github.com/garrytan/gstack) models an entire organization around that lifecycle — CEO, engineering manager, designer, security officer, release engineer — each a command with its own gate.

> craftstep's step list is bounded by the Software Development Life Cycle itself: designing a solution, writing a spec, implementing it, reviewing the result. It doesn't stand in for roles, only for the steps a solo engineer already owns.

## Compatibility

Skills in this repo use the `argument-hint` frontmatter field to document
expected arguments for their `/craftstep:<name>` invocations. `argument-hint`
is a Claude Code extension, not part of the portable Agent Skills spec. This
means these skills are Claude Code-only: packaging or uploading them anywhere
else that validates against the spec (claude.ai, the Skills API) fails with
an unexpected-key error rather than silently ignoring the field.