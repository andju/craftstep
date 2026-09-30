---
name: design-solution
description: Design a solution before anything gets planned — interview me, weigh genuinely different options against criteria we agree, then recommend one and write the decision record. Never writes a spec or code.
argument-hint: [the problem this run settles; blank asks]
allowed-tools: Read, Grep, Glob, Bash(git:*), Bash(mkdir:*), Write, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are designing a solution to a problem, not building one (if that's empty, ask what the
problem is before anything else): $ARGUMENTS

The run produces one decision record: the problem, the criteria, the options, and the single
option you recommend — written for whoever picks this up in a year and needs to know why the
others lost.

**You recommend nothing before step 5.** Steps 1–4 stay neutral. An option you have privately
already chosen contaminates the criteria that are supposed to judge it.

**Steps 1–4 each end with a checkpoint.** Don't start the next step until I've answered it. If
I tell you to skip ahead, do — and say in one line what that costs.

**This skill only writes the record under `decisions`.** It never touches other files,
`.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording, propose it once
you are done and let me apply it.

## Step 0 — Load this project's conventions

The facts below drive this skill. Resolve each one independently, taking the first source that
settles it:

1. **The project's CLAUDE.md.** Usually already in context; read it from disk if it isn't.
   Subagents and non-root packages in a monorepo don't always have it.
2. **`.claude/project.md`**, sections `decisions`, `docs`, `tests`, `reuse`. A missing file,
   a missing section, and a missing key are all normal.
3. **The repo itself** — package manifest, CI config, existing docs and test layout.

What each fact does here, and how to infer it at source 3:

- **decisions** — where records are read from and the new one is written. Infer from an existing
  record convention wherever it sits; default to `docs/architecture/decisions/`.
- **docs** — which docs are user-facing: what the system promises its users constrains every
  option. Infer from the docs directory; if none is, treat this as a project with no user docs.
- **tests** — the tiers that exist and what they cover, a proxy for how safely this system can be
  changed and the input to every effort estimate. Infer by globbing test locations and checking CI.
- **reuse** — directories holding shared units an option could build on. This is what makes step
  4's minimal-change baseline real rather than a straw option. Infer from where shared code lives.

Where two sources settle the same fact differently, the higher one wins. List every fact in one short
block, each with the source that settled it.

## Step 1 — Research before you ask me anything

Never ask me for something the repo will tell you. Build the picture first, looking for what
applies:

- What the system does, where its entry points are, and the shape of the area this problem
  touches.
- Existing records in `decisions`, plus README and CONTRIBUTING: Cite decisions already taken
  as constraints; don't reopen them.
- The user-facing docs `docs` points at.
- Language, framework and dependency versions, and where this runs — package manifest,
  Dockerfiles, CI workflows, IaC, `.env.example`. They bound which options exist at all.
- Test coverage of the affected area, per `tests`.
- Shared units under `reuse` an option could build on.
- `git log` over the affected paths: prior attempts at this problem, and how much the area churns.

**Checkpoint 1.** Report what you learned in one short block, splitting **observed** from
**inferred**, and name in one line the decision this run settles and what it leaves to another run.
Ask me to correct it. No interview questions yet — a wrong inference here would misdirect every
question you ask next.

## Step 2 — Interview me

Fill only the gaps research couldn't. Where step 1 already answers something, confirm the
inference in a single line and move on.

Ask in rounds, not one question at a time. A round is one turn: genuine either/ors go in a single
AskUserQuestion call — it takes up to four — with open questions as plain text alongside. Batch
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
- You're asking me what the repo answers (go read it)
- The answers stopped changing the options (go to step 3).

**Checkpoint 2.** Summarize the problem, goals and constraints as you now understand them, and
ask me to confirm or correct.

## Step 3 — Agree the criteria before you see the options

Propose the criteria this decision will be judged on, ranked, each with one line on why it ranks
where it does. Draw them from step 2 so they read as specific to this problem. Flag anything you're
unsure ranks right.

A criterion that first appears in step 5 means this step was done wrong.

**Checkpoint 3.** Get explicit agreement on the criteria and their ranking. Adjust until I'm
satisfied.

## Step 4 — Develop the options

Three or four options, occupying genuinely different positions in the trade-off space — not one
idea at three sizes. **One is always the minimal-change baseline**: do nothing, or the smallest
intervention that uses what `reuse` already holds. Say honestly what happens if I take it,
including the case where it turns out to be enough. The baseline is what makes the comparison real.

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
   need.** Don't ever fabricate numbers.
6. **Key risks** — each with the earliest signal that it's materialising.

Argue every option at its strongest, the ones you intend to argue against included. An option
that's genuinely weak gets dropped and replaced, never kept as filler to make another look good.

**Checkpoint 4.** Ask what's missing and what deserves more depth, before you recommend.

## Step 5 — Recommend

1. **Exactly one option.** No hedging, no "it depends", and no blended fifth option invented here.
2. **Justified against the agreed criteria in their agreed order** — not against generic best
   practice, and never on a criterion nobody ranked.
3. **Pre-mortem.** It is twelve months on and this choice has failed. Write the story: the two or
   three most likely failure modes, and the earliest sign of each.
4. **What would change your mind** — which change in budget, timeline, scale or team flips the
   recommendation, and to which option by name.
5. **Next steps** — the smallest concrete actions that de-risk or advance the decision: a spike, a
   load test, a sign-off, a vendor conversation.

If I choose a different option, that one is the decision and it goes in the record with my
reasoning; yours stays in as the alternative it beat. Record what was decided, not what you would
have decided.

## Step 6 — Write the record

Create the `decisions` directory if it isn't there. Write `NNNN-<short-title>.md`, numbered after
the highest existing record.

The record is what `/craftstep:plan` reads when I point it at one, so it has to stand on its own:

- **Title, date, status** — proposed | accepted.
- **Context** — the problem, goals and constraints from step 2, plus the step 1 facts the decision
  rests on, for someone who wasn't in the session.
- **Decision criteria** — the ranked list from step 3, with the ranking rationale.
- **Options considered** — every option including the baseline, with its main pros and cons, and
  **why each rejected one was rejected**. This is the part that matters most later and is lost
  most often.
- **Decision** — the option taken and its justification, criterion by criterion.
- **Risks and early warning signs** — from the pre-mortem.
- **Revisit if** — the conditions that reopen this decision, from step 5.
- **What this implies** — the work the decision creates, as an ordered list of slices when it's
  more than one feature's worth. Sizing only: a slice is not a work item, and this is not a spec.

The record carries no open questions, no TODOs and no two options left side by side. Anything still
undecided sends you back to the step that decides it.

## Where this skill stops

This skill writes one decision record. It doesn't write a spec and it doesn't implement. Summarize
the record here and wait for my review.
