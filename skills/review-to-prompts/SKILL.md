---
name: review-to-prompts
description: Review TypeScript source under src/ and e2e/ in a single read pass and write a code-review folder containing incremental findings plus one self-contained fix prompt per work item. Use when the user asks for a code review that produces actionable fix prompts rather than direct edits.
argument-hint: "[optional subpath to narrow scope, e.g. src/api]"
allowed-tools: Read, Glob, Grep, Write, Bash(git rev-parse:*), Bash(git status:*), Bash(find:*), Bash(du:*), Bash(wc:*), Bash(ls:*), Bash(mkdir:*)
model: sonnet
disable-model-invocation: true
user-invocable: true
---

# Inventory

- Repo root: !`git rev-parse --show-toplevel 2>/dev/null || pwd`
- Commit: !`git rev-parse --short HEAD 2>/dev/null || echo "not a git repo"` on !`git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "n/a"`
- Uncommitted: !`git status --porcelain 2>/dev/null | wc -l` file(s)
- Config present: !`ls -1 package.json tsconfig.json eslint.config.* .eslintrc* vitest.config.* jest.config.* playwright.config.* cypress.config.* 2>/dev/null || echo "none found at root"`

Scope rules live in exactly one place — the single traversal below. Change them here and
nowhere else.

!`find src e2e -type f \( -name '*.ts' -o -name '*.tsx' \) ! -name '*.d.ts' ! -path '*/node_modules/*' ! -path '*/dist/*' ! -path '*/build/*' ! -path '*/coverage/*' ! -path '*/__generated__/*' -exec du -k {} + 2>/dev/null | awk -F'\t' 'BEGIN{print "REVIEW SET (traversal order):"} $1<40 {n++; t+=$1; print "  " $2; next} {b++; big[b]=$2} END{printf "\nTOTALS: %d files, ~%d KB\n", n, t; printf "OVERSIZED, excluded and unread: %d\n", b; for(i=1;i<=b;i++) print "  " big[i]}' || echo "TRAVERSAL FAILED — do not guess; report this and stop."`

Sizes come from `du`, which reports disk blocks, so the KB total rounds up and slightly
over-states actual bytes. That is the safe direction for a budget check — treat it as a
ceiling, not a measurement.

# Task

Review the file set above and write `code-review/`.

**Scope narrowing:** if `$1` is non-empty, discard every listed path that does not start with
`$1`, and say so in the report. Otherwise review the full set. Recount the files yourself after
narrowing; the TOTALS line covers the unnarrowed set.

**JSON, snapshots, `.d.ts`, and files of 40 KB or more are deliberately excluded.** They are
token-dense and finding-sparse. Do not read them. The oversized ones are listed above — carry
that list into the report as unreviewed.

## Cost rules — these override any instinct toward thoroughness

1. **Every file is read exactly once, by exactly one agent.** Never re-read a file. Capture the
   line numbers and snippets you need at read time, in the findings file, because you will not
   get a second look.
2. **Iterate over files, applying all lenses to each** — never over lenses, re-scanning the
   tree per lens. This is the single most important rule in this document.
3. **Do not use the Task tool or the Explore agent in Solo mode.** No exploratory globbing
   beyond the inventory above; it is already complete.
4. **No verification re-read pass.** Line references are captured during the single pass.
5. If you want more context than the inventory provides, record it as an open question rather
   than going to fetch it.

## Step 1 — Plan, then stop

Read only `package.json`, the TS config, and the lint/test config listed above. Then print:

```
PLAN
  Files to read:      <n>            Total: <k> KB
  Excluded oversized: <n>
  Mode:               Solo | Split (<n> workers)
  Model:              <current model>
  Estimated reading:  ~<k * 0.3>k tokens of file content, single pass
  Output:             code-review/
```

Mode selection, from the TOTALS line (after any `$1` narrowing):

| Review set | Mode |
|---|---|
| ≤ 120 files **and** ≤ 500 KB | **Solo** — you read everything yourself, no subagents |
| larger | **Split** — partition the file list into **at most 3 disjoint chunks** and dispatch one subagent per chunk, each receiving its explicit file list, the checklist below, and these cost rules |
| > 2 MB | **Abort.** Report the size and tell the user to narrow scope with `$1`. Do not attempt it. |

Solo is the expected mode for a normal project. Chunks in Split mode never overlap — a file
belongs to exactly one worker.

**Then stop and wait for the user to confirm.** Do not read a single source file, and do not
create any directory or file, until they reply. If they say nothing, do nothing.

## Step 2 — Single pass

Create `code-review/` and immediately write `code-review/findings.md` containing the plan
header and an empty findings list.

Then walk the file list in order. For each file, read it once and evaluate it against the whole
checklist:

- **Correctness** — logic errors, off-by-one, wrong operators, unhandled null/undefined,
  unawaited promises, incorrect async/await, race conditions, resource leaks.
- **Security** — injection, unvalidated input, authn/authz gaps, hardcoded secrets, unsafe
  deserialization, path traversal, permissive CORS.
- **Resilience** — swallowed exceptions, missing timeouts and retries, silent failure, missing
  cleanup on error paths, unactionable error messages.
- **Types & contracts** — `any` escapes, unsafe casts, non-null assertions covering real
  nullability, inconsistent interfaces, leaky abstractions.
- **State & data** — shared mutable state, missing transactions, cache invalidation,
  schema/model mismatch.
- **Performance** — N+1 patterns, unbounded loads, repeated work in hot paths, where the
  mechanism is visible in the code.
- **Maintainability** — duplication, dead code, oversized units, misleading names.
- **Test quality** (files under `e2e/`) — assertion-free specs, hard waits instead of proper
  waiting, brittle selectors, order dependence, skipped specs.

**Append to `code-review/findings.md` after every 10 files.** Each finding, one block:

```
- [FILE:LINE] <one-line title> | Severity: Critical|High|Medium|Low | Confidence: high|medium|low | Effort: S|M|L
  Evidence: <the relevant snippet or exact behavior, captured now>
  Why it matters: <the concrete failure mode>
```

Maintain a `Files reviewed:` list at the top of that file and update it on each append. If this
run is interrupted, that file is the deliverable — make it stand alone.

Evidence rules: every finding cites `path:line`. Nothing is reported that was not read. No
style nits the configured linter already handles. Unconfirmable suspicions are recorded at
`Confidence: low` with a note on what would confirm them. A clean file produces nothing.

**Resumption:** if `code-review/findings.md` already exists when you start, read its
`Files reviewed:` list and skip those files.

## Step 3 — Group and write items

From `findings.md` alone — do not revisit source — group findings into work items. One item,
one file. Merge when findings share a root cause, are the same mechanical change repeated, or
would leave the code inconsistent if fixed separately. Split when they touch unrelated
subsystems, differ sharply in severity, or would be reviewed and reverted independently.

Write each item as `code-review/NN-<slug>.md` **one at a time, immediately**, so an interruption
leaves the completed ones on disk:

```markdown
# [NN] <Title>

- **Severity:** <Critical|High|Medium|Low>
- **Confidence:** <high|medium|low>
- **Effort:** <S|M|L>
- **Files:** `path:line`, `path:line`
- **Depends on:** <item IDs, or "none">

## Context
What the code does today, with the snippets captured in Step 2. Written for someone who has
never opened this repo.

## Problem
The concrete failure mode and what it costs.

## Fix prompt
<A self-contained instruction for a fresh Claude Code session with no memory of this review.
Name the files and functions. State the intended behavior after the change, and what must NOT
change: public API, passing tests, formatting, unrelated files. Specific enough that two
engineers would produce substantially the same diff. Write it as an instruction, not a
description of one.>

## Acceptance criteria
- [ ] Checkable outcomes, the exact commands to run, and what passing looks like.
- [ ] Any test that must exist to prove the fix.

## Out of scope
Adjacent temptations, naming the item ID that covers them if one exists.
```

## Step 4 — Overview last

Write `code-review/README.md`: scope and method (paths, commit, date, what was excluded and
why, what you did not run), a one-paragraph project summary, the findings table sorted by
severity then effort, counts by severity, a suggested order of work, and open questions.

Then report in chat, under 150 words: counts by severity, the three items to fix first, and
anything you were blocked on.

## Non-negotiables

- No project source file is modified. No `git add`, `git commit`, or working-tree mutation.
- No fabricated line numbers, paths, or tool output. If you did not run something, say so.
- If you are approaching the end of your context, stop cleanly: finish the current append to
  `findings.md`, write the overview from what you have, and say which files went unreviewed.
  A partial review on disk beats a complete one that never lands.