---
name: fix
description: Work one finding off a review's checklist — the one I name, or the next open one — by checking it against the code, deciding the fix, and ticking it off. Larger fixes go to a subagent.
argument-hint: [finding — e.g. code-03, test-07; blank picks the next open one; add "ask" or "no-ask" to force or skip my confirmation]
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, AskUserQuestion, Agent(finding-fixer), SendMessage
disable-model-invocation: true
user-invocable: true
---

We are fixing a finding from a review that `/craftstep:review-code` or `/craftstep:review-tests`
wrote. Finding for this run (blank → step 1 picks the next open one): $ARGUMENTS

**The finding is a claim, not a fact.** It was written from one read of the code by a session that
never checked it afterwards. Nothing in the code changes until step 2 has confirmed it. That check
is yours, in this session, and is never delegated.

**The fix is yours to decide, too.** The review says what's wrong, where, and which way a fix should
go — not the edit. Step 3 turns that into a brief, against the code as it is now. Small fixes you
apply yourself; anything larger goes to the `finding-fixer` subagent with your brief. Either way,
step 4 proves the result — nobody's word counts, yours included.

**This skill edits code, and one line of the findings document per finding: its checklist entry.**
It never edits anything else in that document, `.claude/project.md` or CLAUDE.md, and never commits.
If the run turns up a fact worth recording, propose it once you are done and let me apply it.

**The working tree may hold my uncommitted work** — often the very changes that were reviewed.
Never `git stash`, `restore`, `checkout`, `reset` or `clean`; an edit is undone with another edit.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (non-root packages in a
   monorepo).
2. **`.claude/project.md`**, sections `commands`, `tests`. A missing file, section or key are all
   normal.
3. **The repo itself** — manifest scripts, CI, existing test layout.

What each fact does here, and how to infer it at source 3:

- **commands** — lint, types, build, each test tier: the brief's `VERIFY`. Infer from manifest
  scripts and CI.
- **tests** — where a file of each tier goes and what shape it takes: the brief's `TEST FIRST`.
  Infer by globbing test locations and reading one existing test of the tier being written into —
  and only once step 3 knows this fix needs a test.

Where two sources settle the same fact differently, the higher one wins. List the facts in one short
block, each with the source that settled it; `tests`, when it waits for step 3, is listed there.

## Step 1 — Pick the finding

**Never read a findings document whole.** It may hold dozens of findings and you need one. Work it
with greps and ranged reads:

- **The checklist** — one line per finding, `- [ ] **NN** · severity · category — title`. The open
  ones: `grep -n "^- \[ \]"`.
- **A finding** — `grep -n "^### NN "`, then a ranged read from that line to the next `### `.

Step 5 marks the checklist; what each state means:

| Line | Meaning | Picked by a blank run? |
|---|---|---|
| `- [ ] …` | not worked yet | yes |
| `- [x] … — fixed` | applied and verified, or found already gone | no — closed |
| `- [x] … — not a defect: <path:line>` | refuted, by the code or by me | no — closed |
| `- [ ] … — needs you: <question>` | waits on a decision of mine | no — open until I name it |

- **Named** — `code-03`, `test-7`, or a bare `03` where only one document has an `03` (both have
  one — ask which). Closed — say so and stop. A `needs you` I name again comes with my decision, in
  $ARGUMENTS or in answer to its question asked again. Its `Apply` waits on a finding that isn't
  fixed — say which, and ask whether to go ahead anyway.
- **Blank** — the first unmarked open line whose finding's `Apply` doesn't wait on one that isn't
  fixed. With both documents present, every `code-` finding comes before any `test-`, so tests are
  written against the code as fixed. Nothing left — say so, list each `needs you` with its
  question, and stop.
- **`Apply: together with NN`** — one fix with NN: check both, brief both as one, tick both.

No `code-findings.md` or `test-findings.md` at the repo root — say so and stop. A checklist
line with no `### NN` section means the review didn't finish writing — say so and stop.

## Step 2 — Check the finding before anything changes

Read the finding's section, then its `Sites` — ranged reads around the cited lines, not whole files.
Then answer three questions in order, stopping at the first that closes it:

1. **Is it still there?** The code says what the finding quotes. Moved lines aren't a contradiction
   by themselves — a fix its `Apply` line names may have moved them; what matters is whether the
   defect survived. Already gone → `fixed`, nothing to edit; the report says what resolved it.
   Contradicted any other way → `needs you`, the contradiction quoted.
2. **Is it real?** Look where its **Check first** points — the caller that may already validate,
   the guard, the type, the config; for a missing test, a test elsewhere in the tree. If it points
   nowhere, find the one place that would make it fine yourself. Grep for it and read only what the
   grep lands on. Found, and it covers every path to the site → `not a defect`, citing the
   `path:line` that shows it.
3. **Is it settled?** At high or medium confidence, surviving 1 and 2 settles it. At low, read on.

Look no further than the sites and that one place. This is a check, not a second review. A finding
this step closes or leaves open goes straight to step 5.

### Low-confidence findings

The finding admits the code didn't settle it; its **Check first** names the one thing that would.
**Never apply one on its word.** Settle it the cheapest way that actually settles it:

- **From the code** — question 2, aimed at what **Check first** names.
- **With a failing test** — where the fix includes a regression test, that test is the settlement:
  written first, it must fail for the stated reason before any production code changes. Step 3
  briefs it that way, whatever the fix's size. It passes → `not a defect`.
- **Neither can** — it turns on intent, production data, or a system outside this repo. Ask me: one
  AskUserQuestion carrying the one thing that would settle it, with the options to fix it as the
  finding's **Direction** says, close it, or leave it open. Open → `needs you`.

**Never fix one defensively** — a guard "just in case", a try/catch around a path nobody showed can
fail. That change, made on a guess, is what this section exists to prevent.

## Step 3 — Decide the fix, then apply it

**Write the brief** — against the code you just read, not the code the review saw — and show it in
chat:

```
FINDING:    <id — title>, with its `together with` partner if any
CONFIRMED:  <what step 2 established: where you looked, what you found>
FIX:        <the instruction: files and symbols, and the behaviour that must hold afterwards —
             specific enough that two engineers would produce substantially the same diff>
DON'T:      <what must not change — public API, passing tests, unrelated files, formatting churn;
             for a test- finding, production code, but for the one seam named here>
TEST FIRST: <a regression test for the defect, or a new test for behaviour that works: the file,
             tier, scenario and what it must show — or "none">
VERIFY:     <the exact `commands` that prove it, and what passing looks like>
```

Follow the finding's **Direction**. Where it leaves a choice open, make it, and say so.

**Ask me before applying when the brief decides something the Direction didn't** — a public name or
signature, an error-handling policy, behaviour a caller or user can observe, a file outside the
`Sites`. Always ask when $ARGUMENTS says `ask`; never when it says `no-ask`. Name the decision the
brief makes, and offer: apply it, apply it with my change, leave it open (`needs you`), or close it
(`not a defect`). Otherwise go ahead without asking — the diff is mine to read before I commit.

**Baseline.** Run `git stash create` — it prints a commit holding the working tree as it is and
changes nothing; empty output means the tree is clean, so use `HEAD` — then
`git status --porcelain`. Step 4 measures the fix against these, so my own uncommitted changes
never count as part of it.

**Small — apply it yourself:** one file, a handful of lines, `TEST FIRST: none`. A subagent would
spend more starting up and re-reading than the edit costs. Follow the brief to the letter.

**Anything else — dispatch one `finding-fixer` subagent**, `run_in_background: false`, with the
brief as its prompt, plus:

- **For a low-confidence finding settled by a test: stop after the failing run.**
- **The tier commands from `commands`, named as a fallback only.**

Pass nothing else — not the findings document, not the review. What comes back:

- **DONE** — step 4.
- **REPRODUCED** — the regression test, stopped before the fix. Read it and run it yourself: it
  must fail *because of the defect*, not on an import, a typo or a fixture. It does → the finding is
  settled; resume the same subagent to apply the fix. It doesn't → a wrong test, below.
- **NOT REPRODUCED** — the regression test passed before any fix. Read it. It really exercises the
  defect → `not a defect`; remove what it added. It doesn't → a wrong test: resume the subagent with
  what's wrong with it.
- **BLOCKED** — the brief or the finding's **Direction** settles it in its own words → quote them
  and resume the same subagent. Otherwise the decision is mine: ask me, carrying its question as
  written. I leave it → `needs you`.

**The subagent gets one round back, in all** — a wrong test here or a failure in step 4 spends it.

## Step 4 — Prove it

Never take "done" from the subagent's report, nor from your own edit:

- **Read the fix's diff** — `git diff <baseline>`, plus the files new in `git status --porcelain`
  since the baseline and any others the report lists. It touches what `FIX` and `TEST FIRST` name,
  and nothing `DON'T` forbids.
- **Run `VERIFY` yourself** and read the output — the brief's commands, not the project's full
  gate. For a new test of working behaviour, read the report's run with the behaviour broken
  against the test's assertions: one that can't fail is a failure here.

Something fails → resume the same subagent, if its round is unspent, with the failing command and
its output quoted, or the stray change named — not your paraphrase. Your own small fix gets one
more attempt the same way. Still failing → `needs you`: leave the changes in the tree, list them,
and quote the failure. Undoing them is my call.

## Step 5 — Tick it off

Edit only the finding's checklist line, keeping its text and appending the outcome:

- `- [x] **03** · … — fixed`
- `- [x] **03** · … — not a defect: <the path:line that shows it>`, or `: closed by you`
- `- [ ] **03** · … — needs you: <the question, in one line>`

A `together with` pair gets both lines. A `needs you` line worked again has its suffix replaced.

## Step 6 — Report

In chat, under 150 words: the finding and its outcome; for a fix, the files changed and the `VERIFY`
outcome; for `not a defect`, the code that refuted it; for `needs you`, the question. Then what's
next — re-run the checklist grep for the next open finding and how many remain — and any fact worth
adding to `.claude/project.md`.

## Where this skill stops

One finding per run, its `together with` partner aside. It doesn't run the project's full gate,
doesn't commit, and in the findings document it only ever edits that finding's checklist line. It
fixes what the finding is about and nothing past it — anything else it notices goes in the report,
for a review to pick up.
