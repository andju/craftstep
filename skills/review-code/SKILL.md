---
name: review-code
description: Review a scope I name — uncommitted changes, a folder, a branch — and write the findings as one checklist document into the repo root, for /craftstep:fix to work through. Never fixes what it finds and excludes tests.
argument-hint: [what to review — e.g. uncommitted changes, src/api, since main; blank asks]
allowed-tools: Read, Grep, Glob, Bash(git:*), Write, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are reviewing code. Scope for this run (blank → step 1 settles it with me): $ARGUMENTS

The findings land on disk as one document, `code-findings.md`: read by me, and worked through one
finding at a time by `/craftstep:fix`, in a later session that has no memory of this review.

**This skill writes one file: `code-findings.md` at the repo root.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording,
propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (subagents, non-root
   packages in a monorepo).
2. **`.claude/project.md`**, sections `commands`, `tests`, `docs`, `reuse`. A missing file,
   section or key are all normal.
3. **The repo itself** — manifest, CI, lint/formatter/type config, test and docs layout.

What each fact does here, and how to infer it at source 3:

- **commands** — lint, formatter, types; their config is a standard step 2 enforces. Infer from
  manifest scripts and CI.
- **tests** — how to identify test files, that drop out of the review set wherever they sit: infer
  from the runner's config and the language's convention.
- **docs** — which docs are user-facing, for findings about docs the code has outgrown. Infer from
  the docs directory; if none is user-facing, the project has no user docs.
- **reuse** — directories of shared units. Duplication is a finding only when something reusable
  already exists; check here. Infer from where shared code lives.

Where two sources settle the same fact differently, the higher one wins. List every fact in one short
block, each with the source that settled it.

## Step 1 — Settle the scope, then print the review set

Turn $ARGUMENTS into an explicit file list *before* reading any of it:

- **Working tree** ("uncommitted", "staged", "what I just wrote") — `git status --porcelain`, plus
  `git diff` and `git diff --cached`.
- **A range** ("since main", "this branch", "the last three commits") — resolve the base against
  the repo's actual default branch, not an assumed `main`, then `git diff <base>...HEAD`.
- **A path** — glob it, directories included.
- **A subsystem in prose** ("the auth flow") — grep its entry points, follow imports one level,
  list what you landed on, and say the boundary is your guess, not mine.
- **Empty** — AskUserQuestion, offering the options above this repo actually has, each with its
  file count so the choice is grounded.

Remove **test files, matched by the `tests` patterns anywhere in the tree**, fixtures, test
factories, mocks and snapshots. Out too: generated, vendored and lockfile paths, the review documents
`code-findings.md` and `test-findings.md`, and anything not readable as text. Print the set
before reading it: file count, exclusions and why, and the commit or working-tree state under
review. Above 150 files, stop and offer to narrow rather than quietly doing
a shallow pass.

**For a diff scope the unit of review is the change, not the file.** Read whole files for context;
report only what the change introduced or broke.

**The repo root holds one run at a time.** Before the read pass, check the repo root
(`git rev-parse --show-toplevel`) for a `code-findings.md`. If one is there, say so and wait while
I clear it. This skill deletes nothing itself, and `test-findings.md` is none of its business.

## Step 2 — One pass, all lenses

Ground yourself first: Read the lint, formatter and type config `commands` points at; a rule this
project configured is a standard you enforce, inline suppressions included.

Walk the set once, applying every lens per file rather than re-scanning per lens. Never re-read a
file. Capture the line numbers and snippets you need at read time. Priority order, which is also
how findings rank:

1. **Correctness** —  logic errors, wrong operators and boundaries, unhandled null, stale or
   shared-mutable state, unawaited or mis-sequenced async, races, swallowed exceptions, missing
   timeouts, resources not released on error paths, broken assumptions about where code runs
   (server vs client, build vs request, before vs after hydration).
2. **Performance** — repeated work in a hot path, unoptimized loops/data structures, sequential
   awaits that could be concurrent, N+1 and unbounded loads, missing memoization where the mechanism
   is visible, work on the wrong side of a boundary.
3. **Security** — Not a security audit - consider only severe: unvalidated input, exposed or
   hardcoded secrets, an entry point missing an authorization check, path traversal, unsafe
   deserialization, permissive CORS.
4. **Maintainability** — duplicated logic (e.g. where a `reuse` unit already covers it), dead
   code, magic values, escape-hatch types (`any` and equivalents, unchecked casts), overly complex
   components, poor separation of concerns, inconsistent patterns across similar files, units too
   large or entangled to test.
5. **Architecture** — coupling that shouldn't exist, misuse of the framework's own conventions,
   business logic in a layer that shouldn't hold it.
6. **Documentation and standards** — missing or wrong doc comments, user-facing docs (per `docs`)
   the code has outgrown, violations of the language's accepted conventions, configured-rule
   violations including suppressed ones.

Evidence: every finding cites `path:line` you actually read, snippet captured as you read it.
Report nothing about a file you didn't open. A finding you can't confirm from the code is written
at low confidence with the one thing that would settle it. A clean scope produces an empty
checklist, and saying so is the correct outcome — never pad it.

## Step 3 — Write the findings document

Write `code-findings.md` at the repo root. Number the findings `NNN`, zero-padded, in
the order fixes should be applied.

One finding is one fix. Merge findings that share a root cause, repeat one mechanical change across
sites, or would leave the code inconsistent if fixed apart. **Two findings must never point
opposite ways on the same lines** — merge them, or state the dependency in `Apply`.

**Write what only this review knows, and stop there.** The fixer confirms each finding against the
code and decides the edit itself, against the code as it is by then — earlier fixes will have moved
it. What it can't get from the code is what this pass saw across the whole set: every site, how
findings depend on each other, which way a fix should go and what it must stay consistent with. The
edit, what it mustn't touch and the commands that prove it are the fixer's; don't write them.

**The fixer reads one section, never the whole document.** Each finding stands alone: no section
leans on another, or on the systemic patterns, except by number. The checklist is the fixer's to
tick off — leave every box empty.

A long document goes in parts: the header and checklist first, then the findings, appended in order.

````markdown
# Code review — <scope> · <commit, or working tree> · <date>

Scope; exclusions and why; counts by severity and by category.

## Checklist

- [ ] **001** · <severity> · <category> — <title>
- [ ] **002** · <severity> · <category> · low confidence — <title>

## Systemic patterns

The three to five patterns worth fixing at the root rather than per instance. One that is itself
actionable is a finding of its own, named here by number.

## Findings

### 001 — <title>

- **Category:** correctness | performance | security | maintainability | architecture | documentation
- **Severity:** critical | high | medium | low
- **Confidence:** high | medium | low
- **Sites:** `path:line`, `path:line-range`
- **Apply:** independent | after NN | together with NN

**What's wrong.** The behaviour today, quoted from the code, for someone who has never opened this
repo.

**Why it matters.** The concrete consequence — the bug that fires, the cost paid, the change this
makes harder.

**Direction.** Which way the fix goes, in a sentence or two: the unit to route it through, the
`reuse` unit that already covers it, the adjacent change it must stay consistent with. Not the edit.

**Check first.** The one place that would make this fine, if anything does — the caller that may
already validate, the guard, the type, the config. At low confidence: the one thing that would
settle it.
````

## Step 4 — Report

In chat, under 200 words: counts by severity, the three findings to fix first and why those,
anything the scope left unreviewed, and any fact worth adding to `.claude/project.md`.

## Where this skill stops

It writes findings; it doesn't decide what gets fixed, or how — that is `/craftstep:fix`'s call,
made against the code as it is when the fix happens.
