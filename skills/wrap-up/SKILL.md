---
name: wrap-up
description: Close out a feature — move the spec's deferred slices into the backlog, draft the commit message, and delete, archive or leave the spec on my answer. Never commits, pushes or touches code.
argument-hint: "optional — anything the commit message should mention; blank is fine"
allowed-tools: Read, Grep, Glob, Bash(git:*), Bash(rm:*), Bash(mv:*), Bash(mkdir:*), Write, Edit, AskUserQuestion
disable-model-invocation: true
user-invocable: true
---

We are closing out a feature that was planned and implemented. Notes for this run (may be
empty): $ARGUMENTS

**This skill writes one file: the `backlog`.** It deletes or moves one file, the spec, and only on
my answer in step 4. It never touches code, `.claude/project.md` or CLAUDE.md, and never commits,
pushes or opens a PR. Its scope is this feature: findings documents and any other file in the working
tree are not its business — don't read, report on or delete them. If the run turns up a fact worth
recording, propose it once you are done and let me apply it.

## Step 0 — Load this project's conventions

Resolve each fact independently; the first source that settles it wins:

1. **The project's CLAUDE.md** — usually in context; read from disk if not (non-root packages in a
   monorepo).
2. **`.claude/project.md`**, sections `plans`, `backlog`. A missing file, section or key are all
   normal.
3. **The repo itself.**

What each fact is, and how to infer it from the repo:

- **plans** — the file the spec lives in. Default to `.craftstep/spec.md`.
- **after wrap-up** — what happens to the spec once the feature is closed out: `delete`, `leave`,
  or `move to <folder>`. In `.claude/project.md` it is the `After wrap-up:` line of the `plans`
  section. Default to `delete`; don't infer anything else from the repo.
- **backlog** — the single file deferred work is parked in. Default to
  `.craftstep/backlog.md`. Never adopt a TODO or roadmap file in another format.

List the three facts in one short block, each with the source that settled it.

## Step 1 — Take stock

Read the spec once, in full. Nothing at the `plans` path — say so and stop: there is nothing to
wrap up.

Report which work items are marked `[x]` and which are not, in one short block. **Unfinished work
items are a question, not a stop:** ask me once whether to go back (`/craftstep:implement`), or to
wrap up anyway and defer what is unfinished.

## Step 2 — Park the deferred work

What goes into the backlog:

- **Every slice under the spec's `Deferred` heading**, in its order.
- **Unfinished work items I chose to defer** in step 1 — as one entry, carrying the items' own text
  and what the finished items already put in place.

What comes out: the entry this feature implemented — the spec names it under **Feature** as
`Backlog entry: <title>`. Remove it only when every work item is done; if some were deferred, the
new entry replaces it.

**Never read the backlog whole.** Its entries are `## ` headings: grep `^## ` for the titles, and
use a ranged read only for an entry you have to change. An entry with the same title as one you are
adding is updated, not duplicated. New entries go where their dependencies put them: after what they
depend on, otherwise at the end. Create the file, with its two header lines, if it isn't there.

````markdown
# Backlog

Work agreed but not yet specified, in the order it should be built. `/craftstep:plan next` picks up the first entry.

## <slice title>

- **From:** <the feature it was cut from> · <date>
- **Depends on:** <what has to exist first> | nothing
- **Decision record:** <path> | none

What the slice covers, the requirements already settled, and why it was deferred — complete enough
that `/craftstep:plan` can start from this entry alone, because the spec it came from will be gone.
````

Carry each slice across as the spec states it; don't re-scope, merge or improve one. A slice too
thin to plan from is completed from the rest of the spec — its **Feature**, **Approach** and
**Rejected alternatives** — never from guesswork: what the spec doesn't say goes in as an open point
for the planning interview.

Nothing deferred and no entry to remove — say so and leave the file alone.

## Step 3 — Draft the commit message

Print it in chat, in a fenced block.

- **Convention first.** Where CLAUDE.md states a commit convention, follow it. Otherwise take the
  style from `git log --oneline -n 10`.
- **Content** — a subject line for what the change does, then a body with the why, drawn from the
  spec's **Feature** and **Approach**, plus what $ARGUMENTS asked for.
- **Claim only what you saw.** The spec says what was planned, not what was proven in this session:
  don't write "all tests pass" or "meets the acceptance criteria" unless my notes say so.

## Step 4 — Clear the spec

The spec is the only file this skill may delete or move. Say whether git tracks it
(`git ls-files --error-unmatch <path>`), then ask me with AskUserQuestion: **delete**, **leave**,
or **move** — to the recorded folder if `after wrap-up` names one; otherwise I type the folder.

**`after wrap-up` settles which option you recommend and list first, never whether you ask.**
Unset, that is *delete*. One case overrides it: recommend *leave* when the spec has unfinished
work items that step 2 did not park, and say why.

Act only on my answer:

- **Delete** — with `rm`.
- **Move** — into the folder, as `<YYYY-MM-DD>-<feature title in kebab-case>.md`, since every spec
  leaves the same path. Create the folder if it isn't there. Never overwrite a file already in it:
  ask me for another name.
- **Leave** — touch nothing.

**The spec is never deleted or moved before step 2's entries are in the backlog file** — check
with a grep on their titles first. If git tracks it, leave the deletion or move unstaged for me to
commit.

## Step 5 — Report

In chat, under 150 words: backlog entries added, updated and removed, by title; whether the spec
was deleted, moved or left; any work item that was neither finished nor parked; any fact
in `.claude/project.md` or CLAUDE.md worth adding, changing or removing — one this run found
missing, wrong or out of date. Then the next step, if the backlog holds one: `/craftstep:plan next`, in a
fresh session.
