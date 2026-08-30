---
name: review-tests
description: Audit the tests for a scope I name and write one self-contained fix prompt per finding into the review folder. Never writes or repairs a test itself.
argument-hint: [what to review — e.g. the whole suite, tests/api, the auth flow, since main; blank asks]
allowed-tools: Read, Grep, Glob, Bash(git:*), Bash(mkdir:*), Write, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are reviewing tests. Scope for this run (blank → step 1 settles it with me): $ARGUMENTS

Every finding lands on disk as its own file, written for a fresh session that has no memory of
this review and will be handed **one such file, or the whole folder, and nothing else**.

**This skill only writes inside `review` folder.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording,
propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (subagents, non-root
   packages in a monorepo).
2. **`.claude/project.md`**, sections `reviews`, `commands`, `tests`, `reuse`, `extra_checks`. A
   missing file, section or key are all normal.
3. **The repo itself** — manifest, runner and coverage config, CI workflows, test layout.

What each fact does here, and how to infer it at source 3:

- **reviews** — where findings are written. Infer nothing; default `docs/code-review/`.
- **tests** — the tiers, what belongs in each, file naming. Both the review set and where a missing
  test goes. Infer from runner config, CI and language convention; confirm by globbing.
- **commands** — how each tier is run; every finding names the command that proves its fix. You
  never run them here. Infer from manifest scripts and CI.
- **reuse** — shared units, including fixture, factory and helper directories. Duplicated setup is
  a finding only when a reusable unit exists. Infer from where tests import their helpers.
- **extra_checks** — cross-cutting questions every change is held against, asked here of the
  suite: what test fails if that promise breaks? Infer nothing.

List every fact you inferred rather than read, in one short block, before continuing.

## Step 1 — Settle the scope, then print the review set

The set has two halves — the tests, and the code they are meant to cover; weak-test findings come
from the first, missing-test findings from the second. Turn $ARGUMENTS into both, explicitly,
*before* reading any of it:

- **A test path or tier** ("tests/api", "the unit tests") — glob it; the covered code is what
  those tests import.
- **A code path or subsystem** ("src/payments", "the auth flow") — glob or grep it, then find its
  tests by the `tests` patterns anywhere in the tree, co-located ones included, matching on the
  symbols they import. List what you landed on, and say the attribution is your guess, not mine.
- **A range** ("since main", "this branch") — resolve the base against the repo's actual default
  branch, then `git diff <base>...HEAD`, split into changed tests and changed code. Changed code
  whose tests didn't move is the point of looking.
- **The whole suite** — every file matching the `tests` patterns, wherever it sits.
- **Empty** — AskUserQuestion, offering the options above this repo actually has, each with its
  file count so the choice is grounded.

Also in scope unless I say otherwise: runner and coverage config, the CI workflows that run tests,
shared fixtures and factories, any coverage report already in the tree. Out: generated, vendored
and lockfile paths, and anything not readable as text. Print all of it before reading it — both
halves, config, exclusions and why, and the commit or working-tree state under review. Above 150
files, stop and offer to narrow; a suite you skimmed produces confident-sounding noise.

## Step 2 — One pass, all lenses

Ground yourself first: read the runner config and the CI workflow that runs tests.

Walk the set once, applying every lens per file rather than re-scanning per lens. Never re-read a
file. Capture the line numbers and snippets you need at read time.

**Is what's there sound** — `approach`, `weak test`, `remove/replace`:

1. **Shape and boundaries** — the balance of fast narrow tests against slow broad ones, and
   whether it fits this kind of system; a unit under test that was never defined; mocking
   third-party internals instead of an owned seam; tests that assert only their own mocks.
2. **Determinism** — wall-clock time, timezone, locale, unseeded randomness, real network, real
   filesystem, order dependence between tests, shared mutable state, `sleep` used as
   synchronisation, anything unsafe under the parallelism CI actually configures.
3. **Isolation and data** — state leaking between tests, a hand-maintained shared database,
   fixture sprawl, per-test setup cost that belongs to the suite.
4. **Assertion quality** — smoke-only tests that assert nothing capable of failing,
   over-specification that breaks on a harmless refactor, snapshots standing in for expectations,
   assertions on implementation where behaviour was the point.
5. **Enforcement** — which tiers actually gate a merge, and what coverage measures, excludes and
   enforces. Coverage is a signal, never a target: high coverage over weak assertions is the
   finding; a percentage is not.
6. **Readability** — names that state the behaviour, arrange/act/assert clarity, helper
   indirection deep enough to hide what is being asserted, duplication a `reuse` unit already
   covers. Test code is held to the standard of production code.

**What isn't there** — `missing test`, in risk order:

7. **Untested units** — exported functions, classes and modules in the covered code that no test
   reaches.
8. **Risk the suite doesn't reach** — critical paths (auth, permissions, money, migration,
   deletion, concurrency, retries, idempotency); failure paths (raised errors, timeouts, partial
   failure, rollback, malformed input, rate limits); boundaries (empty, null, zero, one, maximum,
   unicode, very large, duplicate, out-of-order).
9. **Seams** — API and DB schemas, message formats, external clients: a contract untested, or
    asserted only against a stub this repo also wrote.
11. **Extra checks** — every `extra_checks` question, asked of the suite. If the section is
    absent, say so rather than inventing questions.

Evidence: every finding cites `path:line` you actually read, snippet captured at read time — for a
missing test, the code that lacks one. Never report on a file you didn't open, and never invent a
test name, framework version or coverage number; what you can't confirm is written at low
confidence with the one thing that would settle it. A clean scope produces no files, and saying so
is correct — never pad the folder.

## Step 3 — Ask only what the repo can't answer

Test intent often isn't in the tests. Ask me in one round — either/ors in one AskUserQuestion call,
open questions as plain text alongside — only where the answer changes a finding. Don't ask what
the repo answers. If nothing is worth asking, skip this step.

## Step 4 — Write one file per finding

Create the `reviews` folder if absent. Write each finding immediately as `test-NN.md` — `NN`
zero-padded, numbered in the order fixes should be applied. An `approach` finding that changes a
convention comes before the tests that would be written under it.

One file is one fix. Merge findings that share a root cause, repeat one mechanical change across
sites, or would leave the suite inconsistent if fixed apart; split anything that would touch
unrelated files. **Two findings must never ask for conflicting edits to the same lines** — merge
them, or state the dependency in `Apply`, since the whole folder may be handed over at once.

````markdown
# NN — <title>

- **Category:** approach | missing test | weak test | remove/replace
- **Severity:** critical | high | medium | low
- **Confidence:** high | medium | low
- **Tier:** the tier from `tests` this belongs in
- **Sites:** `path:line`, `path:line-range` — for a missing test, the code that has none
- **Apply:** independent | after NN | together with NN

## What's wrong
The suite as it stands today, quoted from it, for someone who has never opened this repo. For a
missing test: the behaviour that ships with nothing watching it.

## Why it matters
The concrete consequence — the bug that ships undetected, the flake that costs a rerun and the
trust that goes with it, the refactor this makes unsafe.

## Fix
An instruction, not a description of one: the exact file to create or edit, in the tier and naming
convention this repo uses; the runner it runs under; the fixtures, factories and helpers to reuse,
by path; the scenarios to cover and the behaviour each must show. **Describe behaviour and
expected outcome; leave the test code to whoever applies this**. Never weaken a test to make
it pass.

## Don't
What must not change — production code, unrelated tests, the runner config, formatting churn — and
any adjacent temptation, naming the finding that covers it if one does. Where the fix genuinely
needs a seam in the production code, name that one seam and nothing past it.

## Verify
The exact `commands` for this tier and what passing looks like. For a new test, also what it must
be seen to do before it counts: fail, for the stated reason, when the behaviour it guards is
broken.

## If the suite doesn't match
This finding was written against the state above. If what you find contradicts it, stop and report
the contradiction rather than improvising around it.
````

## Step 5 — Index last

Write `test-00-index.md` in the same folder: scope, commit and date; exclusions and why; counts by
severity and by category; the findings table in apply order; the three to five systemic patterns
worth fixing at the root rather than per instance — one that is itself actionable gets its own
finding file, referenced here; and any step 3 question I left unanswered.

The index is for me, not the fixer: **no finding file may depend on it** — each stands alone with
the index absent.

## Step 6 — Report

In chat, under 200 words: counts by severity and category, the three findings to fix first and why
those, anything left unreviewed or dropped as infeasible, any fact worth adding to
`.claude/project.md`, and confirmation that nothing outside the review folder was touched.

## Where this skill stops

It writes findings about tests; it doesn't write them, and it doesn't touch the code under them.
A bug it turns up belongs to `/craftstep:review-code`.
