---
name: setup
description: Set up `.claude/project.md` for this repo — work out each section from the repo, show the evidence, and write only the sections I accept. Never edits CLAUDE.md and never writes code.
argument-hint: [optional — which sections to (re)do, e.g. commands tests; blank does all seven]
allowed-tools: Read, Grep, Glob, Bash(git:*), Bash(mkdir:*), Write, Edit, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are setting up the project facts every other craftstep skill reads. Sections in scope for
this run (if that's empty, all of them): $ARGUMENTS

**This skill writes exactly one file: `.claude/project.md`.** It never edits CLAUDE.md, never
touches code or config, and never creates the folders a section names — a path is recorded
here, not built.

The file is a plain sequence of `## ` sections and nothing else. It exists to settle what the
repo leaves ambiguous. A section that only restates what a skill would infer anyway is a second
copy to keep current, so the default outcome for a section is *omit it*, and a proposal has to
earn its place: it differs from the default, it's expensive to rediscover, or the repo genuinely
doesn't say.

## Step 0 — Where the file goes, and what already settles things

Run `git rev-parse --show-toplevel` for the repo root; the file lands at
`<root>/.claude/project.md`.

Then read, in this order, and let each one shrink the work:

1. **The repo's CLAUDE.md** — root, plus a package-level one if we're inside a package. Skills
   read it before `.claude/project.md`, so any fact stated there is already settled. Drop that
   section from this run and say which sections CLAUDE.md covered.
2. **An existing `.claude/project.md`** — normal on a re-run. Sections already in it are
   decided: leave them alone. Propose only the missing ones, plus any section where the repo
   now plainly contradicts the file — and for those, show both values and let me choose. A
   section already in the file that CLAUDE.md also settles is the copy that loses: name it,
   show both values, and offer to drop it. Never remove one without asking.
3. **Monorepo check.** If the root holds several packages, this file covers the repo. Where a
   package differs materially, record the difference as a line inside the relevant section
   rather than proposing a second file.

## Step 1 — Work out each section from the repo

Every proposal cites its evidence: a file path, a manifest script name, a CI job. Where you
find no evidence, the proposal is the documented default (`plans`, `decisions`, `reviews`) or
"leave unset" (`commands`, `tests`, `docs`, `reuse`) — never a plausible-looking guess.
**Don't run any command you find**, and don't create anything to make a path exist.

- **plans** — the single file specs get written to. Look for an existing `SPEC.md`, a specs
  folder, or a spec path named in CLAUDE.md or `.gitignore`. Default `SPEC.md` at the root.
- **decisions** — where design and decision records live. Glob for `adr`, `decisions`,
  `rfc`, `docs/architecture`. Default `docs/architecture/decisions/`.
- **reviews** — the folder review findings are written to, one file per finding. Look for an
  existing review folder and for an ignore-rule covering one. Default `docs/code-review/`.
- **commands** — how to run tests, lint, build, typecheck. Take them from the package manifest
  scripts, `Makefile`/`justfile`, or the CI workflow, and prefer what CI actually runs over
  what the manifest merely defines. One line each, with a trailing comment naming what it is.
  Omit a command you can't point at.
- **tests** — two halves: the tiers that exist and what belongs in each, and how test files are
  named (this is what lets a skill spot co-located tests without a full glob). Find the test
  locations, read one or two tests from each tier, and check CI for how each tier is run.
  Describe a tier by the boundary it crosses, not by its folder name.
- **docs** — which docs are user-facing. Find the docs tree and work out which part users read;
  an internal half is worth naming explicitly so review skills don't count changes there. If
  the only user-facing document is the README, say that. If there are none, the value is `none`.
- **reuse** — the directories holding shared units to check before writing anything new:
  shared components, hooks, service clients, helpers, and shared test fixtures and factories.
  Infer from where shared code actually lives in this stack, and check what existing code
  imports from most, rather than from directory names alone.

## Step 2 — Show me the draft before asking anything

Print two things, in this order:

1. **One line per section**: the proposed value, where it came from, whether you read it or
   inferred it, and whether it just matches the documented default.
2. **The file as it would land**, in schema order (`plans`, `decisions`, `reviews`, `commands`,
   `tests`, `docs`, `reuse`), so I'm accepting real text rather than a description of it.

The draft is file content, not part of your message: print it **inside a single fenced block,
fenced with four backticks**.

Say plainly which proposals you're least sure of. A command you never ran is unproven, and a
tier boundary read off two files is a sample.

Then ask me with `AskUserQuestion` what to do with the draft as a whole, before any
section-by-section questions:

- **Accept the whole draft** — skip Step 3 and write it as printed.
- **Go through it section by section** — continue to Step 3.
- **Discard it** — write nothing and stop.

## Step 3 — Get each section accepted

Ask about every section still in scope — one question per section, batched four to an
`AskUserQuestion` call. Options:

- **Accept as proposed.**
- **A second candidate**, but only where the repo actually offered one (two plausible docs
  trees, a manifest script and a CI job that disagree). Don't manufacture an alternative.
- **Omit the section.** Recommend this one when the value only matches the documented default,
  restates what any skill would infer in seconds, or duplicates something CLAUDE.md already
  states under a heading of its own.

I can always type my own value instead of picking, so keep options concrete and short.

Alongside the questions, put in plain text anything inference can't reach — whether a tier is
still in use, which half of the docs tree users read, a command whose name doesn't say what it
covers. Never ask about a section CLAUDE.md settles or the existing file already holds.

## Step 4 — Write only what I accepted

`mkdir -p .claude` if needed, then write the accepted sections in schema order:

- A single path → inline code on its own line.
- `commands` → one `bash` fence, one command per line, trailing `#` comment naming each.
- `tests`, `docs`, `reuse` → bullets, each naming the path and what belongs there.
- An HTML comment only where a value would puzzle someone a year from now. Don't annotate the
  obvious.

**Updating an existing file:** use Edit and insert into schema order. Sections I didn't touch
this run come out byte-identical — same wording, same comments, same order.

**Accepting nothing is a valid outcome.** If CLAUDE.md covers everything and the rest are
defaults, don't create the file; say so and stop.

## Where this skill stops

Report the path written, which sections landed, and which were skipped and why (covered by
CLAUDE.md, matched the default, declined). Name any command recorded but never run, so I know
what's unproven.

If the run turned up a fact that belongs in CLAUDE.md rather than here — something every
session should have in context — propose the wording for me to apply. Don't apply it yourself.
