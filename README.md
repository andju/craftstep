# craftstep
Reusable Claude Code skills for individual steps of the software development
lifecycle - design, implementation and review - each one on-demand expertise
you invoke when you hit that step, not a one-size-fits-all agent.

| Step | Run | Leaves on disk |
|---|---|---|
| Once per repo, optional | `/craftstep:setup` | `.claude/project.md` |
| Settle the approach — complex features only | `/craftstep:design-solution <problem>` | a decision record |
| Agree on what gets built | `/craftstep:plan <feature>` | a spec |
| Read the spec, comment where it's wrong | you | a spec you agree with |
| Build it, in a fresh session | `/craftstep:implement` | code gated and checked against the spec |
| Review the result — either, both or neither | `/craftstep:review-code <scope>`<br>`/craftstep:review-tests <scope>` | a checklist of findings |
| Fix the findings, one per run | `/craftstep:fix [finding]` | the fix, ticked off in the checklist |
| Close the feature out | `/craftstep:wrap-up` | deferred work in the backlog; commit message |
| A bug — find the cause first | `/craftstep:diagnose <symptom>` | a finding for `/craftstep:fix` |
| An open question blocks a decision | `/craftstep:spike <question>` | a spike report |

Every step hands over through a file, not a context window, so each one can
start in a fresh session. Those files are grouped in one folder,
[`.craftstep/`](#the-craftstep-folder) at the repo root.
[How it works](#how-it-works) covers each step and what it needs from a model.

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
matches the documented default is recommended for omission. So a repo might end up
with a short file or none at all. Re-run it later and it adds what's missing, leaving
the sections you already have untouched.

<details>
  <summary>Recommended model: Haiku</summary>
  
  What it needs from a model: speed, not judgement.
  
  It reads the repo, proposes, and writes one short file — every value passes
  your acceptance before it lands, so a wrong guess costs a keystroke.
</details>

### The `.craftstep/` folder

The skills write their working files into `.craftstep/` at the repo root:

| File | Written by | Lifetime |
|---|---|---|
| `spec.md` | `/craftstep:plan` | one feature — until wrap-up |
| `backlog.md` | `/craftstep:wrap-up` | across features — until it's empty |
| `code-findings.md`, `test-findings.md` | the reviews | until the findings are worked off |
| `bug-findings.md` | `/craftstep:diagnose` | until the bugs are fixed |
| `spike-<slug>.md` | `/craftstep:spike` | until the design or spec it fed is written |

The spec and the backlog can live elsewhere — see [`plans` and `backlog`](#projectmd).
Decision records are never written here: they go under `decisions`, with your docs.

What gets committed is your call. A reasonable start is a `.craftstep/.gitignore`
that ignores everything but the backlog — the one file meant to last — and itself:

```gitignore
*
!.gitignore
!backlog.md
```

Remove a line, or the whole file, to share more — committed findings let a
teammate run `/craftstep:fix` on your review.

**The catch: ignored files exist only in the checkout that wrote them.** A fresh
clone, another machine or a second git worktree doesn't have them, so a spec
written in one worktree isn't there for `/craftstep:implement` in another, and
the same goes for findings and `/craftstep:fix`. If you split a feature across
worktrees or machines, commit the files you hand over (on a feature branch,
they stay off `main`), or copy them across.

## How it works

The skills are invoked, never triggered, so how much process a change gets is
your call.

Each step says what it needs from a model, then which model meets that today.
Line-ups change faster than the demands of a step do, so when the
recommendation dates, the first part still tells you what to pick.

### Small change

A typo, a copy tweak, a one-line guard, a rename, a version bump: the *what*
and the *how* are both already settled and the only work left is the typing:
Prompt directly.

### Feature

The everyday tier: the approach is obvious, but the scope isn't. Big enough
that you want to agree on what's being built before it's built, and want a
second pass over the result.

Optionally, start on a feature branch: it keeps the spec, the review findings
and the half-finished code off `main` until you're happy with all three.

#### Plan

Describe what you want to implement to `/craftstep:plan`:

```
/craftstep:plan Add user authentication with OAuth
```

It interviews you to clarify open points and writes an implementation-ready
spec. This is the cheapest point at which to change your mind: a wrong assumption
caught in the spec costs a sentence, the same one caught later costs a rewrite.

<details>
  <summary>Recommended model: Opus (xhigh)</summary>
  
  What it needs from a model: completeness, and the nerve to push back.
  
  The spec is executed literally by the next step, so it has to be right about
  the codebase and complete enough that nothing is left to decide later — and
  the interview has to push back where the plan won't work. Those two degrade
  before anything else does on a smaller model, and a spec that hedges still
  reads fine.
</details>

#### Review the spec

Review the spec, add comments if needed
(e.g., `<!-- Revisit if this statement is correct. -->`) and ask Claude Code to
review your comments. If you are using Visual Studio Code, I recommend the extension
[Markdown Pro](https://marketplace.visualstudio.com/items?itemName=AmartyaKhan.markdown-pro-commenter)
to add comments.

#### Implement

Start the implementation in a fresh session:

```
/craftstep:implement
```

It will implement the spec, working one item at a time and verify each before
starting the next. Where the codebase contradicts the spec it stops and asks.

<details>
  <summary>Recommended model: Opus (high–xhigh) for the session</summary>

  What it needs from a model: scepticism on top, diligence underneath.

  The step splits across two models. Your session's model clears the spec for
  dispatch, then runs the full gate and proves every acceptance criterion
  itself; the work items go to a subagent on a cheaper, mid-tier model (the
  `sonnet` alias, set in [spec-implementer.md](agents/spec-implementer.md)),
  with at most two fix rounds. Implementation is the token-heaviest step, so
  that is where a cheaper model saves the most — and the result is checked by a
  model that didn't write it.
</details>

#### Validate and fix

With the code written, it is time to validate it. craftstep provides two
different commands for code and tests:

```
/craftstep:review-code uncommitted changes
/craftstep:review-tests uncommitted changes
```

Each review writes one document into `.craftstep/`: a checklist on top, then
for each finding what's wrong, where, and which way the fix should go. Not the
fix itself — [`/craftstep:fix`](#fix) decides that.

The two reviews are independent; run either, both, or neither. Skipping
`review-tests` on a change with no test surface is normal.

<details>
  <summary>Recommended model: Opus (code: xhigh, tests: xhigh–max)</summary>

  What it needs from a model: thoroughness you can't check afterwards.

  A missed finding leaves no trace — you can't tell a thorough review from a
  shallow one by reading the document it produced — and each run reads its file
  set once, applying every lens as it goes. `review-tests` spends half its pass
  on the tests that *aren't* there, which needs a model of the covered code
  rather than a scan of the suite. Neither review is expensive: a bounded set of
  files in, one short document out. Each sizes its set before reading, prints a
  token estimate, and asks before going past about 8,000 lines.
</details>

#### Fix

Work the findings off one at a time — the one you name, or the next one still
open:

```
/craftstep:fix
```

Each run starts by checking the finding against the code as it is now — it is
a claim from one read, and earlier fixes may have moved things. A
low-confidence finding is never applied on its word: it is settled from the
code, by a regression test that has to fail first, or by asking you.

Mechanical fixes go ahead, since the diff is yours to read before you commit.
Where a fix decides something the finding left open — a public name, an
error-handling policy, behaviour a caller can see — it asks first. Add `ask`
or `no-ask` to force or skip that question.

Each run ticks its finding off in the checklist — fixed, not a defect, or left
open because it needs you. One finding per run, each in a fresh session, since
a review's worth of fixes in one context pays for every earlier one again on
every turn.

<details>
  <summary>Recommended model: Opus (high) for the session</summary>

  What it needs from a model: doubt before the edit, judgement in the fix,
  diligence in the edit.

  The same split as [Implement](#implement): your session's model checks the
  finding, decides the fix and applies the smallest ones itself; anything larger
  goes to a subagent on the `sonnet` alias (set in
  [finding-fixer.md](agents/finding-fixer.md)), whose checks the session re-runs.
</details>

#### Wrap up

When the feature is done, close it out:

```
/craftstep:wrap-up
```

It does three things, in an order that matters: the second may delete the
spec, and the commit message comes last so no question hides it:

1. **Parks what the spec deferred.** Every slice under the spec's **Deferred**
   heading — and any work item you decide not to finish now — becomes an entry
   in the backlog, complete enough to plan from once the spec is gone. If the
   feature itself came from a backlog entry, that entry is removed.
2. **Asks what to do with the spec** — delete it, leave it, or move it to an
   archive folder — recommending what `plans` records under `After wrap-up:`,
   or deletion when nothing is recorded.
3. **Drafts a commit message** from the spec, as the last thing in its final
   report; it never commits.

The spec is the only file it will remove. Findings documents are outside its
scope: it neither reports on them nor deletes them.

<details>
  <summary>Recommended model: Sonnet</summary>

  What it needs from a model: diligence, not judgement.

  It carries text from one file to another and summarizes a spec; nothing in it
  is a call the spec hasn't already made.
</details>

### Backlog — pick up where the last feature stopped

Wrap-up parks every slice a feature deferred in the backlog. To plan the next
one, start a fresh session:

```
/craftstep:plan next
```

`next` takes the first entry whose dependencies are met; give an entry's title
instead to pick that one. The entry stands in for the feature description, so
what it records as settled isn't asked again. From there the
[feature tier](#feature) runs unchanged, starting at [Plan](#plan).

### Bug — find the cause first

Something is broken and you don't know why. A fix written before the cause is
known tends to patch the line that throws rather than the one that is wrong.

```
/craftstep:diagnose Saving a draft twice throws "duplicate key" in orders.test.ts
```

It pins the symptom, reproduces it with the cheapest command that shows it, and
works back to the root cause one hypothesis at a time — stopping to ask after
three that didn't hold. It edits no code: the result is one finding in
`.craftstep/bug-findings.md`, with the reproduction, the mechanism and the
direction of the fix, which `/craftstep:fix bug-001` then works off like any
review finding. A cause it couldn't confirm is written at low confidence, so
the fixer settles it with a failing test before changing anything.

<details>
  <summary>Recommended model: Opus (high–xhigh)</summary>

  What it needs from a model: the discipline to check before believing.

  The failure mode is a plausible cause accepted without a test of it.
</details>

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
decision record under `decisions`. Run `craftstep:plan` (in a fresh session)
next, and use the record as input. From there the feature tier runs unchanged.

<details>
  <summary>Recommended model: Opus, or Fable for a one-way door (xhigh)</summary>

  What it needs from a model: judgement nothing will check, and restraint.

  Nothing downstream checks a decision record: a weak one doesn't fail a test, it
  quietly misdirects every spec written after it. The step also asks for
  self-restraint — hold three or four genuinely different options at their
  strongest, and recommend none of them before the criteria are agreed — which is
  what a smaller model drops first, leaving one idea at three sizes. It costs an
  interview and one document, so there is little to save here and a lot to lose;
  where the choice is expensive to reverse, the most capable model's extra
  reasoning is worth the price on that few thousand tokens.
</details>

#### Spike

Sometimes a decision hangs on a question only running code answers: is the
library fast enough, does the API return what the option needs.

```
/craftstep:spike Can the PDF renderer handle 500-page documents under 2 seconds?
```

It agrees the question with you — and what result means yes, no or inconclusive
— before writing anything, then builds the least that answers it in a separate
git worktree, so your working tree is never touched. The result is a spike
report, `.craftstep/spike-<slug>.md`: the answer, the evidence, and what
the spike doesn't show. The code is thrown away; anything worth keeping is
rebuilt through `/craftstep:plan`. Point `/craftstep:design-solution` or
`/craftstep:plan` at the report to use it.

A design can take a second round the same way a spec can. When a
recommendation rests on something nobody has tested, the record is written as
`proposed` and names that assumption as a spike question. Spike it, then finish
the record in a fresh session:

```
/craftstep:spike docs/architecture/decisions/0007-pdf-rendering.md
/craftstep:design-solution docs/architecture/decisions/0007-pdf-rendering.md .craftstep/spike-pdf-render-speed.md
```

The second run doesn't interview again. The criteria agreed in the first round
stay frozen — evidence may change which option wins, not what the options are
judged on — and the same record is rewritten in place as `accepted`. An
`accepted` record is never revised: reopening it is a new run and a new record.

What has to last from the report is carried into the decision record, which
never links back to it.

<details>
  <summary>Recommended model: Sonnet, or Opus (medium)</summary>

  What it needs from a model: speed, and honesty about the result.

  The code is disposable, so quality buys nothing; what matters is stopping at
  the answer and reporting *inconclusive* when that is what it is.
</details>

## Project facts

Skills need a handful of repo-specific facts that they would otherwise have
to guess at or ask you about. For any one fact, a skill takes the first of
three sources that settles it: the repo's own `CLAUDE.md`, then
`.claude/project.md`, then inference from the repo itself — and it says which
facts were inferred.

Put a fact in `CLAUDE.md` when every session should have it in context anyway;
put it in `.claude/project.md` when only the skill that asks for it needs it — that
file is read on demand, so facts parked there cost no context in sessions where no
skill runs.

### project.md

The schema is organized into sections, each covering one category of
repo-specific fact. Any section, or the file itself, may be absent. **Skip any
section whose content your `CLAUDE.md` already describes** — a second copy is
one more thing to keep current.

````markdown

## plans

`docs/specs/current.md`

After wrap-up: move to `docs/specs/archive/`

<!-- A single file: skills work one spec at a time.
     Defaults to `.craftstep/spec.md` when unset.
     `After wrap-up:` is what `/craftstep:wrap-up` recommends doing with the
     spec once the feature is closed out: `delete`, `leave`, or
     `move to <folder>`. It still asks first. Defaults to `delete` when
     unset. -->

## backlog

`docs/backlog.md`

<!-- A single file: work agreed but not yet specified. Written by
     `/craftstep:wrap-up`, read by `/craftstep:plan next`.
     Defaults to `.craftstep/backlog.md` when unset. -->

## decisions

`docs/architecture/decisions/`

<!-- Where design and decision records live. Never inside `.craftstep/`.
     Defaults to `docs/architecture/decisions/` when unset. -->

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

[mattpocock-skills](https://github.com/mattpocock/skills) starts from the same complaint — frameworks that own the process take away your control — and answers it with small, composable skills: get grilled about a plan, turn it into a spec, split that into tracer-bullet tickets, implement, review the diff against your standards and the spec. It carries a discipline of its own: implementation drives red-green-refactor TDD at agreed seams, a `GLOSSARY.md` and ADRs keep one domain language, and specs and tickets go to your issue tracker — GitHub, GitLab or a local folder. You type the skills that orchestrate; the ones that hold the discipline, TDD and code review among them, the agent can also reach for on its own.

> craftstep shares the stance but not the discipline: no skill fires on its own, none asks for test-first or a glossary, and there is no tracker to configure — each step leaves a plain file in your working tree. It is also narrower: the steps of one change, from decision to wrap-up, with nothing around them for triage or handoffs.

[OpenSpec](https://github.com/Fission-AI/OpenSpec) makes the same first move — agree on a spec before any code is written — and then keeps the spec. Each change gets a folder with a proposal, a design, a task list and spec deltas (requirements with WHEN/THEN scenarios); `/opsx:apply` works through the tasks, and archiving the change merges its deltas into `openspec/specs/`, a living description of what the system does. An npm CLI installs its slash commands into 30+ AI assistants, and it is built for existing codebases as much as new ones.

> craftstep keeps a record of *why* — the decision records — but not of *what*: its spec serves one change, and once that change is in, the code is the source of truth and the file is yours to delete. There is no CLI or folder convention to adopt, and the weight sits on the steps around the spec rather than on its format — an interview that pushes back before it's written, a gate and proven acceptance criteria after implementation, and reviews from a session that never saw the code being written.

Further afield:

- [feature-dev](https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev), Anthropic's own plugin, covers the same lifecycle as one command: seven phases in a single session, with the plan and the findings held in its context. craftstep splits it into steps you invoke one at a time, each handing over through a file.
- [Superpowers](https://github.com/obra/superpowers)' skills fire automatically and carry a methodology: an approved design doc before any plan, then strict TDD. craftstep never triggers itself and requires neither.
- [GSD Core](https://github.com/open-gsd/gsd-core) drives a phase loop of its own across sessions and parallel subagents, carrying project state forward. craftstep skills carry no state between invocations.
- [gstack](https://github.com/garrytan/gstack) models an entire organization — CEO, engineering manager, designer, security officer, release engineer — each a command with its own gate. craftstep stands in for the steps a solo engineer already owns, not for roles.

## Compatibility

The skills are Claude Code-only: they use the `argument-hint` frontmatter
field, a Claude Code extension that validators of the portable Agent Skills
spec (claude.ai, the Skills API) reject with an unexpected-key error.
