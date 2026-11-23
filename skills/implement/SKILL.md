---
name: implement
description: Implement an agreed spec — one work item at a time, verifying each, stopping when the codebase contradicts the plan.
argument-hint: "optional — which work items; blank implements all of it"
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, AskUserQuestion, Agent(spec-implementer), SendMessage
disable-model-invocation: true
user-invocable: true
---

We are implementing a spec that `/craftstep:plan` wrote and I reviewed. Scope for this run
(if that's empty, implement the whole spec): $ARGUMENTS

You are the orchestrator, running in my session, and **every step in this file is yours**.
The implementation is not: the spec's work items, and the tests and docs that go with them,
belong to the `spec-implementer` subagent. They are always delegated. Nothing is committed
unless I ask. **The one file you write yourself is the implementation log** — never the spec,
`.claude/project.md` or CLAUDE.md.

**Stopping rule.** Anywhere below, "stop and ask" means: report what you found, ask me, and
wait. Never improvise around it, re-scope an item to dodge it, or duplicate code to route
around it.

## The implementation log

What I decide during a run has to outlast the session. It goes into a file beside the spec —
the spec's path with `.md` replaced by `.log.md` — which every later run reads. The spec itself
stays as I agreed it.

````markdown
# Implementation log: <the spec's feature title>

Kept by `/craftstep:implement`, cleared with the spec by `/craftstep:wrap-up`. Each decision is
the developer's and amends the spec on the point it names.

## Decisions

1. **<what it settles>** · <YYYY-MM-DD> · item <n>
   <The question as it was put.> → <My answer.> <Where it departs from the spec: the spec's
   wording, quoted, and what holds instead.>

## Findings

- <YYYY-MM-DD> · item <n> — <what the spec got wrong or left unstated, and what holds instead>

## State

<YYYY-MM-DD> — <done | stopped to ask | resumed | fix rounds spent>
- Item <n>: <waiting on: the question | resumed on decision <k> | not started | failing: the command>
````

Write it at three moments, creating it at the first:

- **When you stop and ask** — rewrite **State** first, the question in it, so a session I end
  without answering still leaves the question behind.
- **When I answer** — append the answer to **Decisions**, then rewrite **State** so the item no
  longer waits on the question, both before acting on it. A State still showing an answered
  question is stale. Also log an instruction in $ARGUMENTS as a decision when it goes beyond
  choosing items and the spec doesn't settle it: a constraint, an approval, a departure. Never
  edit a recorded decision; a reversal is a new one that names the one it replaces.
- **Before the report** — append to **Findings** each small correction from this run that a later
  run would otherwise rediscover (a command the spec got wrong, a tool's quirk, a failure expected
  until a later item lands), once you have confirmed it yourself. Then rewrite **State**. A run
  with nothing to record creates no log.

## Step 0 — Load this project's conventions, then check the spec is dispatchable

**Settle `plans` first, and check the spec is there.** If nothing is at that path, stop and say
so before resolving anything else. Don't write a spec yourself, and don't implement from my
request alone.

Then resolve each fact independently, taking the first source that settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it
   isn't. Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `plans`, `commands`, `tests`, `docs`. A missing
   file, a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing test and docs layout.

What each fact does here, who uses it, and how to infer it from the repo:

- **plans** — the file the spec lives in. Default to `.craftstep/spec.md`.
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

List every fact in one short block, each with the source that settled it.

**Then check the spec, before you dispatch.** These are preconditions of dispatching, not
work items: failing one here is a stop in front of me rather than a wasted round-trip.

- Confirm **$ARGUMENTS is covered by the spec.** Don't dispatch work the spec doesn't
  contain.
- Read the **Acceptance criteria** and the **Work items** now, and note which items are
  already marked — an item written `1. [x]` rather than `1. [ ]` is one the spec already
  counts as finished, so any of those mean this is a resumed run: say which. Step 2 proves
  the criteria from this file rather than from the subagent's report, and what you read here
  is what it proves them against — reading them before the dispatch is what makes them
  *the criteria I am holding you to*.
- **Read the implementation log, if there is one**, in full. A title naming another feature
  means a leftover: stop and ask. Say how many decisions and findings it holds and what State
  the last run left; the spec's markers, not State, say which items are done. Its decisions are
  given: never ask about one again, and read the spec, criteria included, as they amend it. If
  State shows an item in scope still waiting on a question, and no later decision answers it,
  stop and ask before you dispatch.

## Step 1 — Dispatch the implementation

Dispatch **one** `spec-implementer` subagent, `run_in_background: false` — Step 2 depends on
its result and nothing else can usefully happen meanwhile.

**The spec is the interface.** Its steps are in its own file and it reads the spec from disk
itself, so the prompt carries only what neither of those holds:

- **The spec path** and **the scope** from $ARGUMENTS.
- **The log path**, when there is a log.
- **The `tests` fact** — where a file of each tier goes and what shape it takes. Mark it read
  or inferred.
- **The tier commands from `commands`, named as a fallback only** — for a work item or a test
  the spec failed to give an exact command for. Say that is all they are.

Pass nothing else.

**A BLOCKED report comes back to you, not to me.** Read it, decide which of two things it is,
and say which:

- **The spec or a logged decision already settles it.** Quote its own wording back and resume
  the same subagent so it keeps its context. Never settle it from your own reading of the code —
  if neither answers it in its own words, it isn't this case.
- **It's a decision only I can take.** Stop and ask, carrying the subagent's question as it
  was written, plus what it had already finished. Once I answer, log the decision, then resume
  the same subagent with it, quoted, under its number in the log.

## Step 2 — Prove it, then report

Run the project's full gate: every command in the `commands` fact that applies to what
changed. Then take the **Acceptance criteria** you read at Step 0 one at a time and say what
makes each true — the behaviour you observed, or the test that covers it. A criterion you
can't demonstrate is not met, however finished the code looks. The criteria include what must
*not* change; those get evidence too. A criterion a logged decision amends is proven as
amended, naming the decision.

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
- Commands run and their outcome. **A failure is quoted verbatim** — the command, its exit
  status, the failing lines — never summarized. A pass is the command and its own summary line
  (`142 passed`), not its full output. And which criteria needed a second round.
- Deviations: each one the subagent reported, next to the spec's own wording. A command the
  spec failed to name is one of them. A departure a logged decision covers is named by that
  decision instead.
- Anything blocked, and the decision from me that would unblock it.
- The log: the decisions this run added, by number, the findings it added, and the State it left.
- A finding that holds beyond this spec, such as the syntax for running a single test: the
  wording to add to `.claude/project.md` or CLAUDE.md, for me to apply.
