---
name: implement
description: Implement an agreed spec — one work item at a time, verifying each, stopping when the codebase contradicts the plan.
argument-hint: [optional — which work items; blank implements all of it]
allowed-tools: Read, Grep, Glob, Bash, AskUserQuestion, Agent(spec-implementer), SendMessage
disable-model-invocation: true
user-invocable: true
---

We are implementing a spec that `/craftstep:plan` wrote and I reviewed. Scope for this run
(if that's empty, implement the whole spec): $ARGUMENTS

You are the orchestrator, running in my session, and **every step in this file is yours**.
The implementation is not: the spec's work items, and the tests and docs that go with them,
belong to the `spec-implementer` subagent. They are always delegated.

**Stopping rule.** Anywhere below, "stop and ask" means: report what you found, ask me, and
wait. Never improvise around it, re-scope an item to dodge it, or duplicate code to route
around it.

## Step 0 — Load this project's conventions, then check the spec is dispatchable

Resolve each fact independently, taking the first source that settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it
   isn't. Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `plans`, `commands`, `tests`, `docs`. A missing
   file, a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing test and docs layout.

What each fact does here, who uses it, and how to infer it at source 3:

- **plans** — the file the spec lives in. Default to `SPEC.md` at the repo root.
- **commands** — each test tier plus the project's own gate: lint, types, build. **The gate
  is yours, at Step 2.** The spec names an exact command per work item and per test, so the
  tier commands go down only as a fallback for where it failed to. Infer from the package
  manifest's scripts and the CI config.
- **tests** — where a file for a given tier goes and what shape it takes. The spec names the
  tier but not the directory or the conventions, so **this is the one fact the subagent
  cannot read out of the spec.** Infer by globbing test locations and reading one or two
  existing tests from the tier being written into.
- **docs** — which docs are user-facing. **Yours**, for acceptance criteria that turn on
  documentation; the spec names the files the subagent touches, so this isn't dispatched.
  Infer from the docs directory; if none are, treat this as a project with no user docs.

List every fact you inferred rather than read, in one short block, before continuing.

**Then check the spec, before you dispatch.** These are preconditions of dispatching, not
work items: failing one here is a stop in front of me rather than a wasted round-trip.

- If nothing is in the `plans` path, stop and say so. Don't write a spec yourself, and don't
  implement from my request alone.
- Confirm **$ARGUMENTS is covered by the spec.** Don't dispatch work the spec doesn't
  contain.
- Read the **Acceptance criteria** and the **Work items** now, and note which items are
  already marked — an item written `1. [x]` rather than `1. [ ]` is one the spec already
  counts as finished, so any of those mean this is a resumed run: say which. Step 2 proves
  the criteria from this file rather than from the subagent's report, and what you read here
  is what it proves them against — reading them before the dispatch is what makes them
  *the criteria I am holding you to*.

## Step 1 — Dispatch the implementation

Dispatch **one** `spec-implementer` subagent, `run_in_background: false` — Step 2 depends on
its result and nothing else can usefully happen meanwhile.

**The spec is the interface.** Its steps are in its own file and it reads the spec from disk
itself, so the prompt carries only what neither of those holds:

- **The spec path** and **the scope** from $ARGUMENTS.
- **The `tests` fact** — where a file of each tier goes and what shape it takes. Mark it read
  or inferred.
- **The tier commands from `commands`, named as a fallback only** — for a work item or a test
  the spec failed to give an exact command for. Say that is all they are.

Pass nothing else.

**A BLOCKED report comes back to you, not to me.** Read it, decide which of two things it is,
and say which:

- **The spec already settles it.** Quote the spec's own wording back and resume the same
  subagent so it keeps its context. Never settle it from your own reading of the code — if
  the spec doesn't answer it in its own words, it isn't this case.
- **It's a decision only I can take.** Stop and ask, carrying the subagent's question as it
  was written, plus what it had already finished.

## Step 2 — Prove it, then report

Run the project's full gate: every command in the `commands` fact that applies to what
changed. Then take the **Acceptance criteria** you read at Step 0 one at a time and say what
makes each true — the behaviour you observed, or the test that covers it. A criterion you
can't demonstrate is not met, however finished the code looks. The criteria include what must
*not* change; those get evidence too.

**On a partial run, scope the proof.** Where $ARGUMENTS or the spec's markers left work items
outside this run, a criterion resting on one of them is outstanding, not failed: name it, name
the items it waits on, and don't send it to the fix loop. Every criterion the dispatched items
do reach still gets its evidence.

### The fix loop — a standing rule, not a one-time step

When the gate or a criterion fails, you may send it back. These hold for every round:

- **At most two rounds.** Still failing after the second, stop and ask.
- **Resume the same `spec-implementer`** so it keeps its context.
- **Hand it the failing criteria and the actual command output** — the command, its exit
  status, the failing lines, quoted. Not your paraphrase of them.
- **The brief carries what the previous round tried and how it failed.** A round that would
  repeat the last one is a stop, not a third attempt at the same idea.
- **After every round, re-run the whole criteria set**, not just what failed. A fix that
  satisfies one criterion by breaking another is exactly what this catches, and the "must not
  change" criteria are where it lands.
- **Never accept "fixed" from the subagent's summary.** Re-derive it: run the commands
  yourself and read the output.

### The report

Yours to write:

- Work items done, and any left.
- Every acceptance criterion, with its evidence — or, where this run was partial, the
  outstanding items it waits on.
- Commands run and their outcome, **quoting the gate output verbatim** rather than
  summarizing it — failures included, and which criteria needed a second round.
- Deviations: each one the subagent reported, next to the spec's own wording. A command the
  spec failed to name is one of them.
- Anything blocked, and the decision from me that would unblock it.

## Where this skill stops

This skill implements the spec. It does not re-plan one: a spec that turns out to be wrong
goes back to `/craftstep:plan`, with the contradiction found. It doesn't reach past the work
items, it changes nothing in the spec but the progress markers, and it doesn't commit unless
I ask.

It writes nothing to disk: the report is prose to me, and the spec's progress markers are the
subagent's to set. It doesn't answer the subagent's questions from its own reading of the
code, doesn't take the subagent's word for a passing gate, and doesn't run a third fix round
— after two, the next move is mine.
