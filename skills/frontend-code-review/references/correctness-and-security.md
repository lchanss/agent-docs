# Correctness and Security

Checklist for the correctness persona. Items only — the reasoning is already
known; what a review needs is coverage.

Threshold: a correctness finding must name a concrete input or sequence that
produces a wrong result. "Might be a problem" is not a finding.

## Contents

- [Null and optional paths](#null-and-optional-paths)
- [Async and ordering](#async-and-ordering)
- [State and closures](#state-and-closures)
- [Data and arithmetic](#data-and-arithmetic)
- [Error handling](#error-handling)
- [Type holes](#type-holes)
- [Security](#security)

## Null and optional paths

- Optional chaining that silently produces `undefined` where a value is required
  downstream
- A default (`?? []`, `|| 0`) that hides a real failure instead of handling it
- Array access by index without a length check; `find`/`at` results used without
  a guard
- A destructured field from a response the API can omit
- Empty-string, `0`, and `false` treated as absent by `||`

## Async and ordering

- Two requests that can resolve out of order writing the same state; later
  request's result overwritten by the earlier one
- `fetch` in an effect with no abort or ignore flag on cleanup
- An `await` inside a loop that serializes independent work
- A floating promise whose rejection nothing handles. `void p` only silences the
  linter; it attaches no handler. Look for an actual `.catch`, a `try` around an
  `await`, or a handler registered on the same promise elsewhere
- Concurrent writes to the same resource with no queue or single-flight guard
- A write path that can run before the read it mirrors has resolved, so a
  placeholder or default is persisted over real data (auto-save, sync, an
  optimistic write against a not-yet-loaded record)
- A debounced or throttled call whose captured payload is read at execution time
  rather than at call time
- `Promise.all` where one rejection should not abort the rest (`allSettled`)

## State and closures

- A handler or timer callback capturing a stale value because it was created in
  an earlier render and never refreshed
- State updated from a previous value without the functional form
- Two state updates that must land together but are issued as separate
  transitions, so an intermediate state is observable or undo splits them
- Derived data in two places that can drift apart, with no single owner

## Data and arithmetic

- Off-by-one in slice/range boundaries; inclusive vs exclusive end confusion
- Float equality; money or duration accumulated in floats
- Unit confusion at a boundary (seconds vs milliseconds vs frames); no single
  conversion point
- `Date` parsed from a string without a timezone; local time compared to UTC
- Sorting with the default comparator on numbers
- Mutating an array or object that is also held as state or props

## Error handling

- A `catch` that swallows without logging, reporting, or surfacing
- `catch (e)` where `e` is used as if it were an `Error` without a check
- An error path that leaves the UI in a loading state forever
- A user-facing message that leaks internal details, or a raw message shown
  where a written one belongs

## Type holes

- Newly introduced `any` (explicit or via an untyped third-party import)
- `as` assertions, especially `as unknown as T`
- Non-null `!` on a value the code cannot guarantee
- Two booleans encoding three valid states; a discriminated union fits
- A union that is never narrowed before member access
- A function whose declared return type is wider or narrower than what it
  actually returns
- `@ts-ignore` / `@ts-expect-error` without a comment saying why

## Security

Frontend-relevant only. Do not speculate about backend controls you cannot see.

- `dangerouslySetInnerHTML` (or `innerHTML`) fed by anything not provably
  constant; sanitization missing or applied after interpolation
- A URL from data placed in `href`/`src` without a scheme check — `javascript:`
  and `data:` are the ones that matter
- `target="_blank"` without `rel="noopener noreferrer"`
- Credentials or tokens in a query string, and therefore in logs, referrers, and
  browser history
- Tokens or personal data written to `localStorage`/`sessionStorage` when a
  shorter-lived store would do
- A secret behind a client-exposed env prefix (`VITE_`, `NEXT_PUBLIC_`, `REACT_APP_`)
- `postMessage` without an origin check; a message listener that trusts
  `event.data` shape
- A regex built from user input, or one with nested quantifiers applied to user
  input (ReDoS)
- Redirect target taken from a query parameter without an allowlist
- An API response rendered as markup rather than text

Skip: anything the repo has already documented as a known issue. Note only if
this change spreads it further.
