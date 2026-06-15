# TRACK 5 — REACT & NEXT.JS

*Your career core, and the largest track. You already write React for a living — this track is about understanding it from the runtime up, so you stop fighting it and start reasoning about it. Every module here uses React + Tailwind (and Next.js for the framework half), because that is your real stack. The goal: when React "does something weird," you know exactly why, because you understand the render model, reconciliation, and the rules underneath the hooks.*

*Reminder: your root `CLAUDE.md` is loaded automatically by Claude Code. Do the Prep first. Never read the spoiler before you finish or give up.*

---

## Module 5.1 — Components, Props & The Render Model

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** React + Tailwind (Vite)

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Your First Component / Passing Props** — https://react.dev/learn/passing-props-to-a-component — *Focus on: props as read-only inputs, the one-way data flow, and that a component is a function of its props and state.*
- **React docs — Render and Commit** — https://react.dev/learn/render-and-commit — *Focus on: the three steps (trigger → render → commit) and that rendering is React calling your component, not touching the DOM.*
- **React docs — Keeping Components Pure** — https://react.dev/learn/keeping-components-pure — *Focus on: why render must be pure (no side effects, no mutation) and what breaks when it isn't.*

**Optional course resources:**
- Josh Comeau — *The Joy of React* — the "React Fundamentals" and "Rendering" modules.

**Search prompts:**
- `"react render vs commit phase"`
- `"react props read only mutation bug"`
- `"react pure component render side effect"`
- Ask an AI: *"Explain React's render model: what does it mean that a component is a pure function of props and state? What are the trigger/render/commit phases? Why must I never mutate props or do side effects during render, and what subtle bugs appear when I do?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.1 — components, props, and the render
model. Completed so far: Tracks 1–4. I write React professionally, so do not over-explain
JSX — probe my model of HOW React renders.

Build me a small React + Tailwind app (Vite) with bugs rooted in the render model and props:
a component that MUTATES a prop (object/array) and causes spooky behavior; a side effect
performed directly during render (e.g. mutating an external variable, or pushing to an array)
that makes output depend on render count; and a component that assumes props are mutable state.
The app should render but behave inconsistently or corrupt data.

Tell me only how to run it. Do not name the bugs. I diagnose each by reasoning about purity and
the render/commit phases. Make me explain why render must be pure, what "props are read-only"
means mechanically, and why a side effect during render is non-deterministic (especially under
re-renders / StrictMode double-invocation).
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A component mutating a prop object/array (`props.items.push(...)` or `props.user.name = ...`) — corrupting the parent's data and breaking one-way data flow.
- A side effect during render: mutating a module-level variable, incrementing a counter, or pushing to an array while rendering, so output depends on how many times React rendered (and StrictMode's intentional double-render in dev exposes it).
- Treating props as local mutable state (assigning to a prop and expecting it to persist).
- Possibly reading/writing the DOM during render instead of in an effect.

The principle: **a React component must be a pure function of its props and state — render computes UI, it does not mutate inputs or perform side effects.** Props are read-only; the parent owns that data. Impure render makes output depend on render count and order, which is exactly why React (in StrictMode) double-invokes render to surface these bugs.
</details>

---

## Module 5.2 — useState & The Rules Of State

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — State: A Component's Memory** — https://react.dev/learn/state-a-components-memory — *Focus on: why state needs `useState` (not a plain variable), and that setting state schedules a re-render.*
- **React docs — State as a Snapshot** — https://react.dev/learn/state-as-a-snapshot — *Focus on: state is fixed for a given render; the variable does not change mid-render even after you call the setter.*
- **React docs — Queueing a Series of State Updates** — https://react.dev/learn/queueing-a-series-of-state-updates — *Focus on: batching, and the updater function form `setX(x => x + 1)` vs the value form.*
- **React docs — Updating Objects/Arrays in State** — https://react.dev/learn/updating-objects-in-state — *Focus on: immutable updates; never mutate state directly.*

**Optional course resources:**
- Josh Comeau — *The Joy of React* — the "State" module.

**Search prompts:**
- `"react state snapshot stale closure setState"`
- `"react setState updater function vs value"`
- `"react state batching multiple setState"`
- Ask an AI: *"Explain 'state as a snapshot': why does logging state right after calling the setter show the OLD value? Why does calling `setCount(count + 1)` three times only add one, but `setCount(c => c + 1)` three times adds three? Explain batching and the updater function."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.2 — useState and the rules of state.
Completed so far: Tracks 1–4, Module 5.1.

Build me a React + Tailwind app with classic state bugs: a counter where clicking "+3" only
adds 1 (value-form setState called multiple times in one handler); a handler that reads state
right after setting it and acts on the STALE value; a component mutating an object/array in
state directly so React does not re-render (or re-renders unpredictably); and a derived value
stored in state that goes out of sync with its source.

Tell me only how to run it. Do not name the bugs. I diagnose each by reasoning about state as a
snapshot, batching, and immutability. Make me explain why state is fixed within a render, when
the updater function is required, and why mutating state silently breaks re-rendering.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A "+3" handler calling `setCount(count + 1)` three times: because `count` is a snapshot fixed for this render, all three compute from the same value and it increments once. Fix: `setCount(c => c + 1)` (updater form reads the latest queued value).
- Reading state immediately after `setState` and acting on the stale snapshot (the variable does not update synchronously).
- Mutating an object/array in state (`state.items.push(...)`, then `setItems(state.items)`) so the reference is unchanged and React may skip the re-render, or the update is unpredictable. Fix: create a new array/object.
- Derived state stored in `useState` that drifts from its source (foreshadows Module 5.6) — should be computed during render.

The principle: **within a single render, state is a constant snapshot; the setter schedules a re-render rather than mutating the variable in place, and updates batch.** The updater function form is required when you need the latest value across multiple updates, and state must be updated immutably so React's reference comparison detects the change.
</details>

---

## Module 5.3 — Reconciliation & Keys

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Preserving and Resetting State** — https://react.dev/learn/preserving-and-resetting-state — *Read fully. Focus on: "same component at the same position preserves state," and how keys and tree position control identity.*
- **React docs — Rendering Lists (keys)** — https://react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key — *Focus on: why keys must be stable and unique, and the index-as-key trap.*
- **React (legacy) — Reconciliation** — https://legacy.reactjs.org/docs/reconciliation.html — *Focus on: the diffing heuristics — different element types, same type, and keyed children.*

**Optional course resources:**
- Josh Comeau — "Why React Re-renders" article — https://www.joshwcomeau.com/react/why-react-re-renders/

**Search prompts:**
- `"react key index anti-pattern state bug"`
- `"react state resets unexpectedly position tree"`
- `"react reconciliation diffing same type"`
- Ask an AI: *"Explain React reconciliation: when does React reuse a component instance vs destroy and recreate it? What does 'same position in the tree' mean? Why does using array index as a key cause state to bleed between list items when the list reorders or items are inserted?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.3 — reconciliation and keys. Completed so
far: Tracks 1–4, Modules 5.1–5.2.

Build me a React + Tailwind app with three reconciliation bugs: (A) a counter that should
RESET when toggled between two variants but preserves its state (same component type, same
position); (B) a list of items each holding local state, keyed by index, where state bleeds to
the wrong items when the list is reordered or an item is prepended; (C) a conditional first
child that resets a sibling's state when toggled, because it shifts the sibling's tree
position.

Tell me only how to run it. Do not name the bugs. I diagnose each by reasoning about what React
compares to decide whether to keep or destroy an instance. Make me explain "same position
preserves state," why index keys break on reorder, and how to force a reset (or prevent one)
using keys and structure.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- **(A)** Two variants of the same component rendered at the same position (`cond ? <Counter a/> : <Counter b/>`): React keeps the instance and its state across the toggle. To force a reset, give them different `key`s (or render at different positions).
- **(B)** A list with `key={index}`: when items reorder or one is prepended, indices stay the same but map to different data, so React reuses instances against the wrong items and local state bleeds. Fix: a stable unique `key` (item id).
- **(C)** A conditionally-rendered first child shifts the sibling down a position, so React sees the sibling at a "different position" and resets it. Fix: keep stable positions (render the conditional as `null` placeholder, or restructure / key appropriately).

The principle: **React identifies a component by its type and position in the tree (refined by `key`); same identity preserves state, different identity destroys and recreates it.** Index keys lie about identity when order changes; conditional siblings change positions. Keys are how you tell React "this is the same thing" or "this is now different."
</details>

---

## Module 5.4 — useEffect & The Dependency Array

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Synchronizing with Effects** — https://react.dev/learn/synchronizing-with-effects — *Focus on: effects synchronize with external systems; the dependency array declares what the effect reads.*
- **React docs — Lifecycle of Reactive Effects** — https://react.dev/learn/lifecycle-of-reactive-effects — *Focus on: effects run/re-run/clean up based on their reactive dependencies, not on "mount/update" thinking.*
- **React docs — Removing Effect Dependencies** — https://react.dev/learn/removing-effect-dependencies — *Focus on: why you cannot just "lie" about dependencies, and how to legitimately remove them.*

**Optional course resources:**
- Josh Comeau — *The Joy of React* — the effects content.

**Search prompts:**
- `"react useEffect dependency array stale closure"`
- `"react useEffect infinite loop object dependency"`
- `"react exhaustive-deps eslint why"`
- Ask an AI: *"Explain the useEffect dependency array as a declaration of what the effect depends on. Why does omitting a dependency cause a stale closure, and why does including an object/function created during render cause an infinite loop? What is the mental model the React team wants me to use instead of 'componentDidMount'?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.4 — useEffect dependencies. Completed so
far: Tracks 1–4, Modules 5.1–5.3.

Build me a React + Tailwind app with dependency-array bugs: an effect with a MISSING dependency
that reads a stale value (a closure over old state/props); an effect that runs on EVERY render
and loops because an object/array/function dependency is recreated each render; an effect with
an empty dep array that should re-run when a value changes but never does; and an effect doing
work that should not be an effect at all (a derived computation).

Tell me only how to run it. Do not name the bugs. I diagnose each by reasoning about what the
effect actually reads and which of those are reactive. Make me explain why the dependency array
is not optional bookkeeping, why an unstable dependency loops, and what the exhaustive-deps
lint rule is protecting me from.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A missing dependency: the effect closes over a value it does not list, so it keeps using the value from the render when the effect last ran (stale closure). Fix: add the dependency (and adapt, e.g. updater functions, refs, or restructuring).
- An infinite loop: a dependency is an object/array/function created inline during render (new reference every render), so the effect re-runs, which often sets state, which re-renders, which recreates the dependency. Fix: memoize the dependency, move it inside the effect, or use primitive deps.
- An empty `[]` dep array where the effect should react to a changing value — it only runs once and goes stale.
- An effect computing derived data and storing it in state (should be computed during render — foreshadows 5.6).

The principle: **the dependency array tells React exactly which reactive values the effect uses, so it can re-synchronize when they change — it is a correctness contract, not a perf knob.** Missing deps cause stale closures; unstable (re-created) deps cause loops; and lots of effects are not effects at all but derived values.
</details>

---

## Module 5.5 — useEffect: Cleanup & Race Conditions

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Synchronizing with Effects: cleanup** — https://react.dev/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed — *Focus on: why effects need cleanup, and that cleanup runs before the next effect and on unmount.*
- **React docs — Fetching data in Effects (and its caveats)** — https://react.dev/learn/synchronizing-with-effects#fetching-data — *Focus on: the race condition when params change mid-fetch, and the `ignore`/AbortController fix.*
- **React docs — You Might Not Need an Effect** — https://react.dev/learn/you-might-not-need-an-effect — *Focus on: when fetching in an effect is acceptable vs better handled elsewhere.*

**Search prompts:**
- `"react useEffect fetch race condition cleanup ignore flag"`
- `"react effect cleanup function when runs"`
- `"react abortcontroller useEffect fetch"`
- Ask an AI: *"Explain the race condition when fetching in a useEffect whose dependency (like a search term or id) changes rapidly: why can a slower earlier request resolve AFTER a faster later one and show stale data? Show me both fixes — an `ignore` boolean in cleanup and AbortController — and explain when cleanup runs."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.5 — effect cleanup and race conditions.
Completed so far: Tracks 1–4, Modules 5.1–5.4. Raise the difficulty.

Build me a React + Tailwind app that fetches based on a changing input (a search box, or a
detail view selected by id) using useEffect, with realistic cleanup/race bugs: rapidly changing
the input causes a STALE response to overwrite a newer one (no cleanup/cancellation); a
subscription or timer set up in an effect is never cleaned up so it leaks/duplicates across
re-runs; and an async effect sets state after the component has unmounted.

Use a fake API with variable latency so the race is reproducible. Tell me only how to run it. Do
not name the bugs. I diagnose by observing wrong/flickering results and reasoning about effect
re-runs and cleanup timing. Make me explain when cleanup runs, why the race happens, and how the
`ignore` flag or AbortController prevents stale writes.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A fetch race: the effect re-runs when the input changes, but an earlier, slower request resolves after a newer one and overwrites the fresh result with stale data. Fix: a cleanup function that sets `ignore = true` (and the effect checks it before setting state), or an `AbortController` that aborts the previous request.
- A subscription/`setInterval`/event listener created in the effect with no cleanup, so each re-run adds another (leak / duplicate handlers).
- An async effect calling `setState` after unmount (cleanup should guard against it via the ignore flag / abort).
- The fix pattern: effects that start something must return a cleanup that stops it; the cleanup runs before each re-run and on unmount.

The principle: **an effect's cleanup function runs before the next run of that effect and on unmount, which is exactly the hook you need to cancel stale async work and tear down subscriptions.** Fetch races happen because effect runs overlap in time; an ignore flag or AbortController makes only the latest run's result win.
</details>

---

## Module 5.6 — Derived State & When NOT To Use Effects

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — You Might Not Need an Effect** — https://react.dev/learn/you-might-not-need-an-effect — *Read the whole page. Focus on: computing derived data during render, transforming data without effects, and resetting/adjusting state without effects.*
- **React docs — Choosing the State Structure** — https://react.dev/learn/choosing-the-state-structure — *Focus on: avoiding redundant and duplicated state; deriving instead of storing.*
- **React docs — Sharing State Between Components** — https://react.dev/learn/sharing-state-between-components — *Focus on: lifting state vs duplicating it.*

**Search prompts:**
- `"react derived state useEffect anti-pattern"`
- `"react compute during render instead of effect"`
- `"react redundant state out of sync"`
- Ask an AI: *"What does 'you might not need an effect' mean? Give me the canonical anti-patterns: storing derived data in state and syncing it with an effect, vs just computing it during render. Why is an effect-based sync slower AND buggier? When IS an effect genuinely the right tool?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.6 — derived state vs effects. Completed so
far: Tracks 1–4, Modules 5.1–5.5.

Build me a React + Tailwind app that overuses effects where derived computation belongs: a
filtered/sorted list stored in state and "kept in sync" with an effect (instead of computed
during render); a `fullName` state synced from `firstName`/`lastName` via effect; a "selected
item" that gets out of sync when the list changes because it is mirrored into separate state;
and an effect that resets state when a prop changes where a `key` would be cleaner. The app
works but flickers, double-renders, and occasionally shows stale derived data.

Tell me only how to run it. Do not name the issues. I diagnose by spotting state that is really
derived, and by noticing the extra render/flicker the effect-sync causes. Make me convert each
to derived-during-render (or a key-based reset) and explain why the effect version is both
slower and more bug-prone.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A filtered/sorted/derived list stored in state and synced via `useEffect` on the source — causes an extra render pass and can show stale data for one render. Fix: compute it during render (optionally `useMemo` if expensive).
- `fullName` mirrored from `firstName`/`lastName` via an effect — should just be `firstName + ' ' + lastName` during render.
- A "selected" or "active" item duplicated into state that drifts when the source list changes — should derive from the source (store the id, look up the item).
- An effect that resets child state when a prop changes — often better expressed by giving the component a `key` so React resets it (ties back to Module 5.3).
- Flicker/double-render is the symptom of effect-based syncing.

The principle: **if a value can be computed from existing props/state, compute it during render instead of storing it and syncing with an effect.** Effect-synced derived state runs an extra render, can be momentarily stale, and adds a synchronization bug surface. Effects are for synchronizing with EXTERNAL systems, not for keeping internal state in sync with other internal state.
</details>

---

## Module 5.7 — useRef & The DOM

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Referencing Values with Refs** — https://react.dev/learn/referencing-values-with-refs — *Focus on: a ref is mutable, persists across renders, and does NOT trigger a re-render when changed.*
- **React docs — Manipulating the DOM with Refs** — https://react.dev/learn/manipulating-the-dom-with-refs — *Focus on: attaching a ref to access a DOM node, and when that is appropriate (focus, measurement, imperative media APIs).*
- **React docs — useRef / forwardRef** — https://react.dev/reference/react/useRef — *Focus on: the `.current` property and forwarding refs to children.*

**Search prompts:**
- `"react useRef vs useState when to use"`
- `"react ref does not trigger rerender"`
- `"react access dom node ref focus measure"`
- Ask an AI: *"When should I use `useRef` instead of `useState`? Explain that a ref is a mutable box that persists across renders but does NOT cause re-renders. Give me legitimate use cases (DOM access, storing a previous value, holding a timer id) and the anti-pattern of using a ref where state is needed."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.7 — useRef and the DOM. Completed so far:
Tracks 1–4, Modules 5.1–5.6.

Build me a React + Tailwind app with ref bugs: a value stored in a ref that the UI is supposed
to display, but the UI never updates because changing a ref does not re-render (should be
state); a DOM ref read during render (before commit, so it is null); a ref used to store
something that SHOULD be state; and an effect that needs the DOM node but accesses it at the
wrong time. Include one legitimate ref use (focusing an input, or measuring an element) done
correctly as a contrast.

Tell me only how to run it. Do not name the bugs. I diagnose each by reasoning about when refs
update vs when state does, and when the DOM node actually exists. Make me explain the difference
between a ref and state, and why reading a DOM ref during render is too early.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A displayed value stored in a ref: updating `ref.current` does not trigger a re-render, so the UI is stale. Fix: it needs to be state.
- Reading `ref.current` (a DOM node) during render — it is `null` until after commit. DOM access belongs in an effect or event handler.
- A ref used where state is required for reactivity (or state used where a ref would avoid needless re-renders — e.g. a timer id).
- An effect that measures/focuses a node but runs before the node exists or before layout (sometimes needs `useLayoutEffect` for measurement to avoid a flash).
- The correct contrast: `inputRef.current.focus()` in an effect/handler, or measuring with `getBoundingClientRect` in `useLayoutEffect`.

The principle: **a ref is a mutable value that persists across renders without causing them — the escape hatch for imperative needs (DOM nodes, timers, previous values) — while state is for values the UI must react to.** DOM refs are only populated after commit, so reading them during render is too early.
</details>

---

## Module 5.8 — useMemo, useCallback & React.memo

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — useMemo** — https://react.dev/reference/react/useMemo — *Focus on: caching an expensive computation, and the "Should you add useMemo everywhere?" section.*
- **React docs — useCallback** — https://react.dev/reference/react/useCallback — *Focus on: memoizing a function identity so memoized children do not re-render.*
- **React docs — memo** — https://react.dev/reference/react/memo — *Focus on: skipping re-renders when props are referentially equal, and why a new function/object prop defeats it.*
- **Dan Abramov — Before You memo()** — https://overreacted.io/before-you-memo/ — *Focus on: structural fixes (composition, lifting content) before memoization.*

**Search prompts:**
- `"react.memo not working new function prop"`
- `"usecallback usememo when actually needed"`
- `"react before you memo composition"`
- Ask an AI: *"Explain when `useMemo`, `useCallback`, and `React.memo` actually help, and when they are wasted. Why does passing a new inline function or object to a `React.memo` child defeat the memoization? What are the structural alternatives (composition, children-as-props) Dan Abramov recommends trying first?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.8 — memoization. Completed so far:
Tracks 1–4, Modules 5.1–5.7. Raise the difficulty.

Build me a React + Tailwind app where memoization is BROKEN and MISUSED: a `React.memo` child
that still re-renders every time because it receives a new inline function/object prop each
render; an expensive computation re-run every render that should be `useMemo`'d; `useMemo`/
`useCallback` sprinkled everywhere with empty or wrong dep arrays (some causing stale values,
some pure overhead); and a case where the RIGHT fix is composition (passing children) rather
than memoization. Add console logs / the Profiler so re-renders are visible.

Tell me only how to run it. Do not name the issues. I diagnose with the Profiler and reasoning
about referential equality. Make me fix the broken memo (stabilize the props), add memoization
only where it is structurally necessary, remove the pointless ones, and solve at least one case
with composition instead. Explain referential equality and why memo depends on it.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A `React.memo` child that re-renders anyway because the parent passes a NEW inline function (`onClick={() => ...}`) or object (`style={{...}}`) each render — new reference defeats the shallow prop comparison. Fix: `useCallback`/`useMemo` to stabilize, or restructure.
- An expensive calculation run on every render that should be wrapped in `useMemo` with correct deps.
- `useMemo`/`useCallback` with wrong/empty deps producing stale values, or applied to trivial computations where the memo overhead exceeds the benefit (pure noise).
- A case better solved by composition: lifting expensive/static children up and passing them as `props.children` so they do not re-render with the parent — no memo needed.
- The Profiler confirms which components re-render before/after.

The principle: **`React.memo` skips a re-render only when props are referentially equal, so unstable function/object props silently defeat it; `useMemo`/`useCallback` exist to provide that stability or to cache real work — not to decorate every value.** Often the cleaner fix is structural (composition), per "Before You memo()". Memoization is a precision tool, not a default.
</details>

---

## Module 5.9 — Context & The Re-render Problem

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Passing Data Deeply with Context** — https://react.dev/learn/passing-data-deeply-with-context — *Focus on: when context is appropriate and the provider/consumer model.*
- **React docs — Scaling Up with Reducer and Context** — https://react.dev/learn/scaling-up-with-reducer-and-context — *Focus on: combining reducer + context, and separating state and dispatch.*
- **React docs — useContext** — https://react.dev/reference/react/useContext — *Focus on: that every consumer re-renders when the provider's value changes (by reference).*

**Search prompts:**
- `"react context re-render all consumers performance"`
- `"react split context value memo provider"`
- `"react context vs state management"`
- Ask an AI: *"Explain why every consumer of a React Context re-renders when the Provider's `value` changes — and why passing a new object as `value` each render makes ALL consumers re-render every time. Show me the fixes: memoizing the value, splitting contexts (state vs dispatch, or by concern), and when to reach for a real state library instead."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.9 — context and re-renders. Completed so
far: Tracks 1–4, Modules 5.1–5.8.

Build me a React + Tailwind app where a single Context holds several unrelated values in one
object passed as `value`, so EVERY consumer re-renders whenever ANYTHING changes — plus the
`value` is a new object each render, so consumers re-render even when nothing meaningful
changed. Include a deep tree with several consumers (a header reading user, a filter bar
reading filter, a list reading items) and console logs / Profiler to expose the re-renders.

Tell me only how to run it. Do not name the issues. I diagnose with the Profiler. Make me fix it
by stabilizing/memoizing the value and SPLITTING the context (so a component only re-renders for
the slice it consumes), reaching for memoization only where structurally necessary. Explain why
context propagates by reference identity and why one big value object is the trap.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- One context whose `value={{ user, items, filter, setFilter, ... }}` is a NEW object every render, so all consumers re-render every render regardless of what changed.
- All concerns bundled in one context, so a component that only needs `filter` still re-renders when `items` changes.
- Fixes: memoize the value object; SPLIT into multiple contexts by concern (UserContext, ItemsContext, FilterContext) or separate state from dispatch — so each consumer subscribes only to what it uses; memoize consumers where needed.
- Possibly an inline handler in the value recreated each render.
- Profiler shows the cascade before, and isolated re-renders after.

The principle: **context delivers updates to every consumer whenever the provider's `value` changes by reference, so a fresh value object or one over-broad context re-renders the whole subtree.** Split contexts by concern and stabilize the value so each component re-renders only for the data it actually reads. Context is a transport, not a performance-optimized store.
</details>

---

## Module 5.10 — useReducer & Complex State

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Extracting State Logic into a Reducer** — https://react.dev/learn/extracting-state-logic-into-a-reducer — *Focus on: when multiple related state updates and complex transitions justify a reducer.*
- **React docs — useReducer** — https://react.dev/reference/react/useReducer — *Focus on: the `(state, action) => newState` contract and dispatching actions.*
- **React docs — Choosing the State Structure (revisit)** — https://react.dev/learn/choosing-the-state-structure — *Focus on: modeling state to avoid impossible combinations.*

**Search prompts:**
- `"react usereducer vs usestate when"`
- `"react reducer state machine impossible states"`
- Ask an AI: *"When is `useReducer` better than several `useState` calls? Show me how a reducer centralizes related transitions and prevents 'impossible states' (e.g. both loading and error true). Walk me through modeling a multi-step form or an async status (idle/loading/success/error) as a reducer."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.10 — useReducer and complex state, build
challenge. Completed so far: Tracks 1–4, Modules 5.1–5.9.

Give me a spec for a component with genuinely complex, interdependent state — e.g. a multi-step
wizard, or an async data panel with idle/loading/success/error plus retry, or a shopping cart
with add/remove/quantity/discount — where multiple `useState`s would create impossible
combinations and scattered update logic. Give me requirements, no code.

My job: model it with `useReducer`, designing actions and a state shape that makes impossible
states unrepresentable. Review me strictly. If my reducer mutates state, or allows contradictory
states (loading AND error), or my actions are really just setters, do not fix it — ask me what
transitions are legal and whether my state shape can even express an illegal one. Make me
explain why a reducer centralizes transition logic and how the state shape prevents bugs.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A pure reducer `(state, action) => newState` that returns NEW state (no mutation) and handles each action as a legal transition.
- A state shape that makes impossible states unrepresentable: e.g. a single `status: 'idle' | 'loading' | 'success' | 'error'` rather than separate booleans that can contradict; data/error tied to the status that can hold them.
- Meaningful actions describing intent (`SUBMIT`, `NEXT_STEP`, `FETCH_SUCCESS`) rather than generic `SET_X` setters.
- Common mistakes: mutating state in the reducer; modeling status as independent booleans (allowing loading && error); actions that are thin setters (no centralization benefit); putting derived data in state.
- The "make impossible states impossible" framing (discriminated-union state) ties back to TypeScript narrowing (Track 8).

The principle: **a reducer centralizes all transitions for a piece of state in one pure function, and a well-chosen state shape makes illegal combinations impossible to represent.** This replaces a scatter of `useState`s and ad-hoc updates with a single, testable model of how state is allowed to change.
</details>

---

## Module 5.11 — Custom Hooks

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Reusing Logic with Custom Hooks** — https://react.dev/learn/reusing-logic-with-custom-hooks — *Focus on: that a custom hook shares stateful LOGIC, not state; each call gets its own state.*
- **React docs — Rules of Hooks** — https://react.dev/reference/rules/rules-of-hooks — *Focus on: call hooks at the top level, only from React functions, in the same order every render.*
- **React docs — useSyncExternalStore** — https://react.dev/reference/react/useSyncExternalStore — *Focus on: subscribing to external stores correctly (a common custom-hook need).*

**Search prompts:**
- `"react custom hook share logic not state"`
- `"react rules of hooks why same order"`
- `"react custom hook useEventListener useLocalStorage"`
- Ask an AI: *"Explain custom hooks: why does extracting logic into a `useX` hook share the LOGIC but give each component its own independent state? Why must hooks be called in the same order every render (the Rules of Hooks)? Walk me through building a robust `useLocalStorage` or `useFetch` hook."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.11 — custom hooks, build challenge.
Completed so far: Tracks 1–4, Modules 5.1–5.10.

Give me a spec to extract and build a few robust custom hooks: e.g. `useLocalStorage(key,
initial)` (synced state persisted to storage), `useDebouncedValue(value, delay)`, and
`useFetch(url)` (with loading/error/cancel from the lessons in Module 5.5). Then use them in a
small component. Give me no solution code.

Review me strictly. If a hook leaks effects, returns an unstable function each render, violates
the Rules of Hooks (conditional hook calls), or shares state where it should be per-instance, do
not fix it — ask me what each consumer should get its own copy of, and why hook order must be
stable. Make me explain that custom hooks reuse logic (not state) and why the Rules of Hooks
exist mechanically.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Hooks that compose built-in hooks (`useState`/`useEffect`/`useRef`) and return a clean API; each consuming component gets its OWN independent state (logic is shared, state is not).
- `useLocalStorage`: reads initial value lazily, writes on change, handles JSON/parse errors, and ideally syncs across tabs via the `storage` event.
- `useDebouncedValue`: an effect with proper cleanup (clears the timer) and correct deps.
- `useFetch`: loading/error states, cancellation/ignore-on-cleanup (race safety from 5.5), stable return where needed.
- Rules of Hooks respected: no conditional/looped hook calls; hooks only in components/hooks.
- Common mistakes: conditional hook calls (breaks the stable call order React relies on to associate state); effects without cleanup; returning new function identities that destabilize consumers; sharing a single state across instances (e.g. module-level state) when per-instance was intended.

The principle: **a custom hook is a function that calls other hooks to package reusable stateful LOGIC — every call site gets its own isolated state — and the Rules of Hooks (same hooks, same order, top level) are what let React match each hook call to its stored state across renders.** Break the order and React mis-associates state.
</details>

---

## Module 5.12 — Compound Components & Render Props

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Passing JSX as children / Sharing state** — https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children — *Focus on: composition via `children` and slot-like patterns.*
- **React docs — Context (for compound components)** — https://react.dev/learn/passing-data-deeply-with-context — *Focus on: using context internally so sub-components share parent state implicitly.*
- **Kent C. Dodds — Compound Components with hooks** — https://kentcdodds.com/blog/compound-components-with-react-hooks — *Focus on: the `Tabs`/`Tabs.Tab`/`Tabs.Panel` pattern and a context hook that errors outside the parent.*
- **patterns.dev — Render Props** — https://www.patterns.dev/react/render-props-pattern/ — *Focus on: render props vs hooks, and when each still applies.*

**Search prompts:**
- `"react compound components pattern context"`
- `"react render props vs custom hooks"`
- `"react state reducer pattern inversion of control"`
- Ask an AI: *"Explain the compound-components pattern: how do `Tabs.Tab` and `Tabs.Panel` share the active-tab state without prop drilling (internal context)? Then explain render props and when they beat a custom hook. What is 'inversion of control' and why does it make a component API more flexible?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.12 — compound components and render props,
build challenge. Completed so far: Tracks 1–4, Modules 5.1–5.11. Raise the difficulty.

Give me a spec to build flexible component APIs WITHOUT solution code: (1) a `Tabs` compound
component used as `<Tabs><Tabs.List><Tabs.Tab/></Tabs.List><Tabs.Panel/></Tabs>` where the
active state lives in `Tabs` and is shared via internal context, with correct ARIA; (2) a
render-prop or children-as-function `DataFetcher` exposing `{data, loading, error, refetch}`;
and (3) a `Toggle` using the state-reducer pattern so a consumer can override its behavior.

Review me strictly. For compound components, if I prop-drill the shared state, ask me where it
should live and how sub-components could read it without explicit props. For the state reducer,
ask me what the consumer can now control that a plain callback could not. Make me explain each
pattern's purpose and what problem it solves that simpler props cannot.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- **Compound components:** `Tabs` holds active state; sub-components (`Tabs.Tab`, `Tabs.Panel`) read it via an internal context (with a custom hook that throws a helpful error if used outside `Tabs`); correct ARIA (`role="tab"`, `aria-selected`, `aria-controls`, `role="tabpanel"`). Sub-components are static properties on `Tabs`. No prop drilling.
- **Render props:** `DataFetcher` calls `props.children({ data, loading, error, refetch })`; the consumer controls rendering. Understand why this is sometimes preferable to a hook (when the consumer needs to control where/how the result renders, or for component-level composition).
- **State reducer:** `Toggle` accepts a `stateReducer` prop called before its internal reducer, letting the consumer veto/override transitions (inversion of control).
- Common mistakes: prop-drilling compound state instead of context; a context hook with no out-of-bounds guard; render props that could just be a hook (recognize the tradeoff); a state reducer that does not actually let the consumer intercept.

The principle: **these patterns invert control to the consumer — compound components share state implicitly via context for a clean declarative API, render props let the consumer own rendering, and the state-reducer pattern lets the consumer override behavior.** They solve flexibility and composition problems that plain props and callbacks cannot express cleanly.
</details>

---

## Module 5.13 — Suspense & Error Boundaries

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Suspense** — https://react.dev/reference/react/Suspense — *Focus on: showing a fallback while children load, and what "suspending" means.*
- **React docs — Error Boundaries (Component catching errors)** — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary — *Focus on: that error boundaries catch render errors in their subtree, and what they do NOT catch (event handlers, async).*
- **React docs — `<Suspense>` for data fetching** — https://react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading — *Focus on: how data libraries integrate with Suspense.*

**Search prompts:**
- `"react error boundary catch render errors not event handlers"`
- `"react suspense fallback boundary placement"`
- Ask an AI: *"Explain Suspense and Error Boundaries. What does it mean for a component to 'suspend,' and how does the nearest `<Suspense>` show a fallback? What errors do Error Boundaries catch (render/lifecycle) and which do they NOT catch (event handlers, async, SSR)? How do I handle the cases boundaries miss?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.13 — Suspense and error boundaries.
Completed so far: Tracks 1–4, Modules 5.1–5.12.

Build me a React + Tailwind app with bugs around boundaries: an Error Boundary placed so that
when a child throws, the WHOLE app blanks instead of just a section (boundary too high); an
error thrown in an EVENT HANDLER that the Error Boundary does not catch (so the dev wrongly
expects it to); a Suspense fallback positioned so the entire page flashes a spinner instead of
just the loading region (boundary too coarse); and a component that suspends with no Suspense
ancestor at all. Use a data source that can suspend and components that can throw.

Tell me only how to run it. Do not name the issues. I diagnose by reasoning about WHERE
boundaries sit and WHAT they catch. Make me reposition boundaries for graceful, localized
fallbacks/errors, handle the event-handler error correctly (try/catch + state), and explain
precisely what Suspense and Error Boundaries do and do not cover.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- An Error Boundary at the app root only, so any child error blanks everything. Fix: place boundaries around independent sections for localized fallback UI.
- An error thrown inside an event handler (onClick) that the Error Boundary does NOT catch (boundaries only catch errors during render/lifecycle). Fix: handle it with try/catch in the handler and set error state (or surface a toast).
- A `<Suspense>` wrapping the whole page so one slow region flashes a full-page spinner. Fix: granular Suspense boundaries around just the loading region for targeted fallbacks.
- A component that suspends (throws a promise / uses a suspending data source) with no Suspense ancestor — crashes or bubbles. Fix: add an appropriate `<Suspense>`.

The principle: **Error Boundaries catch errors thrown during rendering/lifecycle in their subtree (not in event handlers, async code, or after the fact), and Suspense shows the nearest fallback while a descendant is suspended — so WHERE you place both determines how localized your loading and error UX is.** Coarse boundaries blank or spin the whole app; granular ones degrade gracefully.
</details>

---

## Module 5.14 — Next.js: App Router & Server Components

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Next.js (App Router) + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Next.js — App Router: Server and Client Components** — https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns — *Focus on: the server/client boundary, `"use client"`, and what can cross it (serializable props only).*
- **Next.js — Server Components** — https://nextjs.org/docs/app/building-your-application/rendering/server-components — *Focus on: that server components run only on the server, can be async, and cannot use hooks/state/browser APIs.*
- **Next.js — Client Components** — https://nextjs.org/docs/app/building-your-application/rendering/client-components — *Focus on: when you need `"use client"` and what it costs.*
- **React docs — Server Components (conceptual)** — https://react.dev/reference/rsc/server-components — *Focus on: the mental model.*

**Search prompts:**
- `"next.js use client server component boundary"`
- `"next.js useState hook server component error"`
- `"react server component cannot pass function as prop"`
- Ask an AI: *"Explain the React Server Components model in Next.js App Router: what runs on the server vs the client, why server components can be async but cannot use `useState`/`useEffect`/browser APIs, and what can cross the boundary (only serializable props — no functions). When do I add `'use client'`, and what does it cost?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.14 — Next.js App Router and server
components. Completed so far: Tracks 1–4, Modules 5.1–5.13. I use Next.js at work, so probe my
model of the server/client boundary.

Scaffold me a Next.js (App Router) + Tailwind app with realistic boundary bugs: a server
component using `useState`/`useEffect`/`onClick` (errors or doesn't work); a `'use client'`
placed too high so a huge subtree needlessly ships to the client; a function/non-serializable
value passed as a prop from a server component to a client component (boundary violation);
and a component using a browser API (`window`/`localStorage`) on the server. The app should
fail to build or misbehave in instructive ways.

Tell me only how to run it. Do not name the bugs. I diagnose by reasoning about what runs where.
Make me fix each (move `'use client'` to the leaf that needs interactivity, keep server
components server-only, pass only serializable props), and explain the server/client boundary
and why only serializable data crosses it.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A server component using `useState`/`useEffect`/event handlers/`onClick` — not allowed; these require a client component (`'use client'`).
- `'use client'` declared at the top of a large subtree, forcing all of it (and its dependencies) to ship to and run on the client. Fix: push `'use client'` down to the smallest interactive leaf; keep parents as server components.
- Passing a function (or class instance, Date in some cases, etc.) as a prop from a server component to a client component — only SERIALIZABLE props cross the boundary. Fix: pass serializable data, or move the logic.
- Using `window`/`document`/`localStorage` in code that runs on the server. Fix: guard for client, move into a client component / effect.

The principle: **in the App Router, components are server components by default — they run only on the server, can be async, and cannot use state/effects/browser APIs — and `'use client'` marks the boundary where interactivity (and a client bundle cost) begins.** Only serializable data can cross from server to client (no functions). Most boundary bugs are interactive code on the server or an over-broad `'use client'`.
</details>

---

## Module 5.15 — Next.js: Data Fetching & Caching

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Next.js (App Router) + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Next.js — Data Fetching: fetching, caching, revalidating** — https://nextjs.org/docs/app/building-your-application/data-fetching/fetching — *Focus on: fetching in server components, the `fetch` cache, `revalidate`, and `cache: 'no-store'`.*
- **Next.js — Caching in Next.js** — https://nextjs.org/docs/app/building-your-application/caching — *Focus on: the Request Memoization, Data Cache, Full Route Cache, and Router Cache layers.*
- **Next.js — Revalidating data** — https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration — *Focus on: time-based and on-demand revalidation.*

**Search prompts:**
- `"next.js fetch cache revalidate no-store"`
- `"next.js data cache full route cache explained"`
- `"next.js stale data not updating cache"`
- Ask an AI: *"Explain Next.js App Router caching layers: Request Memoization, the Data Cache, the Full Route Cache, and the client Router Cache. When does a `fetch` get cached, and how do `revalidate`, `cache: 'no-store'`, and `revalidateTag`/`revalidatePath` control freshness? Why might my data look stale?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.15 — Next.js data fetching and caching.
Completed so far: Tracks 1–4, Modules 5.1–5.14.

Scaffold me a Next.js (App Router) + Tailwind app with caching bugs: a page showing STALE data
because a `fetch` is cached indefinitely when it should revalidate (or be dynamic); a page that
is unexpectedly fully static when it needed per-request data; duplicate identical fetches in
one render that should be deduped; and a mutation that updates data on the server but the UI
keeps showing the old cached version because nothing revalidates. Use a mock API with a
changing value so staleness is visible.

Tell me only how to run it. Do not name the issues. I diagnose by reasoning about which cache
layer is serving each value. Make me fix freshness with the right tool (`revalidate`, `cache:
'no-store'`, `revalidateTag`/`revalidatePath`) and explain the caching layers and which one was
responsible for each symptom.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A `fetch` that is cached (default Data Cache behavior in many setups) shown as stale when it needed freshness. Fix: `{ next: { revalidate: N } }` for time-based, or `{ cache: 'no-store' }` for always-fresh/dynamic.
- A route rendered statically (Full Route Cache) when it needed per-request/dynamic rendering. Fix: opt into dynamic rendering (dynamic API usage, `no-store`, or route segment config).
- Identical fetches in one render tree that should be deduped by Request Memoization (often already automatic for `fetch`) — or a non-`fetch` data source that needs `React.cache`.
- A mutation that changes server data but the cached page/route does not refresh. Fix: `revalidateTag`/`revalidatePath` after the mutation so the Data/Route cache updates.
- The client Router Cache showing stale data on back-navigation.

The principle: **Next.js layers caching (request memoization, the persistent Data Cache, the Full Route Cache, and the client Router Cache), and "stale data" almost always means one layer is serving a cached value you expected to be fresh.** You control freshness explicitly with `revalidate`, `no-store`, and `revalidateTag`/`revalidatePath` — and diagnosing means identifying which layer is responsible.
</details>

---

## Module 5.16 — Next.js: Server Actions & Mutations

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Next.js (App Router) + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Next.js — Server Actions and Mutations** — https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations — *Focus on: `'use server'`, calling actions from forms and client components, and revalidating after a mutation.*
- **Next.js — Forms and mutations / useFormStatus, useActionState** — https://nextjs.org/docs/app/api-reference/functions/use-form-status — *Focus on: pending UI and returning state from actions.*
- **React docs — `useActionState` / `useOptimistic`** — https://react.dev/reference/react/useOptimistic — *Focus on: optimistic UI tied to actions.*

**Search prompts:**
- `"next.js server action use server form mutation"`
- `"next.js revalidatePath after server action"`
- `"react useOptimistic useActionState server action"`
- Ask an AI: *"Explain Next.js Server Actions: how `'use server'` lets a form or client component call a server function directly, how I get pending state (`useFormStatus`/`useActionState`), and why I must `revalidatePath`/`revalidateTag` after a mutation. How do I add optimistic UI with `useOptimistic`, and how do I handle validation/errors safely?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.16 — Next.js server actions and mutations,
build challenge. Completed so far: Tracks 1–4, Modules 5.1–5.15.

Give me a spec to build a small feature (e.g. a todo list or a comment form) backed by Server
Actions: create/update/delete via `'use server'` actions invoked from a form and from a client
component, with pending UI, server-side validation, post-mutation revalidation so the list
stays correct, and optimistic updates with rollback on failure. Use a mock data store. Give me
no solution code.

Review me strictly. If after a mutation the UI shows stale data, do not tell me — ask me what
needs to revalidate and why. If my optimistic update never rolls back on error, ask me what
happens when the action rejects. If I trust client input without validating on the server, push
back hard. Make me explain the action round-trip, revalidation, and optimistic/rollback.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `'use server'` actions for each mutation, callable from a `<form action={...}>` and/or a client component; pending UI via `useFormStatus`/`useActionState`.
- Server-side validation inside the action (never trust client input), returning structured errors that the UI surfaces.
- `revalidatePath`/`revalidateTag` after a successful mutation so the cached data/route refreshes and the UI reflects the change.
- Optimistic UI with `useOptimistic` that applies the change immediately and ROLLS BACK if the action throws/returns an error.
- Common mistakes: no revalidation (stale UI after mutation); optimistic update with no rollback path; validating only on the client; an action that mutates but returns nothing useful for error handling; forgetting that actions run on the server (no browser APIs).

The principle: **Server Actions let client UI invoke server mutations directly, but you must validate on the server, revalidate the affected cache after the mutation, and (for snappy UX) apply optimistic updates with a rollback path.** The mutation round-trip, cache revalidation, and optimistic/rollback are the three things a correct action flow always gets right.
</details>

---

## Module 5.17 — Next.js: Rendering Strategies (SSR / SSG / ISR / Streaming)

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Next.js (App Router) + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Next.js — Rendering: Static and Dynamic** — https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default — *Focus on: static vs dynamic rendering and what triggers each.*
- **Next.js — Incremental Static Regeneration** — https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration — *Focus on: `revalidate` for ISR and on-demand revalidation.*
- **Next.js — Streaming and Suspense** — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming — *Focus on: `loading.js`, streaming, and partial rendering.*

**Search prompts:**
- `"next.js static vs dynamic rendering when"`
- `"next.js ISR revalidate streaming loading.js"`
- Ask an AI: *"Compare Next.js rendering strategies: static rendering (SSG), dynamic rendering (SSR per request), ISR (static + revalidation), and streaming with Suspense. What makes a route static vs dynamic in the App Router, and how do I choose the right strategy for a marketing page, a personalized dashboard, and a frequently-updated article?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.17 — Next.js rendering strategies,
investigation. Completed so far: Tracks 1–4, Modules 5.1–5.16.

Scaffold me a Next.js (App Router) + Tailwind app with several routes whose rendering strategy
is WRONG for their needs: a personalized/per-user page accidentally rendered statically (so all
users see one cached version); a rarely-changing marketing page forced dynamic (rendered per
request, slow and wasteful); a frequently-updated content page that is fully static and never
refreshes (should be ISR); and a slow page that blocks the whole route instead of streaming the
fast parts first.

Tell me how to run it and how to observe which routes are static vs dynamic (build output,
response headers, behavior). Do not tell me the issues. I diagnose each route's actual strategy
vs what it needs, then fix it (force dynamic, allow static, add `revalidate` for ISR, add
streaming with Suspense/`loading.js`). Make me explain each strategy and its tradeoffs.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A personalized page (depends on the user/cookies/per-request data) that is statically rendered, so everyone sees one cached version. Fix: make it dynamic (use a dynamic data source / `no-store` / dynamic API), so it renders per request.
- A static-friendly marketing page forced dynamic (e.g. by an unnecessary dynamic API or `no-store`), rendered on every request. Fix: let it be static.
- A frequently-updated page fully static with no revalidation, so it never updates. Fix: ISR via `revalidate: N` (or on-demand revalidation).
- A route where one slow data dependency blocks the entire page. Fix: stream — wrap the slow part in `<Suspense>` (or use `loading.js`) so the fast shell renders immediately.

The principle: **each route should match its data's nature: static for unchanging content, dynamic for per-request/personalized content, ISR for content that changes on a cadence, and streaming to send the fast parts before the slow parts.** Choosing wrong makes pages stale, slow, or wasteful; the App Router infers static/dynamic from how you fetch and which APIs you use, so you steer it deliberately.
</details>

---

## Module 5.18 — React Performance Profiling

**Difficulty:** ●●●●○ · **Format:** Investigation · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — React Developer Tools / Profiler** — https://react.dev/learn/react-developer-tools — *Focus on: installing and using the Profiler to record commits and find slow/extra renders.*
- **React docs — Profiler component** — https://react.dev/reference/react/Profiler — *Focus on: `actualDuration` vs `baseDuration` and reading the commit data.*
- **Josh Comeau — Why React Re-renders (revisit)** — https://www.joshwcomeau.com/react/why-react-re-renders/ — *Focus on: the causes of re-renders you will hunt.*

**Search prompts:**
- `"react devtools profiler flame chart wasted renders"`
- `"react highlight updates re-renders find"`
- Ask an AI: *"Teach me to use the React DevTools Profiler: how to record a commit, read the flame chart, find components that rendered when they didn't need to, and use 'Highlight updates' to see re-renders live. How do I distinguish a necessary re-render from a wasted one?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.18 — React performance profiling,
investigation. Completed so far: Tracks 1–4, Modules 5.1–5.17.

Build me a React + Tailwind app that FEELS sluggish on interaction: typing in an input lags
because the whole tree re-renders per keystroke; an expensive child re-renders on unrelated
state changes; a large list re-renders entirely when one item changes; and a context or prop
pattern causes a re-render cascade. Make the slowness real (enough work that it is perceptible,
or use CPU throttle).

Tell me how to run it and how to record with the React DevTools Profiler and use "Highlight
updates." Do not tell me the causes. I will profile, identify which components re-render and
why (referential equality, context, missing memo, list keys), and fix the worst offenders using
the right tool (lift/colocate state, split context, memoize where structural, stabilize props),
re-profiling to confirm. Make me explain how I distinguished necessary from wasted renders.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- State placed too high (e.g. the input's value in a top-level component), so every keystroke re-renders the whole tree. Fix: colocate state in the smallest component that needs it.
- An expensive child re-rendering on unrelated parent state changes — fix via composition (pass it as children), `React.memo` with stable props, or moving state down.
- A large list re-rendering entirely on one item's change — fix with proper keys, memoized rows, and stable handlers; consider virtualization (Track 6) for very large lists.
- A context value object recreated each render causing a cascade (ties to Module 5.9).
- The Profiler distinguishes "rendered and took time" from "rendered unnecessarily"; "Highlight updates" visualizes the cascade.

The principle: **most React perf problems are unnecessary re-renders, and the Profiler tells you exactly which components rendered, how long they took, and (by reasoning) why — so you fix the cause (state placement, referential stability, context shape, keys) rather than sprinkling memo blindly.** Measure, find the wasted work, fix the structural cause, re-measure.
</details>

---

## Module 5.19 — Concurrent React: useTransition & useDeferredValue

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — useTransition** — https://react.dev/reference/react/useTransition — *Focus on: marking a state update as a non-urgent transition so the UI stays responsive, and the `isPending` flag.*
- **React docs — useDeferredValue** — https://react.dev/reference/react/useDeferredValue — *Focus on: deferring a value so expensive renders driven by it do not block urgent updates.*
- **React docs — Keeping the UI responsive with transitions** — https://react.dev/reference/react/startTransition — *Focus on: urgent vs non-urgent updates and how concurrent rendering interrupts.*

**Search prompts:**
- `"react usetransition ispending non-urgent update"`
- `"react usedeferredvalue expensive list filter"`
- `"react concurrent rendering interruptible"`
- Ask an AI: *"Explain `useTransition` and `useDeferredValue`. What is an 'urgent' vs 'non-urgent' update, and how does marking a slow update as a transition keep the input responsive? How does concurrent React interrupt a non-urgent render when an urgent one arrives? When do I use each hook?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 5.19 — concurrent React, build challenge.
Completed so far: Tracks 1–4, Modules 5.1–5.18. This is the React track capstone — make it
genuinely hard.

Give me a spec for a feature where an urgent update (typing in a search/filter input) competes
with an expensive update (filtering/rendering a very large list, or a heavy visualization).
Naively, typing lags because each keystroke triggers the expensive render synchronously. My job:
keep the input perfectly responsive using `useTransition` and/or `useDeferredValue`, showing a
pending state while the expensive view catches up. Give me no solution code.

Review me strictly. If typing still lags, ask me which update is urgent and which is not, and
whether I marked the right one as a transition. If I use both hooks redundantly, ask me what
each one actually does differently. Make me explain how concurrent React lets an urgent update
interrupt a non-urgent one, and the precise difference between transitions and deferred values.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The input's value update stays urgent (synchronous, responsive); the expensive derived render is made non-urgent — either by wrapping the state update that drives it in `startTransition` (`useTransition`, with `isPending` for a "updating…" indicator), or by feeding the expensive component a `useDeferredValue` of the input.
- Understanding the difference: `useTransition` wraps the STATE UPDATE you control (you mark the transition at the source); `useDeferredValue` wraps a VALUE you receive (you defer downstream, useful when you don't control the update). They are not both needed for the same path.
- A pending UI (dimming the stale list, a spinner) driven by `isPending` or by comparing the deferred vs current value.
- Common mistakes: marking the input value itself as a transition (makes typing feel laggy/dropped); using both hooks redundantly; expecting concurrency to make the work FASTER (it makes the UI responsive by prioritizing/interrupting, not by speeding computation); not providing pending feedback so it feels broken.

The principle: **concurrent React can interrupt a low-priority (non-urgent) render to handle a high-priority one, so marking the expensive update as a transition — or deferring the value that drives it — keeps urgent input responsive while the heavy view catches up.** Transitions act at the update source; deferred values act at the consumption point. Neither speeds the work; both protect responsiveness.
</details>

---

*End of Track 5 — React & Next.js. 19 modules.*

*Next: Track 6 — State, Data & Async Architecture, in its own file.*
