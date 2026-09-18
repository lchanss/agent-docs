# Review Notes

Copy this file to `.claude/review-notes.md` in a repository. The
`frontend-code-review` skill reads it during context gathering and treats it as
authoritative — above its own generic checklists.

Write only what a competent reviewer could not infer from the code. Keep it
short; every line is read on every review. Delete sections you do not need.

---

## Invariants

Contracts that must not break, and why. State the failure, not just the rule —
a reviewer who knows the consequence catches more cases.

- `<thing>` must `<hold>`. If it does not, `<concrete failure>`.
  See `<path/to/file.ts>`.

## Intentional patterns

Things that look wrong but are deliberate. Prevents the same false positive
every review.

- `<pattern>` in `<location>` is intentional: `<reason>`.

## Layering

Which module may import which, and where a given kind of code belongs. Flag
violations; do not re-litigate the layering itself.

- New code goes in `<layer>`, not `<deprecated layer>`.
- `<transformation>` happens only at `<boundary>`.

## Known issues

Already tracked. Do not re-report. Do flag changes that spread them further.

- `<issue>` — tracked in `<#N or link>`.

## Constraints

What this repo cannot do yet, so suggestions stay actionable. Without this a
reviewer will ask for tests the test setup cannot run.

- `<constraint>`, so `<what a reviewer should not ask for>`.

## Conventions

Only the ones a reviewer should enforce. Anything a formatter or linter handles
does not belong here.

- Comments and UI copy: `<language>`
- Commit titles: `<format>`
- PR titles: `<format>`
