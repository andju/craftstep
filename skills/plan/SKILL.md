---
name: plan
description: Plan a new feature — interview me, push back on what won't work, then write an implementation-ready spec. Writes the spec and stops; never implements.
argument-hint: [short description of the feature]
allowed-tools: Read, Grep, Glob, AskUserQuestion, Write
disable-model-invocation: true
user-invocable: true
---

We are planning a new feature (if that's empty, ask what we're building
before anything else): $ARGUMENTS

## Step 0 — Load this project's conventions

Six facts drive this skill. Resolve each one independently, taking the first source that
settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it
   isn't. Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `plans`, `commands`, `tests`, `docs`, `reuse`,
   `extra_checks`. A missing file, a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing test and docs layout.

What each fact is, and how to infer it at source 3:

- **plans** — where the spec gets written. Infer from an existing plans or specs
  convention; default to `SPEC.md` at the repo root.
- **tests** — the tiers that exist here and what belongs in each. Infer by globbing for
  test locations and per-language test file patterns, reading one or two existing tests
  from each tier, and checking the CI config for how they're run.
- **docs** — which docs are user-facing. Infer by finding the docs directory and working
  out which part is user-facing. If nothing is, treat this as a project with no user docs.
- **reuse** — directories holding shared units to check before writing new code. Infer
  from where shared code lives in this stack.
- **extra_checks** — cross-cutting questions every change is held against. Infer
  nothing.
- **commands** — how to run each test tier and how to verify a work item. Infer from the
  package manifest's scripts and the CI config.

List every fact you inferred rather than read, in one short block, before continuing.

**Never edit `.claude/project.md` or CLAUDE.md while this skill runs.** If the skill turns
up a fact worth recording, propose the addition after the spec is written and let me apply it.

## Step 1 — Interview me

**If a spec for this feature already exists** at the `plans` path, this is a revision
pass: read it in full first, including any annotations or edits I made directly in the
file. You'll rewrite it incorporating them, so ask only about what those changes left
ambiguous. Don't re-interview on settled points — a revision pass is often one round,
sometimes none. Carry its **Rejected alternatives** forward into the rewrite — drop an
entry only if the revision explicitly revives that option, and then say so under
**Approach**.

Cover requirements, edge cases, and tradeoffs. Ask open questions as plain text; use
AskUserQuestion for genuine either/ors. Don't ask me to confirm what I've already told
you — dig into the hard parts I might not have considered. Read enough of the codebase
first that your questions are grounded in how this project actually works.

Ask in rounds, not one question at a time. A round is one turn: the either/ors go in a
single AskUserQuestion call — it takes up to four — with any open questions as plain text
alongside them. Batch what stands alone; hold back a question whose right form depends on
an answer you don't have yet.

Stop when you could write the step 3 work items — each one's files, finish condition, and
verify command — without guessing. Track the options you raise and I reject, and why.

Two or three rounds settle a normal feature. That count is a diagnostic, not a budget —
never stop asking because you've hit it, and never guess to stay under it. If a fourth
round is starting, something structural is off. Name which before you continue:

- This is several features. Propose the split; spec one slice now and list the rest.
- You're asking around an unsettled decision upstream of the questions. Name it and settle
  only that.
- You're asking things the repo answers. Go read it instead — see the grounding rule above.
- The answers stopped changing any work item. Stop and go to step 2.

Unsettled requirements always send you back for another round; they never become a caveat
in the spec.

## Step 2 — Push back

If any part is impossible or conflicts with how the system works today, say so. If
there's a better route to the same goal, propose it with the tradeoff. Settle this with
me before writing anything.

## Step 3 — Write the spec

The implementer will only have the repo and this spec.

So the spec contains no questions, no TODOs, no "decide during implementation," and no
alternatives left side by side for the implementer to choose between. If something is
still undecided when you reach this step, go back to step 1 and settle it.

Use these headings:

- **Feature** — what we're building and the requirements, in enough detail that someone
  who wasn't in the interview can tell whether the result is right.
- **Approach** — the chosen implementation route.
- **Rejected alternatives** — each option raised and dropped, with the reason, including
  any carried forward from an earlier revision. If the interview produced no real
  alternatives, write that instead of inventing some.
- **Work items** — numbered and ordered. Each one small enough to finish and verify
  alone, and self-contained: the files it touches, what it must not touch, its finish
  condition, and the exact command that proves it. Only write something new when
  nothing existing fits; Where an item builds on an existing unit from the
  `reuse` directories, it names that unit and the extension it needs — a prop,
  a variant, a parameter, an interface change.
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
- **Extra checks** — an explicit answer to every item in the `extra_checks` section.
  If the section is absent, say so exlicitly.
- **When this plan is wrong** — name the specific places you're least confident the
  codebase matches the plan, then state the rule: if reality contradicts this spec, stop
  and report the contradiction rather than improvising around it. Only take uncertainty
  about the codebase, not decisions nobody made.

## Where this skill stops

This skill writes exactly one file: the spec. It does not implement the feature. Summarize
the spec here and wait for my review.
