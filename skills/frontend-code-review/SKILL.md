---
name: frontend-code-review
description: >
  Reviews React/TypeScript frontend code against readability, predictability,
  cohesion and coupling criteria plus correctness, security, accessibility and
  performance. Runs specialist sub-agents in parallel, verifies every candidate
  finding against the real source before reporting, and emits only what it could
  confirm. Use for frontend pull request review, pre-commit self review of a
  branch diff, or auditing existing components, hooks and pages. Triggers on
  requests to review a PR, review a diff, audit frontend code, or judge whether
  a frontend change is ready to merge. Korean requests trigger it too: 프론트엔드
  리뷰, 프론트 코드 리뷰해줘, 프론트 쪽 좀 봐줘, 컴포넌트 리뷰, 훅 리뷰,
  이 화면 코드 봐줘, 리액트 코드 점검, 프론트 PR 리뷰.
  Not for backend, infrastructure, database, or CI-only changes, and not for
  non-web code; on a mixed change it reviews the frontend portion only.
---

# Frontend Code Review

Reviews React/TypeScript code and reports confirmed findings. Does not approve,
reject, or merge anything.

**Scope gate.** If the target contains no frontend code (backend, infra, docs
only), say so and stop — another reviewer fits better. If the target is mixed,
review the frontend portion and name what you skipped.

**Report language.** Write the report in the language the user wrote their
request in, not the language of these instructions. If the request gives no
signal (a bare PR number, a path), follow the codebase — the language of its
comments, UI strings, and commit messages. Korean when neither settles it.

## Principles

These outrank everything below. When a later step and a principle disagree, the
principle wins.

1. **Never approve or request changes.** `gh pr review --approve` and
   `--request-changes` are forbidden. Comments only; the merge decision is the
   author's.
2. **Silence is the default.** No findings is a valid and common outcome. Volume
   is not value — most AI review comments are noise, and this skill exists to
   not be that.
3. **The author knows the codebase better than you.** A pattern used
   consistently is a decision, not a mistake.
4. **Every finding needs a failure scenario**: a concrete input or sequence that
   produces a wrong result. For a readability finding, a concrete reason the
   code misleads a reader. If you cannot describe one at all, drop the finding.
   If you can describe one but cannot reach it from the source you have, the
   finding survives as `PLAUSIBLE` — say what would confirm it. What never
   survives is a finding with no scenario, only a feeling.
5. **Every suggestion states its trade-off** — what improves and what gets
   worse. If nothing gets worse it is a defect, so say that plainly instead.
6. **Judge the code, not the author.** Describe the code's behavior. No praise
   or criticism of the person, no speculation about intent.

**Never report:**

- Formatting a formatter owns, or rules a configured linter already enforces
- Design-system values, tokens, or anything from an external contract
- Repo conventions used consistently, even when you would choose differently
- A small helper colocated in the only file that uses it
- File placement preference with no stated cost
- "Could be cleaner" with no named defect
- Performance or memoization without evidence (see `references/performance.md`)
- Something already documented as a known issue, unless this change spreads it.
  In audit mode there is no "this change", so report a documented issue only
  when the documentation understates it — the tracked note describes a smaller
  problem than the code actually has. Say that plainly and cite the note.

## Modes

Determine the mode from the arguments. Say which mode you chose in the report.

| | **diff mode** (default) | **audit mode** |
| --- | --- | --- |
| Invoked by | `#123`, a PR URL, or no argument | a path or glob (`src/features/editor/`, `**/hooks/*.ts`) |
| Target | changed lines, read in whole files | the named files entirely |
| Pre-existing problems | only what this change introduced or worsened | all in scope |
| Caps | nits ≤ 3, praise ≤ 1 | ≤ 3 per file, ≤ 15 total, praise ≤ 1, severity-ordered |
| Automated checks | run them | skip (nothing changed) |
| `--comment` | allowed | not allowed |

In audit mode:

- **Group findings by shared root cause** rather than listing them flat, and
  give each group a rough size ("one file", "touches every page"). A list of 15
  unrelated items does not get acted on. The per-file cap counts individual
  findings, not groups.
- **Consequences may be traced outside the target.** If a defect in a target
  file only becomes serious at a caller outside the named scope, cite that
  caller — it is what makes the finding real. The *defect* must still be in a
  target file.

## Workflow

Copy this checklist into your response and check items off as you go.

```
Review progress:
- [ ] 1. Resolve target and mode
- [ ] 2. Gather context
- [ ] 3. Size the review
- [ ] 4. Run persona agents in parallel
- [ ] 5. Verify every candidate; discard what you cannot reproduce
- [ ] 6. Run automated checks (diff mode only)
- [ ] 7. Emit report
```

### Step 1 — Resolve target and mode

**PR number or URL** → diff mode.

```bash
gh pr view <n> -R <owner/repo> --json title,body,author,headRefOid,files
gh pr diff <n> -R <owner/repo>
```

**Commit SHA, tag, or range** → diff mode against that commit's parent.

```bash
git show <sha> --stat
git show <sha>              # or: git diff <a>...<b>
```

Read post-change files with `git show <sha>:<path>` when the working tree has
moved on since.

**Path or glob** → audit mode. Expand it with Glob and confirm the file count
before reading.

**No argument** → diff mode on the current branch.

```bash
git symbolic-ref --quiet --short refs/remotes/origin/HEAD   # fall back to "main"
git diff <base>...HEAD --stat
git diff <base>...HEAD
```

If that diff is empty, review uncommitted work: `git diff HEAD`. If that is
empty too, ask the user what to review rather than guessing.

### Step 2 — Gather context

Reviewing a diff without its surroundings is the largest single source of false
findings. Do all of this before any analysis.

- **Read whole files, not hunks.** For every target file, Read the file. A hook
  cannot be judged without seeing its callers.
- **Read `CLAUDE.md`** at the repo root, plus any nested one covering the target
  directory.
- **Read `.claude/review-notes.md` if it exists.** This is the project's
  extension point and it outranks the generic checklists in `references/`.
  Skip quietly when absent.
- **Read `package.json`.** Confirm framework, library versions, and available
  scripts. Never apply a checklist item for a library the project does not use,
  or an API its version does not have.
- **Check the linter config** (`eslint.config.*`, `.eslintrc*`). Anything it
  enforces is the linter's job, not yours.
- **Trace usage.** Grep for every symbol the change adds, renames, or changes
  the signature of, and read the call sites.
- **diff mode: read the intent.** If the PR body or a commit says `Closes #N`,
  run `gh issue view N`. *Whether the change actually accomplishes the stated
  intent* is itself a review item, and a mismatch is usually the most valuable
  thing a review can surface.

### Step 3 — Size the review

**Small** — ≤ 3 files **or** ≤ 150 lines, counting changed lines in diff mode
and total file lines in audit mode. Skip the agents: walk the five axes
yourself, reading the reference files as you go. Five agents on a small target
costs more in coordination than it returns.

**Larger**: go to step 4.

### Step 4 — Run persona agents in parallel

Send **all five Agent calls in a single message** so they run concurrently. Use
a read-only sub-agent type (`Explore`).

If sub-agents are unavailable — no Agent tool, or this skill is itself running
inside one — walk the five axes yourself in the order of the table below,
reading each axis's references as you reach it. Say in the report that the
review ran sequentially.

| # | Persona | Looks for | Reads |
| --- | --- | --- | --- |
| 1 | 🐛 Correctness & safety | runtime errors, logic faults, races, state drift, missing error handling, type holes, XSS, credential exposure | `references/correctness-and-security.md`, `references/react-typescript.md` |
| 2 | 📖 Readability & predictability | context load, naming, top-to-bottom flow, inconsistent return shapes, hidden side effects | `references/code-quality.md` (axes 1–2) |
| 3 | 🧩 Cohesion & coupling | colocation, single responsibility, props drilling, wrong abstractions, layering violations — maintainability and extensibility live here | `references/code-quality.md` (axes 3–4) |
| 4 | ♿ Accessibility & UX states | semantics, names, keyboard and focus, loading/empty/error coverage | `references/accessibility-ux.md` |
| 5 | ⚡ Performance | complexity regressions, unbounded resources, leaks, bundle cost | `references/performance.md` |

Write the diff to a file first and pass agents the path, not the text. Pasting a
large diff into five prompts wastes it five times over.

Give every agent all five of these. A persona missing its references or the
project conventions reviews against the wrong rules.

1. The mode (diff or audit)
2. The target file list
3. The diff path (diff mode) or the file paths (audit mode)
4. The project conventions from step 2, including anything in
   `.claude/review-notes.md`
5. The reference paths it must read first, and the output contract below

A **candidate** is what an agent submits. A **finding** is what survives step 5
and reaches the report. The two words mean different things throughout this
skill; do not use them interchangeably.

**Each agent returns at most 3 candidates.** Fewer is better; zero is fine.
Each candidate must carry all of:

```
- file:line
- severity: blocking | important | nit
- claim: one sentence, the defect itself
- failure scenario: concrete input or state -> wrong outcome
                    (for a readability nit: why a reader is misled)
- evidence: the exact lines the claim rests on
- trade-off: required for any suggestion; omit only for objective defects
```

Tell each agent, in these words:

- **Zero candidates is the expected answer.** Three is the ceiling, not a quota.
  An axis with nothing to report is the normal case, not a failed pass.
- **If you cannot write the failure scenario, do not submit the candidate.**
  This rule removes most false positives before they reach the orchestrator.
- **Do not invent a hypothetical confused reader.** A readability finding needs
  something demonstrably misleading: a name that contradicts what the code does,
  a comment that contradicts the code, a control flow that cannot be followed
  without opening another file. "Someone might find this confusing" is not
  evidence.
- You are reviewing **only your axis**; other agents cover the rest.
- You must not report anything on the "Never report" list.

### Step 5 — Verify

This step is what separates a useful review from a noisy one. Do not skip it,
and do not pass an agent's output through unexamined.

You verify against **the source, the project's own rules, and the "Never report"
list** — not against the reference checklists, which are the agents' business.
Open a reference file only when a candidate's validity actually turns on what it
says.

For each candidate:

1. **Open the file at that line.** If the quoted code is not there, discard.
2. **Walk the failure scenario through the real code.** If you cannot trace a
   path from the input to the wrong outcome, discard.
3. **Check for precedent.** If the same pattern appears elsewhere in the repo
   and works, it is a convention. Discard, or reframe as a repo-wide question
   rather than a finding against this change.
4. **Check the project's own rules.** If `CLAUDE.md` or `.claude/review-notes.md`
   permits it, discard.
5. **diff mode: check attribution.** If the problem predates the change and the
   change did not worsen it, discard. Worsening includes reach: untouched code
   whose defect this change made newly reachable, or newly likely to be hit, is
   in scope. Say which it is in the finding.
6. **Merge duplicates.** Several agents finding the same thing is one finding.
7. **Label what survives.** `CONFIRMED` when you traced the failure.
   `PLAUSIBLE` when the reasoning holds but you could not confirm it from source
   alone — say what would confirm it. Put the label inline, right after the
   failure scenario, at every severity.

Report the discard count **with a one-line reason for each significant
discard**. The number alone is not auditable; the reasons are what show the
verification happened, and they are what tells you when a threshold needs
tuning.

### Step 6 — Automated checks (diff mode only)

Run the scripts that exist in `package.json`, in this order, and stop at the
first that fails:

```
lint -> test -> typecheck/build
```

Skip any that is absent. If CI already runs them on this PR (`gh pr checks`),
read the result instead of re-running.

A failure is a blocking finding on its own. Quote the relevant output rather
than summarizing it. Do not fix anything — this skill reviews.

### Step 7 — Emit report

Severities: 🔴 blocking / 🟠 important / 🟡 nit / 🌟 praise.

A sensible default shape; adapt it to what you actually found. Omit empty
sections rather than printing them empty. In audit mode, keep the severity
headings but put grouped findings under a shared sub-heading that names the root
cause and its size, as in `### 🔴 머지 전 수정 (2): 한 뿌리, <원인>`, so the
reader sees one problem rather than several.

Write Korean output without em-dashes. Use a comma, a colon, or a period where
an English draft would reach for `—`.

~~~markdown
## 리뷰: <target>

**범위**: 6 files, +212 −178 (diff mode)
**자동 검사**: lint ✅ / test ✅ (5 passed) / build ✅

### 🔴 머지 전 수정 (1)

**1. <one-sentence claim>**
`path/to/file.ts:34`

<failure scenario: the concrete input and the wrong outcome> (CONFIRMED)

```ts
// 제안
```

### 🟠 고려할 것 (2)

**2. <claim>**
`path/to/other.tsx:88`

<why it matters>
> 트레이드오프: <what improves> / <what gets worse>

### 🟡 사소한 것

- `file.ts:12` — <claim>

### 🌟 좋았던 점

- <one specific thing, only if genuine>

### 규약 점검

- [x] <convention from CLAUDE.md or review-notes>
- [ ] <one that is not met>

---
후보 11건 중 4건 확정, 7건은 검증 단계에서 폐기.

- `<폐기한 지적>`: <한 줄 사유>
- `<폐기한 지적>`: <한 줄 사유>
~~~

If nothing survived verification, say so directly and give the discard count.
Do not pad the report with observations to look thorough.

## `--comment`

Only with this flag may you write to GitHub. Read
`references/posting-comments.md` and follow it. Blocking and important findings
go inline; nits and praise stay in the chat report. Approval remains forbidden.

## Project extension point

A repository can steer this skill by adding `.claude/review-notes.md`: the
invariants that must not break, patterns that look wrong but are intentional,
layering rules, known issues not to re-report, and constraints that keep
suggestions actionable. Step 2 reads it automatically and it outranks the
generic references.

`assets/review-notes-template.md` is the starting point. Offer it when a review
turns up project knowledge that was not written down anywhere.
