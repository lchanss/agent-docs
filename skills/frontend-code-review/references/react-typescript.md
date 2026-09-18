# React and TypeScript

Checklist items for React 19, TypeScript 5, and TanStack Query 5. Items only.

Before applying any section, confirm the library is actually in `package.json`
at a version where the item applies. Do not flag a React 19 item in a React 18
codebase, and do not suggest an API the installed version lacks.

## Contents

- [Hooks](#hooks)
- [Rendering](#rendering)
- [Effects](#effects)
- [React 19 specifics](#react-19-specifics)
- [TanStack Query 5](#tanstack-query-5)
- [TypeScript](#typescript)
- [Component shape](#component-shape)

## Hooks

- State holding a value that could be computed during render from existing
  props/state
- A custom hook that returns a different shape depending on its arguments
- Sibling hooks in the same folder disagreeing on return convention (object vs
  tuple, query object vs `data`)
- A hook that both owns state and accepts a setter for that same state from its
  caller — ownership is split and the two can diverge
- A `ref` read or written during render
- Initial state computed by an expensive call passed directly rather than as a
  lazy initializer

**The `optionsRef` pattern** — a ref holding the latest callbacks, refreshed on
every render, so consumers need not memoize:

```tsx
const optionsRef = useRef(options);
useEffect(() => { optionsRef.current = options; });   // no dep array: every render
```

Legitimate when the callbacks are read only from event handlers, timers, or
subscriptions. A defect when the ref is read during render or inside an effect
that runs before the refresh — the value is then one render stale. Check which
one this is before flagging or approving.

## Rendering

- Array index as `key` on a list that can reorder, filter, or splice
- `key` derived from a value that is not unique among siblings
- A side effect (mutation, logging, navigation, `Date.now()` into state) during
  render
- A new object, array, or inline function passed to a child wrapped in `memo`,
  defeating it
- A component defined inside another component's body, so it remounts every
  render and loses state
- Conditional rendering with `&&` on a number, rendering a literal `0`

## Effects

- An effect whose only job is to copy props or state into other state
- An effect used to respond to a user event that could be handled in the handler
- An effect that fetches data a query library already manages
- A dependency array that omits a value the effect reads
- A dependency array padded to silence the linter rather than fix the closure
- An effect whose cleanup does not undo everything its setup did: a listener,
  timer, subscription, object URL, observer, or socket left open
- An effect not idempotent under StrictMode's double invoke in development
- A state updater function with side effects inside it (it may run twice)

## React 19 specifics

- `use()` called outside a component or hook, or in a code path that cannot
  suspend
- `forwardRef` retained where `ref` can now be a plain prop
- An Action or `useActionState` form that also duplicates manual pending state
- `useOptimistic` without a matching failure path
- A Server/Client boundary crossed by a non-serializable prop (if the app uses
  RSC at all)

## TanStack Query 5

- A `queryKey` written as an inline literal at the call site instead of
  referencing the shared options object's key, so invalidation can drift
- `invalidateQueries` after a mutation that misses one of the keys the mutation
  affects
- A long or infinite `staleTime` without a corresponding `setQueryData` on
  write — the cache then serves a value the server no longer has
- `setQueryData` writing a shape that differs from what the query function
  returns
- `enabled` guarding a query in only some of its call sites rather than inside
  the hook that owns it
- `refetchInterval` with no terminal condition, polling forever
- `isLoading` used where `isPending` or `isFetching` is meant
- A mutation with no `onError`, leaving the UI stuck or silently wrong
- Query options constructed inline in a component rather than in the shared
  options module the repo uses

## TypeScript

- A type-only import not marked `import type` where the project sets
  `verbatimModuleSyntax`
- `enum` where the project sets `erasableSyntaxOnly` or prefers `as const` objects
- An exported type whose name collides with an exported value of different meaning
- A wire/DTO type used directly in components instead of the domain type, so the
  transport format leaks upward
- A numeric or string literal union whose values must match an external contract
  (DB column, API enum) declared in more than one place
- Optional properties used where a discriminated union expresses the states
- `Function`, `object`, or `{}` as a type
- A generic parameter that appears exactly once, which means it is not generic

## Component shape

- A component taking 3+ boolean props that encode mutually exclusive modes
- A prop named for how it is implemented rather than what it does
- Required props with defaults applied inside the body instead of in the signature
- A component that renders nothing for a state it can actually be in
- Business logic in JSX that the surrounding module could name
