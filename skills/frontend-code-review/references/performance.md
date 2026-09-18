# Performance

## Threshold — read this first

Report a performance finding only when one of these holds:

- **A complexity regression is visible in the code.** Work that grows with data
  size where it previously did not, or nested iteration over the same
  user-scaled collection.
- **A resource is unbounded.** A list, a cache, a subscription, or a poll with
  no ceiling and no termination.
- **A leak is provable.** A subscription, timer, listener, or object URL created
  without a matching release.
- **A measurement exists.** A benchmark, a bundle report, a profile, or a
  reproducible timing the change author or you can point at.

Everything else is premature optimization. `memo`, `useMemo`, `useCallback`, and
virtualization suggested on intuition are noise — and each carries its own cost,
so suggesting them without evidence can make the code worse. When something
looks suspicious but you cannot meet the bar, say what to measure instead of
asserting a problem.

## Contents

- [Bundle](#bundle)
- [Render](#render)
- [Network and data](#network-and-data)
- [Assets and layout](#assets-and-layout)
- [Memory](#memory)

## Bundle

- A new dependency added for something small the platform or an existing
  dependency already does. *Check: is it already in the lockfile, and what does
  the package weigh?*
- A heavy library imported at the top of a route that most users never reach.
  *Check: is there a dynamic import boundary, and did this change cross it?*
- A deep import flattened into a barrel/index import, pulling in siblings.
  *Check: build and compare chunk sizes; do not assume.*
- Route-level or component-level code splitting removed by this change.
- A polyfill or locale bundle imported wholesale for one function.

## Render

- A list rendered without a bound where the data can grow with user activity.
  *Check: what caps the length? If nothing does, that is the finding.*
- A derived computation that is O(n²) over a user-scaled collection — a nested
  `find`/`filter`/`includes` inside a `map`.
- A context value or provider prop rebuilt every render, re-rendering every
  consumer. *Check: how many consumers, and how often does the provider render?*
- State placed high in the tree that only a deep subtree uses, so unrelated
  siblings re-render.
- Layout read and written in the same frame (reading `offsetHeight`/
  `getBoundingClientRect` then setting style), forcing synchronous reflow.
- A scroll, resize, mousemove, or input handler doing real work every event with
  no throttle, debounce, or rAF.
- An animation driven by state updates per frame rather than by CSS or a
  transform.

## Network and data

- Independent requests awaited in sequence that could run in parallel.
- A request chain where a later request's parameters do not depend on the
  earlier response — a waterfall with no reason.
- A request inside a list item, so rendering N items issues N requests.
- A poll interval short enough to matter with no termination condition.
- A response over-fetched and then filtered client-side when the request could
  narrow it.
- Cache configuration that forces a refetch on every mount for data that rarely
  changes, or the reverse — a long stale window with no write-through, which is
  a correctness issue as much as a performance one.

## Assets and layout

- An image served at full resolution into a small box, or in a format with no
  modern alternative.
- Below-the-fold media without `loading="lazy"`.
- Media or an embed with no `width`/`height` or `aspect-ratio`, shifting layout
  on load.
- A web font loaded without a `font-display` strategy or a real fallback stack.
- A font family or weight added where one is already loaded.

## Memory

Missing cleanup on a listener, timer, subscription, or object URL belongs to the
correctness persona, not this one. Report only unbounded growth:

- A `Map`/`Set` cache keyed by something with no ceiling
- A log, history, or undo stack that never truncates
- Data accumulated across renders that is never released
