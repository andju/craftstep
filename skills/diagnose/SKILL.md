---
name: diagnose
description: Diagnose a bug I describe — reproduce it, find the root cause, and write it up as one finding in bug-findings.md for /craftstep:fix. Never fixes it and never edits code.
argument-hint: "the symptom — an error, a failing test, steps to reproduce; blank asks"
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are diagnosing a bug, not fixing it (if that's empty, ask what goes wrong before anything
else): $ARGUMENTS

**This skill writes one file: `bug-findings.md`.** It never edits code, tests or config — not to
fix, and not to instrument: a probe is a command you run, and it leaves nothing on disk. It never
touches `.claude/project.md` or CLAUDE.md. If the run turns up a fact worth recording, propose it
once you are done and let me apply it.

**The working tree may hold my uncommitted work.** Never `git stash`, `restore`, `checkout`,
`reset`, `clean` or `bisect`. History is read with `git log`, `git blame` and `git show`.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (non-root packages in a
   monorepo).
2. **`.claude/project.md`**, sections `commands`, `tests`. A missing file, section or key are all
   normal.
3. **The repo itself** — manifest scripts, CI, existing test layout.

What each fact is, and how to infer it from the repo:

- **commands** — how to run the thing that fails: a test tier, the build, the app. Infer from
  manifest scripts and CI.
- **tests** — how test files are named, to find the tests around the failing code and to say which
  tier a regression test belongs in. Infer from the runner's config and the language's convention.

List the facts in one short block, each with the source that settled it.

## Step 1 — Pin the symptom

Before looking for a cause, state in three lines: what happens, what should happen instead, and how
to make it happen.

Ask me, in one round, only for what is missing and the repo can't tell you: the exact input, the
environment, whether it ever worked, what changed since.

## Step 2 — Reproduce it

Run the cheapest thing that shows the failure — the one failing test, not its tier; the one
command, not the suite. Quote the failing output.

**Not reproduced** — say what you ran and what it showed, then ask me for the difference between my
run and yours. Don't diagnose a failure you haven't seen from the description alone — unless it
can't be reproduced here at all (production data, an external system, a race). Then say so, and the
finding is written at low confidence.

## Step 3 — Find the root cause

Work back from where the failure surfaces to where it starts. The line that throws is rarely the
defect; the defect is where the wrong value, state or assumption was introduced.

- **Read narrowly.** Start from the stack trace or the failing assertion. Grep first, then ranged
  reads around what the grep lands on; a whole file only when it is short.
- **One hypothesis at a time, cheapest check first.** State it, state what you would see if it were
  true, run the check, and say whether it held.
- **A cause is confirmed when it explains the symptom and a check bears it out** — you can say why
  this input fails and a neighbouring one doesn't. "This looks wrong" is a hypothesis.
- **After three refuted hypotheses, stop.** Report what is ruled out and what you would look at
  next, and ask me before spending more.

Once the cause is confirmed, grep for the same mistake elsewhere — the same call, the same pattern,
a copy of the function. Other sites belong in the same finding.

**Not a defect in this repo** — a wrong environment, bad data, a dependency's bug, the thing used as
it was never meant to be: write no finding. Step 5 reports what it is and what would resolve it.

## Step 4 — Write the finding

`bug-findings.md` at the repo root (`git rev-parse --show-toplevel`). The file collects bugs across
runs, so it may already exist — **never read it whole**:

- **It exists** — grep `^### ` for the highest number; this finding is the next. Add its checklist
  line after the last line matching `^- \[`, and its section at the end of the file, with Edit.
- **It doesn't** — write it with the header below.

**Write what only this run knows, and stop there.** The edit and the commands that prove it are the
fixer's; don't write them.

````markdown
# Bug findings

## Checklist

- [ ] **001** · <severity> · correctness — <title>
- [ ] **002** · <severity> · correctness · low confidence — <title>

## Findings

### 001 — <title>

- **Category:** correctness
- **Severity:** critical | high | medium | low
- **Confidence:** high | medium | low
- **Sites:** `path:line`, `path:line-range` — where the cause sits, not where it surfaces
- **Apply:** independent | after NNN

**Symptom.** What goes wrong and what should happen instead, for someone who never saw the report.

**Reproduce.** The exact command and the failing output, quoted. Not reproduced: what was tried.

**What's wrong.** The mechanism, quoted from the code: where the wrong value, state or assumption
is introduced, and how it gets from there to the symptom.

**Why it matters.** Who hits it and when; what else the same cause breaks.

**Direction.** Which way the fix goes, in a sentence or two, and the tier a regression test for it
belongs in. Not the edit.

**Check first.** The one place that would make this fine or move the cause, if anything does. At low
confidence: the one thing that would settle it.
````

Confidence is high only for a cause step 3 confirmed on a failure step 2 reproduced. A cause you
believe but could not confirm is written at low confidence. Number three digits, zero-padded; leave
every box empty.

## Step 5 — Report

In chat, under 150 words: the root cause in one sentence, the finding's id, its confidence and what
it rests on, hypotheses ruled out, and anything else noticed along the way — unfixed, for a review
to pick up. Then the next step: `/craftstep:fix bug-NNN`, in a fresh session.
