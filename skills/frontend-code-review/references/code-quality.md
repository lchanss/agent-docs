# Code Quality Criteria

Four axes for judging whether code is **easy to change**. Adapted from Frontend
Fundamentals (frontend-fundamentals.com/code-quality) for review use: each rule
is a judgment question plus the line where flagging becomes noise.

## Contents

- [How to use this file](#how-to-use-this-file)
- [1. Readability](#1-readability) — reducing context, naming, top-to-bottom flow
- [2. Predictability](#2-predictability) — name collisions, return types, hidden logic
- [3. Cohesion](#3-cohesion) — colocation, single source of truth, form structure
- [4. Coupling](#4-coupling) — single responsibility, allowed duplication, props drilling
- [Trade-off rules](#trade-off-rules)

## How to use this file

Read the axis you were assigned. For each rule, ask the **Ask** question against
the target code. If the answer is yes, check the **Skip** line before writing it
up — most false positives die there.

**Evidence bar.** These axes are the easiest place to produce confident-sounding
noise, because a hypothetical confused reader can be imagined for any code. So a
finding here needs something checkable:

- a name that contradicts what the code does
- a comment that contradicts the code
- a control flow that cannot be followed without opening another file
- a shape that disagrees with its own siblings in the same folder
- a value or rule duplicated where changing one and missing the other is the
  normal failure

If your reasoning reduces to "a reader might be confused", you have taste, not a
finding. Return nothing. Zero findings on these axes is a common and correct
outcome.

These axes conflict with each other by design. Duplication can be better than a
wrong abstraction; cohesion can increase coupling. When two rules disagree, say
so in the finding instead of picking one silently.

---

## 1. Readability

### 1.1 Reduce context

**Separate code that does not run together**

- Ask: does one component branch on a role/flag/mode and then carry both
  branches' state, effects, and markup in the same body?
- Skip: a single ternary in JSX with no branch-specific effects or state.

**Abstract implementation details**

- Ask: does a high-level handler expose low-level mechanics (raw `fetch` with
  headers, `localStorage` keys, `window.location` assignment) inline?
- Skip: a one-off call in a leaf module whose whole job is that mechanic.

**Split functions that merge unrelated logic types**

- Ask: does one hook or module return data fetching, UI toggles, and validation
  from the same body?
- Skip: two responsibilities that always change together and are under ~30 lines.

### 1.2 Naming

**Name complex conditions**

- Ask: does a boolean expression combine 3+ clauses inline inside a
  `filter`/`map`/`if`?
- Skip: a two-clause condition, or one the enclosing function name already
  describes.

**Name magic numbers**

- Ask: is a bare numeric or string literal carrying domain meaning (a delay, a
  threshold, a status code, a limit)?
- Skip: `0`, `1`, `-1` used as indices or identity; array indices; values whose
  meaning is stated by the parameter name at the call site.

### 1.3 Read top to bottom

**Reduce timeline shifts**

- Ask: does the body alternate between present-time checks, async work, and
  effects rather than grouping them?
- Skip: ordering forced by the Rules of Hooks.

**Simplify branching**

- Ask: are ternaries nested, does one chain express 3+ outcomes, or does a
  conditional nest 3+ levels deep before the body that matters?
- Skip: a single-level ternary; nesting that mirrors a real decision tree with
  no early-return rewrite available.

**Read left to right**

- Ask: is the subject of a comparison on the right?
  `if (MAX < count)` reads worse than `if (count > MAX)`. Same for a condition
  phrased as a negation of a negation.
- Skip: this is a nit at most. Only raise it when the expression is already
  hard to parse for another reason.

---

## 2. Predictability

**Avoid name collisions**

- Ask: does a local module export a name that shadows a well-known library
  export (`Button`, `Link`, `http`, `useQuery`) and get imported alongside it?
- Skip: a local name that is only ever used locally and never imported next to
  its namesake.

**Unify return types across sibling functions**

- Ask: do functions of the same kind disagree on shape — one hook returning the
  query object and its sibling returning `query.data`, one fetcher returning
  `null` on failure and its sibling throwing?
- Skip: a deliberate difference the name states (`getUser` vs `getUserOrNull`).

**Reveal hidden logic**

- Ask: does a function named for one job also log, track, mutate a cache, or
  navigate, with no hint in its name or signature?
- Skip: cross-cutting behavior the project applies uniformly through a wrapper
  (an `apiFetch` that attaches auth headers everywhere).

---

## 3. Cohesion

**Keep files that change together in the same directory**

- Ask: does the change touch a feature whose parts are scattered across
  type-based folders (`components/`, `hooks/`, `utils/`) so one feature edit
  spans four directories?
- Skip: genuinely shared code used by 3+ features; that belongs in `shared/`.
- Skip: whatever layout the repo already uses consistently. This rule judges
  new structure, not existing structure.

**One source of truth for shared constants**

- Ask: does the same meaningful value appear in 2+ files, so changing it means
  finding every copy?
- Skip: coincidentally equal values with unrelated meanings.

**Match validation to form structure**

- Ask: for interdependent fields (password confirmation, date ranges, totals),
  is validation split per field so no single place sees the relationship?
- Ask: conversely, for independent reusable fields, is validation centralized in
  a form-level blob that prevents reuse?
- Skip: forms with fewer than three fields.

---

## 4. Coupling

**One responsibility per unit**

- Ask: does one hook or component own state for unrelated domains (user + posts
  + modal visibility), so every consumer re-renders on any of them?
- Skip: a container assembled at the page level whose job is composition.

**Allow duplication rather than a wrong abstraction**

- Ask: does a shared helper branch on a `type`/`variant` parameter to serve
  callers that do not actually share behavior? That is coupling, not reuse.
- Ask, before suggesting extraction: is the logic duplicated 3+ times, and is it
  the *same* logic rather than code that merely looks alike today?
- Skip: two occurrences. Two is not a pattern.
- Skip: small helpers colocated in the only file that uses them.

**Remove props drilling**

- Ask: is a prop passed through 3+ components that do not use it?
- Prefer composition (pass children/elements) over context; suggest context only
  when the hierarchy is deep and dynamic.
- Skip: 1–2 levels.

---

## Trade-off rules

Every suggestion must state both sides. Extraction improves reuse and costs
navigation. Context removes drilling and costs explicitness. Splitting a
component improves focus and costs one more file to open.

If you cannot name what gets worse, the suggestion is not ready — either it is
an objective defect (then say so plainly, no trade-off needed) or it is taste
(then drop it).

Size is not a defect. A long file that is a flat list of cases is fine, and no
line count, function length, or nesting depth is a finding by itself. Name the
rule that is broken, not the number.
