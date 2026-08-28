---
name: implement
description: Implement an agreed spec — one work item at a time, verifying each, stopping when the codebase contradicts the plan.
argument-hint: [optional — which work items, or which spec; blank implements all of it]
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are implementing a spec that `/craftstep:plan` wrote and I reviewed. Scope for this run
(if that's empty, implement the whole spec): $ARGUMENTS

**Stopping rule.** Anywhere below, "stop and ask" means: report what you found and wait for
me. Never improvise around it, re-scope an item to dodge it, or duplicate code to route
around it.

**Never edit `.claude/project.md` or CLAUDE.md while this skill runs.** If the run turned up
a fact worth recording — a test tier you had to infer, a command the spec guessed at —
propose it as a `.claude/project.md` addition at the end for me to apply.

## Step 0 — Load this project's conventions

The facts below drive this skill. Resolve each one independently, taking the first source
that settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it
   isn't. Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `plans`, `commands`, `tests`, `docs`,
   `extra_checks`. A missing file, a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing test and docs layout.

What each fact does here, and how to infer it at source 3:

- **plans** — where the spec lives. Infer from an existing plans or specs convention;
  default to `SPEC.md` at the repo root.
- **commands** — each test tier plus the project's own gate: lint, types, build. The spec
  names a verify command per work item; this fact is the fallback when it doesn't, and what
  you run at the end. Infer from the package manifest's scripts and the CI config.
- **tests** — where a file for a given tier goes and what shape it takes. Infer by globbing
  test locations and reading one or two existing tests from the tier you're writing into.
- **docs** — which docs are user-facing. Infer from the docs directory; if none are, treat
  this as a project with no user docs.
- **extra_checks** — the cross-cutting questions every change is held against. The spec
  answered them against a design; you confirm them against the code you write. Infer nothing.

List every fact you inferred rather than read, in one short block, before continuing.

## Step 1 — Find the spec and read all of it

The `plans` fact gives either a fixed path (`SPEC.md`) or a per-feature pattern
(`docs/specs/<feature-slug>.md`). For a pattern, glob it: $ARGUMENTS may name the spec; if it
doesn't and more than one candidate exists, ask with AskUserQuestion rather than taking the
most recently modified. If there's no spec at that path, stop and say so. Don't write one
yourself, and don't implement from my request alone.

Read the spec in full before touching anything, including the sections that read as context:
**Rejected alternatives** and **Reuse** are routes already turned down — don't re-litigate
them; **When this plan is wrong** is what step 2 checks.

Then confirm two things, stopping and asking on either:

- **The spec is about this feature.** Under a single-file `plans` convention it may be left
  over from a different one — check its **Feature** section.
- **$ARGUMENTS is covered by it.** Don't improvise work the spec doesn't contain.

A spec that doesn't use the `/craftstep:plan` headings still works if it has an ordered list
of work items — use what it has, and say which sections are missing. With no ordered work
items at all, stop: there is nothing to sequence or verify against.

## Step 2 — Check the plan against the code before changing any of it

A spec is written from a reading of the codebase that may be weeks old. Before the first
edit, confirm what the plan stands on, and report in one short block:

- Every file, symbol, directory and dependency the **Work items** name — does it exist, and
  does its interface still match what the spec assumes? Where an item names an existing unit
  to build on, does the extension it describes still fit?
- The spots **When this plan is wrong** flags as least confident. That section was written
  to be checked here.

If reality contradicts the spec, stop and ask — that is the spec's own instruction, and it
outranks your reading of what it meant. I'll either revise the spec with `/craftstep:plan`
or tell you how to proceed.

## Step 3 — Work the items in order

The **Work items** are ordered, and each is verifiable alone. One at a time, in order:

1. Restate the item's scope as a checklist first: the files it touches, what it must not
   touch, its finish condition.
2. Implement that item and nothing else.
3. Run the item's own verify command — the exact one the spec gives.
4. Mark the item done in the spec.

**Between items the repo may legitimately be broken.** The order is a dependency order:
earlier items land migrations, deletions and interface changes that later items consume. So
run the item's own command, not the project's full gate, until step 5. A failure whose cause
is an item you haven't reached is expected — record it and move on; don't fix it, and don't
pull that item forward. A failure caused by your own change, or in a test that passed before
you started, is yours.

**Small corrections are fine** — a helper name the spec didn't fix, an import it didn't
mention, a branch its prose implies. Note each one in the summary. Anything that changes the
approach, touches a file outside the item's list, or takes a decision the spec doesn't make
(a public name, a signature, a migration rule, an error-handling policy, a new dependency)
is a stop. So is an existing file that turns out not to take the extension an item names.

**Never mark an item done behind a stub, a TODO, a skipped test or a widened type.** Half an
item reported honestly beats a whole one guessed at.

If items are already marked when you start, this is a resumed run. Skip those, say which,
and start at the first unmarked one. Step 2 still applies.

## Step 4 — Tests, docs and the extra checks

Each of these spec sections is an obligation:

- **Tests** — write the tests it lists, each in the tier it names, in that tier's location
  from the `tests` fact. Where it marked a command as inferred rather than read, run it and
  say whether the guess held.
- **Docs** — make the user-facing changes it names. If it says the change is internal-only,
  or that the project has no user docs, say that in the summary.
- **Extra checks** — confirm every answer still holds against the code you actually wrote,
  and flag any that flipped. If the spec says the section was absent, repeat that here.

Where a work item's own scope covers its tests or docs, do them inside that item rather than
saving them for the end.

## Step 5 — Prove it, then report

Run the project's full gate: every command in the `commands` fact that applies to what
changed. Then take the **Acceptance criteria** one at a time and say what makes each true —
the behaviour you observed, or the test that covers it. A criterion you can't demonstrate is
not met, however finished the code looks. The criteria include what must *not* change; those
get evidence too.

Report:

- Work items done, and any left.
- Every acceptance criterion, with its evidence.
- Commands run and their outcome, quoting failures rather than summarizing them.
- Deviations: each small correction from step 3, next to the spec's own wording.
- Anything blocked, and the decision from me that would unblock it.

## Where this skill stops

This skill implements the spec. It does not re-plan one: a spec that turns out to be wrong
goes back to `/craftstep:plan`, with the contradiction you found. It doesn't reach past the
work items, it changes nothing in the spec but the progress markers, and it doesn't commit
unless I ask.
