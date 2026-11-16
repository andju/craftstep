---
name: design-solution
description: Design a solution before anything gets planned — interview me, weigh genuinely different options against criteria we agree, then recommend one and write the decision record; or revise a proposed record once a spike has tested what it rests on. Never writes a spec or code.
argument-hint: "the problem this run settles, or a proposed record to revise; blank asks"
allowed-tools: Read, Grep, Glob, Bash(git:*), Write, AskUserQuestion, Agent(Explore)
disable-model-invocation: true
---

We are designing a solution to a problem, not building one (if that's empty, ask what the
problem is before anything else): $ARGUMENTS

**You recommend nothing before step 5.** An option you have privately already chosen
contaminates the criteria that are supposed to judge it.

**Steps 1–4 each end with a checkpoint.** Don't start the next step until I've answered it. If
I tell you to skip ahead, do — and say in one line what that costs.

**This skill only writes the record under `decisions`.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording, propose it once
you are done and let me apply it.

## Step 0 — Load this project's conventions

The facts below drive this skill. Resolve each one independently, taking the first source that
settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it isn't.
2. **`.claude/project.md`**, sections `decisions`, `docs`, `tests`, `reuse`. A missing file,
   a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing docs and test layout.

What each fact does here, and how to infer it from the repo:

- **decisions** — where records are read from and the new one is written. Infer from an existing
  record convention wherever it sits; default to `docs/architecture/decisions/`.
- **docs** — which docs are user-facing: what the system promises its users constrains every
  option. Infer from the docs directory; if none is, treat this as a project with no user docs.
- **tests** — the tiers that exist and what they cover. Infer by globbing test locations and
  checking CI.
- **reuse** — directories holding shared units an option could build on. Infer from where shared
  code lives.

List every fact in one short block, each with the source that settled it.

## A record I point you at — the revision pass

If $ARGUMENTS names a decision record, read it in full, with any annotations I made in the file,
and any spike report I named alongside it. Never go looking for either. Then go by its status:

- **`accepted`** — it is not revised. Say so and stop: reopening it is a new run, whose record
  names the one it supersedes.
- **`proposed`** — this run finishes it:
  - Its **Context** and **Decision criteria** are settled. Skip steps 1–3: re-read only the code
    the new evidence touches, and interview only on what my annotations left ambiguous.
  - **The criteria and their ranking are frozen.** They were agreed before the evidence came in,
    which is what stops the evidence from choosing them. If it makes one look wrong, say which and
    why, and change it only on my answer — the record then states the change and its reason.
  - Open with one block: what the record decided, the assumption it rested on, and what the
    evidence says about it — quoting the report's answer and its **What this doesn't show**. An
    *inconclusive* spike tested nothing: say so and ask whether to spike again or decide without it.
  - Then redo step 4 for the options the evidence moves, leaving the others as recorded, and
    steps 5 and 6 in full.

## Step 1 — Research before you ask me anything

Never ask me for something the repo will tell you. Build the picture first, looking for what
applies:

- What the system does, where its entry points are, and the shape of the area this problem
  touches.
- Existing records in `decisions`, plus README and CONTRIBUTING: Cite decisions already taken
  as constraints; don't reopen them.
- A spike report I pointed you at: evidence for or against an option, bounded by its own
  **What this doesn't show** — not a decision taken.
- The user-facing docs `docs` points at.
- Language, framework and dependency versions, and where this runs — package manifest,
  Dockerfiles, CI workflows, IaC, `.env.example`. They bound which options exist at all.
- Test coverage of the affected area, per `tests`.
- Shared units under `reuse` an option could build on.
- `git log --oneline -n 30 -- <paths>` over the affected paths: prior attempts at this problem,
  and how much the area churns. Open a commit only when its subject says it bears on the problem.

**Locate before you read.** Grep and Glob first; read what they land on, by ranged read where a
file is long. Where finding the area would take a sweep across many files or directories, hand
the search to one `Explore` subagent: give it the question and ask for paths, symbols and line
ranges back, not file contents. Then read what it located yourself — a subagent's summary is a
pointer, not something to report as observed.

**Checkpoint 1.** Report what you learned in one short block, splitting **observed** from
**inferred**, and name in one line the decision this run settles and what it leaves to another run.
Ask me to correct it. No interview questions yet — a wrong inference here would misdirect every
question you ask next.

## Step 2 — Interview me

Fill only the gaps research couldn't. Where step 1 already answers something, confirm the
inference in a single line and move on.

Ask in rounds, not one question at a time. A round is one turn: genuine either/ors go in a single
AskUserQuestion call, with open questions as plain text alongside. Batch
what stands alone; hold back a question whose right form depends on an answer you don't have yet.

Cover, as needed:

- **The real problem** — what is broken, slow, risky or missing, and what made it unacceptable now.
- **Goals** — what "solved" looks like in observable terms.
- **Constraints** — budget, timeline, team size and skills, systems that must survive, compliance,
  security, operational limits.
- **Scale** — who and what is affected, today's volume, the growth you have to hold.
- **Trade-off appetite** — where I'll give ground: speed vs. cost, build vs. buy, ideal vs. shippable.
- **Reversibility** — cheaply reversible or a one-way door: it sets how much evidence step 5 needs.
- **Prior attempts** — tried before, and what happened; also what has already been ruled out,
  and why — a rejected option needs a reason before it stays off the table.
- **Your instinct** — what you're currently leaning towards, and your gut worry. It gives step 5 a
  position to argue with rather than one to discover.

If the answers say the stated problem isn't the real problem, say so plainly and get the reframing
agreed before continuing — don't design against a problem you no longer believe in. If a stated
constraint contradicts a stated goal, surface it in the round you notice it, not in the write-up.

Stop when you could argue three genuinely different options at their strongest. When a fourth round
of questions is starting, name why before you continue:
- This is several decisions rather than one (split them, say which has to settle first because the
others depend on it, and design only that one)
- The answers stopped changing the options (go to step 3).

**Checkpoint 2.** Summarize the problem, goals and constraints as you now understand them, and
ask me to confirm or correct.

## Step 3 — Agree the criteria before you see the options

Propose the criteria this decision will be judged on, ranked, each with one line on why it ranks
where it does. Draw them from step 2 so they read as specific to this problem. Flag anything you're
unsure ranks right.

**Checkpoint 3.** Get explicit agreement on the criteria and their ranking. Adjust until I'm
satisfied.

## Step 4 — Develop the options

Three or four options, occupying genuinely different positions in the trade-off space — not one
idea at three sizes. **One is always the minimal-change baseline**: do nothing, or the smallest
intervention that uses what `reuse` already holds. Say honestly what happens if I take it,
including the case where it turns out to be enough.

Open with a comparison table — options as rows, the agreed criteria as columns, plus effort and a
one-line characterisation. Then the detail, per option:

1. **Name** — short, memorable, and used for it everywhere after this.
2. **Summary** — two or three sentences.
3. **How it works** — enough to judge it, not a specification.
4. **Pros** and **cons** — tied to the agreed criteria, the cons including the ones that only show
   up in year two.
5. **Effort** — relative sizing (S/M/L), informed by what `tests` told you about how safely this
   area can be changed. Any figure in time or money states the assumptions under it. **If you don't
   have the information to estimate, write "insufficient information to estimate" and say what you'd
   need.**
6. **Key risks** — each with the earliest signal that it's materialising.

Argue every option at its strongest, the ones you intend to argue against included. An option
that's genuinely weak gets dropped and replaced, never kept as filler to make another look good.

**Checkpoint 4.** Ask what's missing and what deserves more depth, before you recommend. Name any
question only running code would answer, and the options whose standing hangs on it.

## Step 5 — Recommend

1. **Exactly one option.** No hedging, no "it depends", and no blended fifth option invented here.
2. **Justified against the agreed criteria in their agreed order** — not against generic best
   practice, and never on a criterion nobody ranked.
3. **Pre-mortem.** It is twelve months on and this choice has failed. Write the story: the two or
   three most likely failure modes, and the earliest sign of each.
4. **What would change your mind** — which change in budget, timeline, scale or team flips the
   recommendation, and to which option by name.
5. **What it rests on** — any assumption the recommendation depends on that nobody has tested,
   put as a question `/craftstep:spike` could answer with yes, no or a number, and the option that
   takes over, by name, if the answer goes against it. This is not a hedge: the recommendation
   stands, and says what it stands on. Nothing untested — say so.
6. **Next steps** — the smallest concrete actions that de-risk or advance the decision: a spike, a
   load test, a sign-off, a vendor conversation.

If I choose a different option, that one is the decision and it goes in the record with my
reasoning; yours stays in as the alternative it beat.

## Step 6 — Write the record

Write `NNNN-<short-title>.md` under `decisions`, numbered after the highest existing record. On a
revision pass, rewrite the record I named instead — same file, same number, never a second record.

The record is what `/craftstep:plan` reads when I point it at one, so it has to stand on its own:

- **Title, date, status** — `proposed` while step 5 named an untested assumption; `accepted`
  otherwise. A revision pass sets the date to today.
- **Context** — the problem, goals and constraints from step 2, plus the step 1 facts the decision
  rests on, for someone who wasn't in the session. Where a spike report fed the decision, carry
  its question, its answer, how that was measured and what it doesn't show into the record — here
  or under the option it bears on. The report is a throwaway file: never link to it.
- **Decision criteria** — the ranked list from step 3, with the ranking rationale.
- **Options considered** — every option including the baseline, with its main pros and cons, and
  **why each rejected one was rejected**. When a revision pass changes the decision, the option
  that lost stays here, with the evidence that beat it.
- **Decision** — the option taken and its justification, criterion by criterion.
- **Risks and early warning signs** — from the pre-mortem.
- **Revisit if** — the conditions that reopen this decision, from step 5. An untested assumption
  leads the list, as the spike question and the option that takes over if it fails; it leaves the
  list once a revision pass has carried its evidence into the record.
- **What this implies** — the work the decision creates, as an ordered list of slices when it's
  more than one feature's worth. Sizing only: a slice is not a work item, and this is not a spec.

The record carries no open questions, no TODOs and no two options left side by side. Anything still
undecided sends you back to the step that decides it. An untested assumption is none of those: the
decision is taken, and the record says what it rests on.

Summarize the record here and wait for my review. For a `proposed` record, end with the two
commands that finish it: `/craftstep:spike <record path>`, then `/craftstep:design-solution
<record path> <report path>` in a fresh session.
