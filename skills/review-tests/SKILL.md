---
name: review-tests
description: Audit the tests for a scope I name and write the findings as one checklist document into the repo root, for /craftstep:fix to work through. Never writes or repairs a test itself.
argument-hint: [what to review — e.g. the whole suite, tests/api, the auth flow, since main; blank asks]
allowed-tools: Read, Grep, Glob, Bash(git:*), Write, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are reviewing tests. Scope for this run (blank → step 1 settles it with me): $ARGUMENTS

The findings land on disk as one document, `test-findings.md`: read by me, and worked through one
finding at a time by `/craftstep:fix`, in a later session that has no memory of this review.

**This skill writes one file: `test-findings.md` at the repo root.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording,
propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (subagents, non-root
   packages in a monorepo).
2. **`.claude/project.md`**, sections `commands`, `tests`, `reuse`. A missing file, section
   or key are all normal.
3. **The repo itself** — manifest, runner and coverage config, CI workflows, test layout.

What each fact does here, and how to infer it at source 3:

- **tests** — the tiers, what belongs in each, file naming. Both the review set and the tier a
  missing test belongs in. Infer from runner config, CI and language convention; confirm by
  globbing.
- **commands** — how each tier is run, for the enforcement lens. You never run them here. Infer
  from manifest scripts and CI.
- **reuse** — shared units, including fixture, factory and helper directories. Duplicated setup is
  a finding only when a reusable unit exists. Infer from where tests import their helpers.

Where two sources settle the same fact differently, the higher one wins. List every fact in one short
block, each with the source that settled it.

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
- **The whole suite** — every file matching the `tests` patterns, wherever it sits; the covered
  half is what those tests import. That bounds the missing-test lenses to code some test already
  reaches — say so, because a unit no test imports at all is exactly what they would otherwise
  be looking for.
- **Empty** — AskUserQuestion, offering the options above this repo actually has, each with its
  file count so the choice is grounded.

Also in scope unless I say otherwise: runner and coverage config, the CI workflows that run tests,
shared fixtures and factories, any coverage report already in the tree. Out: generated, vendored
and lockfile paths, the review documents `code-findings.md` and `test-findings.md`, and anything
not readable as text. Print all of it before reading it — both
halves, config, exclusions and why, and the commit or working-tree state under review. Above 150
files, stop and offer to narrow; a suite you skimmed produces confident-sounding noise.

**The repo root holds one run at a time.** Before the read pass, check the repo root
(`git rev-parse --show-toplevel`) for a `test-findings.md`. If one is there, say so and wait while
I clear it. This skill deletes nothing itself, and `code-findings.md` is none of its business.

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

Evidence: every finding cites `path:line` you actually read, snippet captured at read time — for a
missing test, the code that lacks one. Never report on a file you didn't open, and never invent a
test name, framework version or coverage number; what you can't confirm is written at low
confidence with the one thing that would settle it. A clean scope produces an empty checklist, and
saying so is correct — never pad it.

## Step 3 — Ask only what the repo can't answer

Test intent often isn't in the tests. Ask me in one round — either/ors in one AskUserQuestion call,
open questions as plain text alongside — only where the answer changes a finding. Don't ask what
the repo answers. If nothing is worth asking, skip this step.

## Step 4 — Write the findings document

Write `test-findings.md` at the repo root. Number the findings `NNN`, zero-padded, in
the order fixes should be applied. An `approach` finding that changes a convention comes before
the tests that would be written under it.

One finding is one fix. Merge findings that share a root cause, repeat one mechanical change across
sites, or would leave the suite inconsistent if fixed apart; split anything that would touch
unrelated files. **Two findings must never point opposite ways on the same lines** — merge them, or
state the dependency in `Apply`.

**Write what only this review knows, and stop there.** The fixer confirms each finding against the
code and decides the change itself, against the suite as it is by then — earlier fixes will have
moved it. What it can't get from the code is what this pass saw across the whole set: every site,
how findings depend on each other, which scenarios matter and which way a fix should go. The test
code, what it mustn't touch and the commands that prove it are the fixer's; don't write them.

**The fixer reads one section, never the whole document.** Each finding stands alone: no section
leans on another, or on the systemic patterns, except by number. The checklist is the fixer's to
tick off — leave every box empty.

A long document goes in parts: the header and checklist first, then the findings, appended in order.

````markdown
# Test review — <scope> · <commit, or working tree> · <date>

Scope; exclusions and why; counts by severity and by category; any step 3 question I left
unanswered.

## Checklist

- [ ] **001** · <severity> · <category> — <title>
- [ ] **002** · <severity> · <category> · low confidence — <title>

## Systemic patterns

The three to five patterns worth fixing at the root rather than per instance. One that is itself
actionable is a finding of its own, named here by number.

## Findings

### 001 — <title>

- **Category:** approach | missing test | weak test | remove/replace
- **Severity:** critical | high | medium | low
- **Confidence:** high | medium | low
- **Tier:** the tier from `tests` this belongs in
- **Sites:** `path:line`, `path:line-range` — for a missing test, the code that has none
- **Apply:** independent | after NN | together with NN

**What's wrong.** The suite as it stands today, quoted from it, for someone who has never opened
this repo. For a missing test: the behaviour that ships with nothing watching it.

**Why it matters.** The concrete consequence — the bug that ships undetected, the flake that costs a
rerun and the trust that goes with it, the refactor this makes unsafe.

**Direction.** Which way the fix goes. For a missing test: the scenarios to cover and the behaviour
each must show, and the fixtures, factories and helpers to build on, by path. Otherwise: what
replaces what's there, and the tests it must stay consistent with. Not the test code.

**Check first.** The one place that would make this fine, if anything does — for a missing test, a
test elsewhere in the tree that may already cover it; for determinism, a setup file the runner
loads that may already fake time or seed randomness. At low confidence: the one thing that would
settle it.
````

## Step 5 — Report

In chat, under 200 words: counts by severity and category, the three findings to fix first and why
those, anything left unreviewed or dropped as infeasible, any fact worth adding to
`.claude/project.md`, and confirmation that nothing besides `test-findings.md` was touched.

## Where this skill stops

It writes findings about tests; it doesn't write them, and it doesn't touch the code under them.
A bug it turns up belongs to `/craftstep:review-code`; how a finding gets fixed is
`/craftstep:fix`'s call.
