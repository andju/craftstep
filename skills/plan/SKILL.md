---
name: plan
description: Plan a new feature — interview me, push back on what won't work, then write an implementation-ready spec. Writes the spec and stops; never implements.
argument-hint: "short description of the feature, or next for the first backlog entry"
allowed-tools: Read, Grep, Glob, AskUserQuestion, Write, Agent(Explore)
disable-model-invocation: true
user-invocable: true
---

We are planning a new feature (if that's empty, ask what we're building
before anything else): $ARGUMENTS

**This skill only writes the spec at `plans`.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording,
propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

The facts below drive this skill. Resolve each one independently, taking the first source that
settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it
   isn't. Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `plans`, `backlog`, `commands`, `tests`, `docs`, `reuse`.
   A missing file, a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing test and docs layout.

What each fact is, and how to infer it from the repo:

- **plans** — the file the spec gets written to. Default to `.craftstep/spec.md`.
- **backlog** — the file deferred slices are parked in. Read only when I ask for an entry;
  never written here. Default to `.craftstep/backlog.md`.
- **tests** — the tiers that exist here and what belongs in each. Infer by globbing for
  test locations and per-language test file patterns, reading one or two existing tests
  from each tier, and checking the CI config for how they're run.
- **docs** — which docs are user-facing. Infer by finding the docs directory and working
  out which part is user-facing. If nothing is, treat this as a project with no user docs.
- **reuse** — directories holding shared units to check before writing new code. Infer
  from where shared code lives in this stack.
- **commands** — how to run each test tier and how to verify a work item. Infer from the
  package manifest's scripts and the CI config.

List every fact in one short block, each with the source that settled it.

## Step 1 — Settle what is already decided, then the scope

**If I ask you to plan from a decision record or design:** Read the path I named in $ARGUMENTS,
or a document I pasted or attached in full before anything else. Never go looking for one. A
record I did point you at has already settled things this skill would otherwise interview for:

- Its decision **is** the Approach. Step 3 pushes back on how that decision meets this
  codebase, never on the decision itself. If you think the decision is wrong, say so once
  and stop.
- Its discarded options are **Rejected alternatives**. Carry each across with the reason it
  was recorded under; add to that list rather than re-arguing it.
- Interview strictly below its altitude: Anything the record settles is not a question.
- Its risks describe operating the decision, not places the codebase might contradict this
  spec. They don't belong in **When this plan is wrong**.

**A record whose status is `proposed` rests on an assumption nobody has tested.** Warn me before
anything else: quote the assumption from its **Revisit if**, name the option that takes over if
it fails, and say that a spec written now is built on it. Then ask me with AskUserQuestion, the
assumption restated in the question itself: **stop** — so I can run `/craftstep:spike <record
path>` and finish the record first — or **plan anyway**. If I plan anyway, the assumption goes
under **When this plan is wrong**, in the record's words.

**If I ask for a backlog entry** — `next`, or an entry's title: find it in the `backlog` file with
a grep on `^## ` and a ranged read of that one entry; never read the backlog whole. `next` is the
first entry whose **Depends on** is met. Nothing there — say so and stop.

- The entry is the request. What it records as settled is not a question.
- It was written against the code as it was then: check what it assumes against the code as it is
  now before relying on it.
- Read the decision record it names, and treat it as above.
- Leave the backlog file alone. Name the entry under **Feature** as `Backlog entry: <title>`;
  `/craftstep:wrap-up` removes it once the feature is done.

**Settle the size.** Whatever you're working from can cover more than one feature. If what
you're holding needs more than one spec, say so now: propose the split as an ordered list
of slices, name the one this run specs, and record the rest under **Deferred**. Interview about
that slice only.

**Once you know which feature this run specs, check the `plans` path.** The spec is a working
document holding one feature at a time, so anything already there is either an earlier pass at
this feature or a leftover from a finished one. Read its **Feature** section and say which:

- **The same feature — a revision pass.** Read it in full first, including any annotations or
  edits I made directly in the file. You'll rewrite it incorporating them, so ask only about
  what those changes left ambiguous, and don't re-interview on settled points. Carry its
  **Rejected alternatives** forward into the rewrite — drop an entry only if the revision
  explicitly revives that option, and then say so under **Approach**. Read the implementation
  log too, if `/craftstep:implement` left one beside the spec (its path with `.md` replaced by
  `.log.md`): its decisions are answers I already gave and its findings are facts about the
  code. Fold in each one that changes what gets built, and ask about none of them again. The
  rewrite supersedes the log, so your summary tells me to delete it before the next
  `/craftstep:implement`.
- **A different feature.** Name the feature it holds and confirm with me before overwriting
  it. If it has **Deferred** slices, say that overwriting loses them unless
  `/craftstep:wrap-up` has already moved them to the backlog; the same goes for the decisions in
  an implementation log beside it.

This step still ends by naming the scope: say in one line what this run specs, and what it
leaves out.

## Step 2 — Interview me

Cover requirements, edge cases, and tradeoffs. Ask open questions as plain text; use
AskUserQuestion for genuine either/ors. Don't ask me to confirm what I've already told
you — dig into the hard parts I might not have considered. Read enough of the codebase
first that your questions are grounded in how this project actually works.

**Locate before you read.** Grep and Glob first; read what they land on, by ranged read where a
file is long. Where finding the area would take a sweep across many files or directories, hand
the search to one `Explore` subagent: give it the question and ask for paths, symbols and line
ranges back, not file contents. Then read what it located yourself — the spec has to be right
about the code, and a subagent's summary is a pointer, not evidence.

Ask in rounds, not one question at a time. A round is one turn: the either/ors go in a
single AskUserQuestion call — it takes up to four — with any open questions as plain text
alongside them. Batch what stands alone; hold back a question whose right form depends on
an answer you don't have yet.

Stop when you could write the step 4 work items — each one's files, finish condition, and
verify command — without guessing. Track the options you raise and I reject, and why.

Two or three rounds settle a normal feature. That count is a diagnostic, not a budget —
never stop asking because you've hit it, and never guess to stay under it. If a fourth
round is starting, something structural is off. Name which before you continue:

- This is several features. Propose the split; spec one slice now and put the rest under
  **Deferred**.
- You're asking around an unsettled decision upstream of the questions. Name it and settle
  only that.
- You're asking things the repo answers. Go read it instead — see the grounding rule above.
- The answers stopped changing any work item. Stop and go to step 3.

Unsettled requirements always send you back for another round; they never become a caveat
in the spec.

## Step 3 — Push back

If any part is impossible or conflicts with how the system works today, say so. If
there's a better route to the same goal, propose it with the tradeoff. Settle this with
me before writing anything.

## Step 4 — Write the spec

The implementer will only have the repo and this spec.

So the spec contains no questions, no TODOs, no "decide during implementation," and no
alternatives left side by side for the implementer to choose between. If something is
still undecided when you reach this step, go back to step 2 and settle it.

Use these headings:

- **Feature** — what we're building and the requirements, in enough detail that someone
  who wasn't in the interview can tell whether the result is right. A spec planned from
  a backlog entry names it here: `Backlog entry: <title>`.
- **Approach** — the chosen implementation route. Where a decision record settled it, cite
  the record and cover only what the record left to this level.
- **Rejected alternatives** — each option raised and dropped, with the reason, including
  any carried forward from an earlier revision or from a decision record. If the interview
  produced no real alternatives, write that instead of inventing some.
- **Work items** — numbered and ordered. Each one small enough to finish and verify
  alone, and self-contained: the files it touches, what it must not touch, its finish
  condition, and the exact command that proves it. Only write something new when
  nothing existing fits; Where an item builds on an existing unit from the
  `reuse` directories, it names that unit and the extension it needs — a prop,
  a variant, a parameter, an interface change. Work item titles are written as an
  unchecked checkbox: `1. [ ] <what the item does>`
- **Acceptance criteria** — checkable statements of done, in terms of behaviour rather
  than implementation. Include what must *not* change.
- **Tests** — which tests and which tier each belongs to, justified against the boundary
  you observed in the existing suite rather than the tier's name, plus the exact command
  to run each. Mark any command you inferred rather than read from a stated convention.
  If you conclude no tests are warranted, justify that.
- **Docs** — which user-facing file(s) change, or whether a new page is needed and where
  it links from. If the change is internal-only or there is no user-facing documentation,
  say so explicitly.
- **Reuse** — the units from the `reuse` directories you checked and did *not* build
  on, and what didn't fit about each. If everything you checked fits, say that instead
  of leaving the section empty.
- **Deferred** — every slice cut from this spec, at any step, that is still meant to be built,
  in the order it should be built. Per slice: a title, what it covers, the requirements the
  interview already settled, what it depends on, and why it was cut. `/craftstep:wrap-up` moves
  these to the backlog and a later run plans from them, so each stands alone without the rest of
  this spec. The implementer builds none of it. If nothing was cut, say so.
- **When this plan is wrong** — name the specific places you're least confident the
  codebase matches the plan, then state the rule: if reality contradicts this spec, stop
  and report the contradiction rather than improvising around it. Only take uncertainty
  about the codebase, not decisions nobody made.

Summarize the spec here and wait for my review.
