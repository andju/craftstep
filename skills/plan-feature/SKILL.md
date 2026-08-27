---
description: Plan a new feature — interview, flag issues, check tests and docs
argument-hint: [short description of the feature]
allowed-tools: Read, Grep, Glob, AskUserQuestion, Write
---

We are planning a new feature: $ARGUMENTS

If that's empty, ask what we're building before anything else.

**Before anything else, confirm you can write.** If Plan Mode is active,
this command can't finish — the final write to `SPEC.md` will be blocked.
Tell me to exit Plan Mode (Shift+Tab) and rerun the command, and stop here.
Don't run the interview first and discover this at the end.

**If `SPEC.md` already exists** and covers the same feature, this is a
revision pass: read it — including any annotations or edits made directly in
the file — and rewrite it incorporating them. Ask only about what those
changes left ambiguous. Don't re-interview on settled points.

Otherwise, read enough of the codebase to ground your questions before
asking them.

**First, interview me.** Cover requirements, edge cases, and tradeoffs. Ask
open questions as plain text; use AskUserQuestion for genuine either/ors.
Don't ask me to confirm what I've already told you — dig into the hard parts
I might not have considered. Keep going until you could implement this
without guessing.

**Then work through this checklist:**

1. **Push back.** If any part is impossible or conflicts with how the system
   works today, say so. If there's a better route to the same goal, propose
   it with the tradeoff.

2. **Tests.** Which tests, and where they belong in the existing suite —
   `tests/unit`, `tests/integration`, or `tests/e2e`; check existing specs in
   each for the boundary between them. If you conclude none are warranted,
   justify it.

3. **Documentation.** Which file(s) in `docs/user/` change, or whether a new
   page is needed and where it links from. If purely internal, say so
   explicitly.

4. **Component reuse.** Check `src/lib/components/` for existing components
   the feature could build on instead of new markup. For each candidate, say
   what prop or variant it would need to support the new case; only propose a
   new component when nothing existing fits.

**Finally, write the plan to `SPEC.md`** (overwriting any existing one)
covering: the feature and its requirements, the implementation approach,
files you expect to touch, the tests, the docs changes, and the existing
components to reuse or extend (with the props/variants each needs). That
file only — no docs, no implementation. Then summarize it here and wait for
my review.
