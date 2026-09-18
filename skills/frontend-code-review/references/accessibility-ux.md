# Accessibility and UX States

WCAG 2.1 AA items that are decidable from source alone, plus the user-visible
states a change is expected to handle.

## Baseline rule — read this first

Two checks before writing up any accessibility finding:

1. **Is a linter already covering it?** If `eslint-plugin-jsx-a11y` is
   configured, do not report what it reports. Say the lint run is the check.
2. **What is the repo's current level?** If surrounding pages have no ARIA at
   all, do not open a site-wide audit from a single change. Ask only whether
   **this change lowers the existing baseline** — a new interactive element with
   no accessible name, a new form control with no label, focus handling removed.
   In audit mode this rule is relaxed, but still report the highest-impact items
   rather than every attribute.

Skip anything requiring rendering to judge: contrast ratios from computed
styles, reading order after CSS, zoom and reflow behavior. Say it needs manual
checking instead of guessing.

## Semantics

- A `div` or `span` with `onClick` and no `role`, `tabIndex`, and key handler
- `<button>` used for navigation, or `<a>` used for an in-page action
- A link with no `href`
- Heading levels skipped, or headings chosen for size rather than structure
- A list of items not marked up as a list
- A table used for layout, or a data table with no `<th>` and scope
- `<img>` with no `alt`; decorative images with a non-empty `alt`
- Custom controls rebuilding what a native element provides

## Names and labels

- An input with no associated `<label>`, `aria-label`, or `aria-labelledby`
- A placeholder used as the only label
- An icon-only button with no accessible name
- A decorative icon or emoji without `aria-hidden="true"`
- A generic link name ("여기", "click here") with no surrounding context
- An `aria-label` that contradicts the visible text

## Live state

- A loading indicator with no `role="status"` or `aria-live`
- An error message not connected to its field via `aria-describedby`
- An invalid field with no `aria-invalid`
- A toast or async result announced visually only
- A busy region with no `aria-busy` where the wait is long

## Keyboard and focus

- An interactive element unreachable by Tab
- A focus outline removed with no replacement indicator
- A modal or drawer without focus trap, Esc to close, and focus returned to the
  trigger on close
- A positive `tabIndex`
- DOM order that does not match visual order for a focusable sequence
- A hover-only interaction with no keyboard or focus equivalent
- A dynamically revealed region with focus left behind

## UX states

A change that fetches or submits should handle all three. Missing ones are
findings even when accessibility is fine.

- Loading: is there an indicator, and does it clear on both success and failure?
- Empty: is a zero-length result distinguishable from still-loading?
- Error: is failure visible, and can the user retry or recover?
- Submit: is the control disabled while in flight, so double submission cannot
  happen?
- Optimistic update: is there a rollback on failure?
- Destructive action: is there a confirmation, and does cancelling do nothing?
- Long list: is there a bound on what is rendered, or is it genuinely small?

## Forms

- Validation that fires on every keystroke before first blur
- Errors shown only after submit when they could have been shown at blur
- A submit button whose disabled reason is not stated
- Focus not moved to the first invalid field on failed submit
- An input missing `type`, `inputMode`, or `autoComplete` where it would help
- A required field marked only by color or an asterisk with no text equivalent
