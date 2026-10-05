# craftstep
Reusable Claude Code skills for individual steps of the software development
lifecycle - design, implementation and review - each one on-demand expertise
you invoke when you hit that step, not a one-size-fits-all agent.

The goal is to keep the developer in control of the process, with room to
course-correct early rather than after a large diff has already landed.

The plugin is intended for small to medium-sized projects maintained by single
developers or small teams. For more complex setups or full automation, an
[alternative framework](#alternative-frameworks) might work better.

## Installation

craftstep is installed from its [GitHub repository](https://github.com/andju/craftstep),
which doubles as a plugin marketplace named `craftstep`. The skills are then
available as `/craftstep:<name>`.

### Claude Code (CLI)

Add the marketplace and install the plugin:

```
/plugin marketplace add andju/craftstep
/plugin install craftstep@craftstep
```

The same works from your shell:

```
claude plugin marketplace add andju/craftstep
claude plugin install craftstep@craftstep
```

To update, run `/plugin marketplace update craftstep`.

### Claude Code for Visual Studio Code

1. Type `/plugins` in the prompt box to open **Manage plugins**.
2. In the **Marketplaces** tab, add `andju/craftstep`.
3. In the **Plugins** tab, click **Install** on `craftstep` and choose the scope:
   for you (all projects), for the project, or locally.
4. If asked, restart Claude to apply the plugin changes.

## Setting it up

Create the [project.md](#projectmd) file by executing:

```
/craftstep:setup
```

It works each section out from your project, shows you the draft with the evidence
behind every value, and asks you to accept, replace or omit each one before writing.
Sections your `CLAUDE.md` already covers are dropped, and a section whose value only
matches the documented default is recommended for omission. So a repo might ends up
with a short file or none at all. Re-run it later and it adds what's missing, leaving
the sections you already have untouched.

**Recommended model: Haiku.** It reads the repo, proposes, and writes one short file
— every value passes your acceptance before it lands, so a wrong guess costs a
keystroke.

## How it works

The skills are invoked, never triggered, so how much process a change gets is
your call.

### Small change

A typo, a copy tweak, a one-line guard, a rename, a version bump: the *what*
and the *how* are both already settled and the only work left is the typing:
Prompt directly.

### Feature

The everyday tier: the approach is obvious, but the scope isn't. Big enough
that you want to agree on what's being built before it's built, and want a
second pass over the result.

Optionally, start by cutting a feature branch. Nothing in craftstep requires
one, but the spec and the review findings are files that land in your working
tree, and a branch keeps them — along with the half-finished code — off `main`
until you're happy with all three.

#### Plan

Describe what you want to implement to `/craftstep:plan`:

```
/craftstep:plan Add user authentication with OAuth
```

It interviews you to clarify open points and writes an implementation-ready
spec. This is the cheapest point at which to change your mind: a wrong assumption
caught in the spec costs a sentence, the same one caught later costs a rewrite.

**Recommended model: Opus (xhigh).** The spec is executed literally by the next step,
so it has to be right about the codebase and complete enough that nothing is
left to decide later — and the interview has to push back where the plan won't
work. Those two degrade before anything else does on a smaller model, and a
spec that hedges still reads fine.

#### Review the spec

Review the spec, add comments if needed
(e.g., `<!-- Revisit if this statement is correct. -->`) and ask Claude Code to
review your comments. If you are using Visual Studio Code, I recommend the extension
[Markdown Pro](https://marketplace.visualstudio.com/items?itemName=AmartyaKhan.markdown-pro-commenter)
to add comments.

The spec is a working document — keep it on the feature branch, or gitignore
it. What happens to the file, once the feature is implemented, is your call:
leave it, delete it, or move it to an archive folder. No skill here cleans it
up.

#### Implement

Open a new Claude Code Session to get a fresh context window and start the
implementation:

```
/craftstep:implement
```

It will implement the spec, working one item at a time and verify each before
starting the next. Where the codebase contradicts the spec it stops and asks.

**Recommended model: Opus (high–xhigh).** The step splits itself across two
models: `/craftstep:implement` runs on the specified model and delegates the
work items to a Sonnet subagent. The specified model resolves the project
facts once, clears the spec for dispatch and hands down only what the spec
itself can't state — the subagent reads the rest from the spec — then runs the
full gate itself, proves every acceptance criterion, and owns the fix loop — at
most two rounds, each one re-run and re-checked rather than taken on the
subagent's word. Sonnet does the editing.

What the split buys is cost, and a check by a model that didn't write the code.
Implementation is the token-heaviest step of the lifecycle, and Sonnet costs
half as much per token — less in practice, since most of a long run's tokens
are cache reads, which cost the same on both. And Opus proves the result
without having made the choices behind it, so it has no reasoning of its own to
take for evidence. The same distance is what the split costs: Opus judges the
result from the working tree and from re-running the commands itself, not from
having watched the work happen — so a subagent that routed around an obstacle
instead of reporting it gets caught by the gate and the acceptance criteria, or
not at all.

#### Validate and fix

With the code written, it is time to validate it. craftstep provides two
different commands for code and tests:

```
/craftstep:review-code uncommitted changes
/craftstep:review-tests
```

For each finding a self-contained fix prompt is written into the `reviews`
folder (prefixed `code-` and `test-`). You can reference each file in a
prompt and ask to fix it.

The two reviews are independent; run either, both, or neither. Skipping
`review-tests` on a change with no test surface is normal.

**Recommended model: Opus (code: xhigh, tests: xhigh–max).** A missed finding
leaves no trace — you can't tell a thorough review from a shallow one by
reading the folder it produced — and each run reads its file set once, applying
every lens as it goes. `review-tests` spends half its pass on the tests that
*aren't* there, which needs a model of the covered code rather than a scan of
the suite. Neither review is expensive: a bounded set of files in, short
documents out.

### Complex feature — design the solution first

The *approach* is unsettled, not just the scope. The signals: more than one
plausible architecture, a choice that's expensive to reverse (a schema, a
public API, a dependency you'll live with), or a decision someone will ask you
to justify a year from now.

```
/craftstep:design-solution Migrate to a microservices architecture
```

It interviews you, agrees the criteria *before* weighing anything, puts
genuinely different options against them, and recommends one — writing a
decision record under `decisions`. Run `craftstep:plan` (in a separate session)
next, and use the record as input. From there the feature tier runs unchanged.

Running this skill on a change that has one obvious implementation wastes an
interview to rediscover that; the cost of skipping it when you shouldn't have
is a design decision made implicitly, inside a spec, with no record of what
lost.

**Recommended model: Opus, or Fable for a one-way door (xhigh).** Nothing downstream
checks a decision record: a weak one doesn't fail a test, it quietly misdirects
every spec written after it. The step also asks for self-restraint — hold three or four
genuinely different options at their strongest, and recommend none of them
before the criteria are agreed — which is what a smaller model drops first,
leaving one idea at three sizes. It costs an interview and one document, so
there is little to save here and a lot to lose; where the choice is expensive
to reverse, Fable's extra reasoning is worth the price on that few thousand
tokens.

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

### project.md

Facts you don't keep in `CLAUDE.md` live in one shared file per repo:
`.claude/project.md`. Every skill reads from this file, so the facts stay in
one place as the repo evolves. It exists specifically to resolve ambiguity the
repo doesn't settle, so where it contradicts what a skill would otherwise infer
from the code, the file wins.

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

- Named `*.test.ts`, wherever they sit — pure logic, no I/O, no framework mounting
- `tests/integration` — crosses a real boundary (db, filesystem, HTTP client)
- `tests/e2e` — drives the app through the browser; expensive, keep the count low

<!-- Two halves: the tiers, and how test files are named. The latter is important
     to identify co-located tests. Information is maintained to avoid expensive
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
````

## Alternative frameworks
craftstep is one of several attempts to bring discipline to AI-assisted development, but it's deliberately narrower than most: it supplies expertise for individual steps you invoke, not a methodology that runs your process for you.

[feature-dev](https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev), Anthropic's own plugin, is the closest neighbour: the same lifecycle, no methodology attached. It is one command, though — `/feature-dev` runs discovery, codebase exploration, architecture, implementation and review as seven phases of a single session, fanning out to explorer, architect and reviewer subagents, and holding the plan and the findings in that session's context.

> craftstep splits the same lifecycle into skills you invoke one at a time, and each writes its output to disk: a decision record, a spec you read before anything is implemented, one file per review finding. Nothing has to survive between steps in a context window, and the reviewer never remembers writing the code.

[Superpowers](https://github.com/obra/superpowers)' skills fire automatically the moment the agent notices you're building something, and they carry a specific methodology with them: a design doc gets approved before any plan exists, and implementation is strict TDD (red-green-refactor; code written before its test gets deleted).

> craftstep never triggers itself and isn't bound to a methodology: It only acts on the one step you called it for, and doesn't require a design doc or test-first discipline to use it.


[GSD Core](https://github.com/open-gsd/gsd-core) is a methodology with its own execution model: a phase loop it drives across sessions, coordinating work over parallel subagents and carrying project state forward itself.

> craftstep skills carry no state between invocations: each does its one step, leaves its output on disk and hands control straight back — there's nothing for it to resume, because it isn't running your process, you are. `implement` does run a loop — dispatch, gate, at most two fix rounds — but it's one skill deep and ends when that skill does, and every seam it crosses is a file you could have started from instead.

[gstack](https://github.com/garrytan/gstack) models an entire organization around that lifecycle — CEO, engineering manager, designer, security officer, release engineer — each a command with its own gate.

> craftstep's step list is bounded by the Software Development Life Cycle itself: designing a solution, writing a spec, implementing it, reviewing the result. It doesn't stand in for roles, only for the steps a solo engineer already owns.

## Compatibility

Skills in this repo use the `argument-hint` frontmatter field to document
expected arguments for their `/craftstep:<name>` invocations. `argument-hint`
is a Claude Code extension, not part of the portable Agent Skills spec. This
means these skills are Claude Code-only: packaging or uploading them anywhere
else that validates against the spec (claude.ai, the Skills API) fails with
an unexpected-key error rather than silently ignoring the field.
