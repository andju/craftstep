---
description: Implement the feature, tests and docs from the agreed plan
argument-hint: [optional — which parts to implement, or leave blank for all]
model: sonnet
disable-model-invocation: true
---
Implement the plan in `SPEC.md`.

If `SPEC.md` doesn't exist, stop and say so — run `/plan-feature` first.

Scope for this run: $ARGUMENTS
If that's empty, implement the whole spec. If it names something `SPEC.md`
doesn't cover, stop and ask rather than improvising — the file is scratch and
may be left over from a different feature.

Implement the code, the tests and the docs changes the spec calls for. If
reality diverges from the plan: small corrections are fine, just note them in
your summary; anything that changes the approach or touches files outside the
spec's list, stop and ask first.

Mark items in `SPEC.md` as you complete them so a partial run can be picked up
later.

Before calling this done, run the checks and tests CLAUDE.md requires for
what changed.
