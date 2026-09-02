---
name: review-code
description: Review a scope I name — uncommitted changes, a folder, a branch — and write one self-contained fix prompt per finding into the review folder. Never fixes what it finds and excludes tests.
argument-hint: [what to review — e.g. uncommitted changes, src/api, since main; blank asks]
allowed-tools: Read, Grep, Glob, Bash(git:*), Bash(mkdir:*), Write, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are reviewing code. Scope for this run (blank → step 1 settles it with me): $ARGUMENTS

Every finding lands on disk as its own file, written for a fresh session that has no memory of
this review and will be handed **one such file, or the whole folder, and nothing else**.

**This skill only writes inside `reviews` folder.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording,
propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (subagents, non-root
   packages in a monorepo).
2. **`.claude/project.md`**, sections `reviews`, `commands`, `tests`, `docs`, `reuse`. A missing
   file, section or key are all normal.
3. **The repo itself** — manifest, CI, lint/formatter/type config, test and docs layout.

What each fact does here, and how to infer it at source 3:

- **reviews** — where findings are written. Infer nothing; default `docs/code-review/`.
- **commands** — lint, types, build, each test tier; every finding names the command that proves
  its fix. Infer from manifest scripts and CI.
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
factories, mocks and snapshots. Out too: generated, vendored and lockfile paths, and anything not
readable as text. Print the set before reading it: file count, exclusions and why, and the commit or
working-tree state under review. Above 150 files, stop and offer to narrow rather than quietly doing
a shallow pass.

**For a diff scope the unit of review is the change, not the file.** Read whole files for context;
report only what the change introduced or broke.

**The folder holds one run at a time.** Before the read pass, check `reviews` for files under
this run's `code-` prefix. Say what is there and wait while I clear it. This skill deletes nothing
itself, and the other prefix is none of its business.

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
at low confidence with the one thing that would settle it. A clean scope produces no files, and
saying so is the correct outcome — never pad the folder.

## Step 3 — Write one file per finding

Create the `reviews` folder if absent. Write each finding immediately as `code-NN.md`
— `NN` zero-padded, numbered in the order fixes should be applied.

One file is one fix. Merge findings that share a root cause, repeat one mechanical change across
sites, or would leave the code inconsistent if fixed apart. **Two findings must never ask
for conflicting edits to the same lines** — merge them, or state the dependency in `Apply`, since
the whole folder may be handed over at once.

````markdown
# NN — <title>

- **Category:** correctness | performance | security | maintainability | architecture | documentation
- **Severity:** critical | high | medium | low
- **Confidence:** high | medium | low
- **Sites:** `path:line`, `path:line-range`
- **Apply:** independent | after NN | together with NN

## What's wrong
The behaviour today, quoted from the code, for someone who has never opened this repo.

## Why it matters
The concrete consequence — the bug that fires, the cost paid, the change this makes harder.

## Fix
An instruction, not a description of one: name the files and symbols, state the behaviour that
must hold afterwards, specific enough that two engineers would produce substantially the same
diff. Describe the change; leave the code to whoever applies it.

## Don't
What must not change — public API, passing tests, unrelated files, formatting churn — and any
adjacent temptation, naming the finding that covers it if one does.

## Verify
The exact `commands` that prove it and what passing looks like, plus any regression test that must
exist, in the tier and location `tests` gives.

## If the code doesn't match
This finding was written against the state above. If what you find contradicts it, stop and report
the contradiction rather than improvising around it.
````

## Step 4 — Index last

Write `code-00-index.md` in the same folder: scope, commit and date; exclusions and why; counts by
severity and by category; the findings table in apply order; and the three to five systemic
patterns worth fixing at the root rather than per instance — one that is itself actionable gets
its own finding file, referenced here.

The index is for me, not the fixer. **No finding file may depend on it**; each stands alone with
the index absent.

## Step 5 — Report

In chat, under 200 words: counts by severity, the three findings to fix first and why those,
anything the scope left unreviewed, and any fact worth adding to `.claude/project.md`.

## Where this skill stops

It writes findings; it doesn't decide what gets fixed.
