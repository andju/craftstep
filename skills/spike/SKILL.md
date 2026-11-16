---
name: spike
description: Answer one open question with throwaway code — agree the question and what counts as an answer, build the least that settles it in a separate worktree, and write a spike report. The code is never merged.
argument-hint: "the question to answer, or the decision record it comes from; blank asks"
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are answering a question, not building a feature (if that's empty, ask what the question is
before anything else): $ARGUMENTS

**This skill writes one file in my repo: the report, `spike-<slug>.md` at the repo root.** Everything
else it writes lands in the spike's worktree. It never edits a decision record, a spec,
`.claude/project.md` or CLAUDE.md, and never merges, cherry-picks or copies spike code back. If the
run turns up a fact worth recording, propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

This skill needs one fact; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (non-root packages in a
   monorepo).
2. **`.claude/project.md`**, section `commands`. A missing file, section or key are all normal.
3. **The repo itself** — manifest scripts, CI.

- **commands** — how to install, build and run this project, for getting the worktree to a state
  where the spike can run. Infer from manifest scripts and CI.

State it in one short block, with the source that settled it.

## Step 1 — Agree the question before writing anything

If $ARGUMENTS names a decision record, read it; the question comes from its **Revisit if** — a
`proposed` record leads that list with the assumption it rests on — or from its risks. Otherwise
read only enough of the repo to phrase the question against how this system actually works.

Then put four things to me, and wait for my answer:

- **The question** — one, answerable with yes, no or a number. Two questions are two spikes: name
  the one this run answers.
- **What counts as an answer** — the observation that means yes, the one that means no, and what
  would leave it inconclusive. A threshold is agreed now, not read off the result afterwards.
- **What you will build** — the least that produces that observation, in three lines; and what you
  will leave out: tests, error handling, edge cases, structure.
- **The box** — where you stop if it grows. Default: the build as described; if it turns out to
  need about twice that, you stop and report *inconclusive* rather than enlarging it.

If reading the repo, a doc or a one-line command already answers the question, say so and answer it
— no spike.

## Step 2 — Set up the worktree

`git worktree add ../<repo-name>-spike-<slug> -b spike/<slug>` from the current `HEAD`, then install
what the spike needs there. My uncommitted changes are not in it — if the question depends on them,
say so and ask me before continuing.

## Step 3 — Build the least that answers it

- Hardcode, stub and skip whatever doesn't bear on the question.
- An obstacle beside the question gets routed around the cheapest way. An obstacle that *is* the
  question — the library can't do it, the API doesn't return it — is the answer: stop and record it.
- Stop the moment the agreed observation is in. Don't finish the feature around it.

## Step 4 — Observe

Run it and quote what it printed. A number comes with how it was measured: the command, the input
size, the machine, the number of runs. One run is an anecdote; say so if that is all there is.

## Step 5 — Write the report

`spike-<slug>.md` at the root of my repo — the checkout this session started in, not the spike's
worktree. If a file of that name is already there, say so and ask before overwriting it.

The report is a working document, not a record: it lasts until the design or spec it feeds is
written, and whatever has to outlive it is carried into that document. It stands on its own, for
`/craftstep:design-solution` and `/craftstep:plan` to read when I point them at it:

- **Question** — as agreed in step 1, with what was to count as an answer.
- **Answer** — yes | no | inconclusive, in one line, then the number if there is one. A spike that
  hit its box is *inconclusive*, with what would settle it.
- **Evidence** — the commands run and their output, quoted.
- **What was built** — in a few lines, and the shortcuts taken that make the code unfit to merge.
- **What this doesn't show** — the conditions the spike didn't cover: load, data shape, platforms,
  failure paths.
- **Surprises** — what turned up that the question didn't ask about and a design should know.
- **What it means for the decision** — which option or assumption it strengthens or weakens, by
  name where a record was given. Evidence for a decision, not a new recommendation.
- **The code** — the branch and worktree path, or that both were removed.

## Step 6 — Throw the code away

Ask me: remove the worktree and its branch — the default — or keep them for reference. On remove:
`git worktree remove --force <path>`, then `git branch -D spike/<slug>` — only the worktree and the
branch this run created. Update the report's last section to match.

## Step 7 — Report

In chat, under 150 words: the question, the answer, what it rests on and what it doesn't show, and
the report's path. Code worth keeping is rebuilt properly, through `/craftstep:plan`. Where the
question came from a `proposed` record, end with the command that finishes it:
`/craftstep:design-solution <record path> <report path>`, in a fresh session.
