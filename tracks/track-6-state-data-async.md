# TRACK 6 — STATE, DATA & ASYNC ARCHITECTURE

*The patterns that separate apps that scale from apps that rot. By now you understand React's render model; this track is about the harder question of where data lives, how it stays consistent, and how async flows are structured. You build a state library from scratch, tame server state, go offline-first, and end at real-time collaboration. Mostly React + Tailwind, with vanilla where building from scratch teaches more.*

*Reminder: your root `CLAUDE.md` is loaded automatically. Prep first. No spoilers early.*

---

## Module 6.1 — Lifting State & Prop Drilling

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — Sharing State Between Components** — https://react.dev/learn/sharing-state-between-components — *Focus on: lifting state to the closest common ancestor, and the controlled/uncontrolled distinction.*
- **React docs — Passing Data Deeply with Context** — https://react.dev/learn/passing-data-deeply-with-context — *Focus on: when prop drilling becomes painful and context is the answer (and when it is overkill).*
- **React docs — Thinking in React** — https://react.dev/learn/thinking-in-react — *Focus on: deciding where each piece of state should live.*

**Search prompts:**
- `"react lift state up common ancestor"`
- `"react prop drilling vs context vs composition"`
- Ask an AI: *"Explain the decision: should this state be lifted, kept local, or shared via context? What are the three solutions to prop drilling (lift + pass, context, component composition), and when is each the right call? Why is over-lifting state as much a problem as prop drilling?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.1 — lifting state and prop drilling.
Completed so far: Tracks 1–5.

Build me a React + Tailwind app with two opposite problems: (1) state drilled through many
intermediate components that don't use it (ugly, fragile prop chains), and (2) state lifted TOO
high, so a value that only one leaf needs lives at the top and re-renders the whole tree on
every change. Two sibling components also need to share a piece of state but each keeps its own
copy, so they drift out of sync.

Tell me only how to run it. Do not name the issues. I diagnose where each piece of state SHOULD
live, then fix it: lift the shared state to the right common ancestor, push the over-lifted
state down, and replace the worst drill chains with composition or context. Make me justify each
placement and explain the tradeoffs of drilling vs context vs composition.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A prop drilled through 3–5 intermediate components that don't use it, just to reach a deep leaf. Fix: composition (pass the leaf as `children`) or context if many distant consumers.
- State lifted to the top that only one subtree needs, causing whole-tree re-renders on change. Fix: colocate it lower.
- Two siblings each holding their own copy of state that should be shared (they drift). Fix: lift to the nearest common ancestor and pass down.
- Recognizing that the answer is sometimes composition (no lifting/context at all) — restructure so the consumer is a child whose content is passed in.

The principle: **state belongs at the closest common ancestor of everything that needs it — no higher, no lower.** Too low and siblings can't share; too high and unrelated subtrees re-render. Prop drilling, context, and composition are three tools for delivering state to where it's needed, each with different costs.
</details>

---

## Module 6.2 — Implement A State Store From Scratch (Redux Core)

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS (then React binding)

### Prep — Read Before You Start

**Primary sources:**
- **Redux — createStore source** — https://github.com/reduxjs/redux/blob/master/src/createStore.ts — *Read the actual implementation (~100 lines). This is what you build.*
- **Redux — compose source** — https://github.com/reduxjs/redux/blob/master/src/compose.ts — *The 10-line function behind middleware chaining.*
- **Redux — Middleware (derivation)** — https://redux.js.org/understanding/history-and-design/middleware — *Focus on: deriving the `store => next => action` signature from first principles.*
- **Dan Abramov — You Might Not Need Redux** — https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367 — *Context for when this pattern earns its weight.*

**Search prompts:**
- `"redux createStore implementation subscribe dispatch"`
- `"redux middleware chain compose explained"`
- Ask an AI: *"Walk me through implementing Redux's core from scratch: `createStore` with `getState`/`dispatch`/`subscribe`, `combineReducers`, and `applyMiddleware`. Explain the middleware signature `store => next => action` and how `compose` chains middleware. Then how would I bind this store to React with `useSyncExternalStore`?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.2 — build a state store from scratch, build
challenge. Completed so far: Tracks 1–5, Module 6.1.

Give me a starter with a UI (a counter + a todo list) already wired to a `store` that does not
yet exist, plus failing assertions. My job is to implement the store core in vanilla JS:
`createStore(reducer, enhancer)` with `getState`/`dispatch`/`subscribe`(+unsubscribe);
`combineReducers`; a `loggerMiddleware`; and `applyMiddleware`. Then bind it to a small React
view using `useSyncExternalStore`. Give me no solution code.

Review me strictly. Before I write `createStore`, make me state what it must track. For
middleware, make me explain what `next` is when two middlewares are chained. If a reducer
mutates state or `subscribe` doesn't return an unsubscribe, ask me why that breaks things. After
it works, make me read the real Zustand source and articulate what Zustand traded away to be
simpler.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `createStore`: holds current state + a listener set; `getState` returns it; `dispatch` runs `state = reducer(state, action)` then notifies all listeners; `subscribe` adds a listener and returns an unsubscribe; if an enhancer is passed, returns `enhancer(createStore)(reducer)`.
- `combineReducers`: returns a reducer that calls each slice reducer with its slice and assembles the combined state.
- Middleware: `store => next => action => {...}`; `applyMiddleware` wraps `dispatch` so each middleware's `next` is the next middleware's dispatch, with the original `dispatch` at the end of the chain (via `compose`).
- React binding via `useSyncExternalStore(subscribe, getState)` so components re-render on store changes safely (no tearing).
- Common mistakes: mutating state in reducers; `subscribe` not returning unsubscribe; misunderstanding the middleware `next` chain; notifying listeners with stale state.

The principle: **a store is just current state + a dispatch that runs a pure reducer + a subscription list — and middleware is function composition around dispatch.** Building it demystifies every state library; `useSyncExternalStore` is the correct, tearing-safe bridge to React. Reading Zustand afterward shows how much ceremony Redux's guarantees cost.
</details>

---

## Module 6.3 — Zustand & Minimal State

**Difficulty:** ●●●○○ · **Format:** Build challenge + broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Zustand — Documentation** — https://zustand.docs.pmnd.rs — *Focus on: the `create` API, selectors, and that components subscribe to slices.*
- **Zustand — vanilla.ts source** — https://github.com/pmndrs/zustand/blob/main/src/vanilla.ts — *Read it; it's short and echoes what you built in 6.2.*
- **Zustand — Preventing rerenders with selectors** — https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow — *Focus on: selecting minimal slices and shallow equality.*

**Search prompts:**
- `"zustand selector prevent rerender"`
- `"zustand vs redux vs context"`
- Ask an AI: *"Explain Zustand: how `create` makes a store, how selectors let a component subscribe to only the slice it needs, and why selecting the whole store (or returning a new object from a selector) causes unnecessary re-renders. When is Zustand the right choice over context or Redux?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.3 — Zustand and minimal state. Completed so
far: Tracks 1–5, Modules 6.1–6.2.

Two parts.

Part 1 (build): give me a spec for a feature with shared global-ish state (e.g. a cart, or a
multi-panel filter) and have me model it with Zustand — actions colocated in the store,
components subscribing via narrow selectors.

Part 2 (debug): then give me a Zustand app with performance bugs: components selecting the whole
store (re-render on any change); a selector returning a new object/array each call (defeats
equality, re-renders always); and an action mutating state outside the `set` updater.

Do not name the bugs. Make me use the React DevTools Profiler to find the over-subscribing
components, fix the selectors (and shallow compare where needed), and explain why a narrow
selector is the whole point and why a new-object selector breaks it.
```

### 🔒 SPOILER — What It Tests / The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Part 1: a `create` store with state + actions colocated; components use `useStore(s => s.thePieceINeed)` selectors so they only re-render when that slice changes.

Part 2 — likely bugs:
- A component selecting the entire store (`useStore(s => s)`) so it re-renders on every change. Fix: select the minimal slice.
- A selector returning a NEW object/array each call (`s => ({a: s.a, b: s.b})`) — fails reference equality, re-renders every time. Fix: select primitives separately, or use `useShallow`/a shallow comparator.
- An action mutating state directly instead of using `set`, so subscribers aren't notified / state is corrupted.

The principle: **Zustand's leverage is selector-based subscription: a component re-renders only when its selected slice changes — so selecting too much, or returning fresh references from a selector, throws that away.** It's the from-scratch store of 6.2 with an ergonomic selector layer; the discipline is selecting narrowly and stably.
</details>

---

## Module 6.4 — Server State vs Client State

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **TanStack Query — Overview / Motivation** — https://tanstack.com/query/latest/docs/framework/react/overview — *Focus on: the distinction between server state (async, shared, can go stale) and client state (synchronous, owned).*
- **TanStack Query — Does this replace Redux?** — https://tanstack.com/query/latest/docs/framework/react/community/does-react-query-replace-redux — *Focus on: why server cache is a different problem from UI state.*
- **Kent C. Dodds — Application State Management** — https://kentcdodds.com/blog/application-state-management-with-react — *Focus on: separating kinds of state.*

**Search prompts:**
- `"server state vs client state react query"`
- `"why put server data in redux is wrong"`
- Ask an AI: *"Explain the difference between server state and client state. Why is treating fetched server data as ordinary client state (e.g. storing it in Redux/useState and manually syncing) the source of stale-data, caching, and refetch bugs? What does a server-cache library solve that a state manager doesn't?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.4 — server state vs client state. Completed
so far: Tracks 1–5, Modules 6.1–6.3.

Build me a React + Tailwind app that treats SERVER data as if it were client state: fetched data
copied into useState/a global store, manually refetched in effects, with hand-rolled loading and
error flags, cache invalidation done by ad-hoc re-fetching, and the same data fetched separately
in multiple components (no sharing/dedup) so they show inconsistent versions. It "works" but is
brittle: stale data, duplicate requests, inconsistent screens.

Tell me only how to run it (use a mock API whose value can change). Do not name the issues. I
diagnose by spotting which state is really server state being mismanaged. Make me separate the
two kinds of state (and recognize that the server-state half wants a cache library — set up in
the next module), and explain why server state has fundamentally different needs (staleness,
sharing, background refetch) than client state.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Fetched data copied into `useState`/a global store and treated as the source of truth, then manually kept fresh — drifts from the server.
- The same resource fetched independently in multiple components (no dedup/sharing), so different parts of the UI show different versions.
- Hand-rolled `loading`/`error`/`data` flags duplicated everywhere, inconsistently.
- Cache invalidation done by ad-hoc refetch calls, easy to forget after a mutation.
- The fix direction: recognize the fetched data as SERVER state (async, shared, can go stale, needs background refetch and a single cache) distinct from CLIENT state (UI toggles, form inputs, selections) — and that server state wants a dedicated cache (TanStack Query, next module), while only genuine UI state belongs in useState/Zustand.

The principle: **server state and client state are different problems: client state is synchronous and owned by your app; server state is an async, shared snapshot of remote data that can go stale and must be cached, deduped, and refetched.** Most "stale data / duplicate requests / inconsistent screens" bugs come from managing server state with client-state tools.
</details>

---

## Module 6.5 — TanStack Query: Caching & Deduplication

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **TanStack Query — Queries / Query Keys** — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys — *Focus on: how the query key identifies and dedupes a query.*
- **TanStack Query — Caching** — https://tanstack.com/query/latest/docs/framework/react/guides/caching — *Focus on: `staleTime` vs `gcTime`, the fresh→stale→inactive→garbage lifecycle.*
- **TanStack Query — Important Defaults** — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults — *Focus on: refetch-on-window-focus, retries, and the defaults that surprise people.*

**Search prompts:**
- `"tanstack query staleTime gcTime difference"`
- `"react query N+1 deduplication query key"`
- Ask an AI: *"Explain TanStack Query: how the query key dedupes identical requests across components, what `staleTime` and `gcTime` each control (fresh vs stale vs garbage-collected), and the default behaviors (refetch on focus/reconnect, retries). Walk me through fixing an N+1 fetch pattern with shared query keys."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.5 — TanStack Query caching and dedup.
Completed so far: Tracks 1–5, Modules 6.1–6.4.

Build me a React + Tailwind app (mock API that logs every call) with an N+1 fetch problem and
cache misuse: a list of posts where each post renders a `UserBadge` that fetches its author in a
`useEffect` — 20 posts, 3 unique authors, so ~20 requests instead of 3; a list refetched on every
mount with no caching; and a query whose key is constructed wrongly so identical requests are NOT
deduped (or unrelated ones collide). The console should show far more requests than necessary.

Tell me only how to run it. Do not name the issues. I diagnose by counting network requests and
comparing unique resources to requests fired. Make me convert to TanStack Query with correct
query keys (so the 3 authors are fetched once and shared), tune `staleTime`, and explain how the
key drives deduplication and what the cache lifecycle does.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- N+1: each `UserBadge` fetches its author independently in an effect, so 20 posts → 20 author requests despite only 3 unique authors. Fix: `useQuery(['user', userId], ...)` — TanStack Query dedupes by key, so each unique author is fetched once and shared across all badges.
- A list refetched on every mount with no cache. Fix: a query with an appropriate `staleTime` so remounts serve cached data.
- A query key that's wrong: too coarse (different requests collide and return wrong data) or unstable (identical requests not deduped). Fix: a precise, serializable key array.
- Understanding `staleTime` (how long data is "fresh" and won't refetch) vs `gcTime` (how long unused data stays cached before garbage collection).

The principle: **the query key is the identity of a piece of server state — requests with the same key are deduped and share one cache entry, and `staleTime`/`gcTime` govern freshness and retention.** The N+1 disappears not by batching but because many components asking for the same key get one shared, cached request.
</details>

---

## Module 6.6 — Optimistic Updates & Rollback

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **TanStack Query — Mutations** — https://tanstack.com/query/latest/docs/framework/react/guides/mutations — *Focus on: `useMutation`, and the `onMutate`/`onError`/`onSettled` lifecycle.*
- **TanStack Query — Optimistic Updates** — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates — *Focus on: snapshotting previous state in `onMutate`, rolling back in `onError`, invalidating in `onSettled`.*
- **React docs — useOptimistic** — https://react.dev/reference/react/useOptimistic — *Focus on: the React-native optimistic UI primitive.*

**Search prompts:**
- `"tanstack query optimistic update rollback onMutate"`
- `"optimistic ui pattern snapshot rollback"`
- Ask an AI: *"Explain optimistic updates with rollback: in `onMutate`, cancel in-flight refetches, snapshot the current cache, and apply the optimistic change; in `onError`, restore the snapshot; in `onSettled`, invalidate to reconcile with the server. Why is each step necessary? What goes wrong if you skip the snapshot or the cancel?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.6 — optimistic updates and rollback.
Completed so far: Tracks 1–5, Modules 6.1–6.5.

Build me a React + Tailwind app (mock API with ~30% failure rate and latency) where mutations
(toggle, edit, delete) feel sluggish and buggy: changes only appear after the server responds
(no optimism, laggy UX); and where an "optimistic" attempt exists, it never ROLLS BACK on
failure so the UI shows a change the server rejected; plus a missing cancel so an in-flight
refetch clobbers the optimistic state.

Tell me only how to run it (let failures actually happen). Do not name the bugs. I implement
proper optimistic updates with `useMutation`: cancel queries + snapshot in `onMutate`, roll back
in `onError`, invalidate in `onSettled`. Make me explain why each step exists and prove rollback
works by triggering a failure.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Mutations that wait for the server before updating the UI (no optimism) — laggy.
- An optimistic update with no rollback: on failure the UI keeps the change the server rejected, so the UI lies. Fix: snapshot in `onMutate`, restore it in `onError`.
- No `cancelQueries` in `onMutate`, so a refetch already in flight resolves and overwrites the optimistic value.
- No `invalidateQueries` in `onSettled`, so the cache never reconciles with the server's actual result.
- The full correct pattern: `onMutate` (cancel in-flight queries, snapshot previous, apply optimistic), `onError` (rollback to snapshot), `onSettled` (invalidate to refetch truth).

The principle: **optimistic UI applies the change immediately and reconciles with the server afterward — which only works if you snapshot before changing (to roll back), cancel in-flight refetches (so they don't clobber you), and invalidate after (to converge on the server's truth).** Skip any step and the UI either lags, lies on failure, or flickers from a racing refetch.
</details>

---

## Module 6.7 — Offline-First With IndexedDB

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **MDN — Using IndexedDB** — https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB — *Focus on: object stores and transactions.*
- **idb library** — https://github.com/jakearchibald/idb — *Focus on: the promise-based API you'll use.*
- **MDN — online/offline events & navigator.onLine** — https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event — *Focus on: detecting connectivity to trigger sync.*
- **Ink & Switch — Local-first software** — https://www.inkandswitch.com/local-first/ — *Conceptual depth (read after building).*

**Search prompts:**
- `"offline first indexeddb sync queue pending"`
- `"online event auto sync when reconnect"`
- Ask an AI: *"Walk me through an offline-first flow: write to IndexedDB first, update the UI immediately, attempt server sync if online, leave records 'pending' if offline, and auto-sync when the `online` event fires. How do I prevent duplicate syncs if the sync routine runs concurrently?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.7 — offline-first with IndexedDB, build
challenge. Completed so far: Tracks 1–5, Modules 6.1–6.6. Raise the difficulty.

Give me a starter: a React + Tailwind todo app with an online/offline indicator and sync-status
badges, a mock API (latency + ~30% failure), and stubbed IndexedDB and handler functions. My job
is to implement the offline-first flow: writes go to IndexedDB first and show immediately (marked
pending), sync to the server if online, stay pending if offline, and AUTO-SYNC when connectivity
returns — with a guard so concurrent sync runs don't duplicate requests. Give me no solution
code.

Review me strictly. Before I code, make me state the order of operations on a write (UI, DB,
server — which first and why). Ask me what happens if the sync routine runs three times at once.
Ask me the exact event name for reconnection. Make me prove it works by going offline (DevTools),
adding items, and coming back online.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Write order: persist to IndexedDB first (durability), update React state immediately (instant UI, marked `pending`), then attempt server sync if online; on success mark `synced`, on failure/offline leave `pending`.
- IndexedDB ops via `idb` with correct transaction scopes; a `getPending` query for unsynced records.
- An `online` event listener that triggers `syncPending`; a CONCURRENCY GUARD (a flag or lock) so overlapping sync runs don't double-submit the same pending records (the classic race).
- Failures (30%) leave records pending for the next sync attempt; idempotency considered.
- Common mistakes: syncing before persisting locally (data loss on crash); no concurrency guard (duplicate creates); not handling partial-batch failures; wrong event name; UI not reflecting pending vs synced.

The principle: **offline-first means local storage is the source of truth for the write, the UI reflects it instantly, and the server is reconciled opportunistically — with connectivity events driving sync and a concurrency guard preventing duplicate submissions.** The hard parts are ordering (local before remote), the reconnect trigger, and the sync race.
</details>

---

## Module 6.8 — Forms: Controlled, Uncontrolled & Libraries

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React docs — `<input>` / Controlled components** — https://react.dev/reference/react-dom/components/input — *Focus on: controlled vs uncontrolled inputs and the `value`/`onChange` contract.*
- **React docs — You Might Not Need an Effect (form derived state)** — https://react.dev/learn/you-might-not-need-an-effect — *Focus on: not syncing form-derived state via effects.*
- **React Hook Form — docs** — https://react-hook-form.com/get-started — *Focus on: uncontrolled-by-default performance and the validation model.*

**Search prompts:**
- `"react controlled vs uncontrolled input warning"`
- `"react form re-render every keystroke performance"`
- `"react hook form why uncontrolled"`
- Ask an AI: *"Explain controlled vs uncontrolled inputs in React, and the 'a component is changing an uncontrolled input to controlled' warning. Why can large controlled forms re-render on every keystroke, and how do libraries like React Hook Form avoid it with uncontrolled inputs + refs? When is each approach right?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.8 — forms. Completed so far: Tracks 1–5,
Modules 6.1–6.7.

Build me a React + Tailwind form app with realistic form bugs: an input that flips between
uncontrolled and controlled (triggers the React warning, value sometimes resets); a large form
where every keystroke re-renders the whole form (all fields controlled at a high level) causing
lag; validation logic run in effects that lags a render behind the input; and a number/checkbox
input mishandled (string vs number, checked vs value).

Tell me only how to run it. Do not name the bugs. I diagnose: fix the controlled/uncontrolled
consistency, restructure or isolate state so typing doesn't re-render everything (or move to an
uncontrolled approach), compute validation during render not in effects, and handle input types
correctly. Make me explain the controlled/uncontrolled model and the performance tradeoff.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- An input whose `value` is sometimes `undefined`/`null` then a string — flipping uncontrolled→controlled, producing React's warning and value resets. Fix: always provide a defined value (e.g. `value={x ?? ''}`).
- A large form with all field state controlled at a top-level component, so every keystroke re-renders all fields (lag). Fix: isolate field state lower, or use uncontrolled inputs (refs / React Hook Form) so keystrokes don't re-render siblings.
- Validation computed in a `useEffect` keyed on the value, so the error message lags one render behind. Fix: derive validation during render.
- Input-type handling bugs: reading `e.target.value` (string) where a number was needed, or `value` instead of `checked` for a checkbox.

The principle: **a controlled input ties its value to state (predictable, but re-renders on every change); an uncontrolled input lets the DOM hold the value (fast, accessed via ref) — and mixing them, or controlling a huge form too high, causes warnings and lag.** Validation is derived state (compute in render), and input types need correct value extraction.
</details>

---

## Module 6.9 — Data Fetching Patterns & The N+1

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **TanStack Query — Parallel & Dependent Queries** — https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries — *Focus on: dependent (waterfall) vs parallel queries and when each is correct.*
- **TanStack Query — Query invalidation** — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation — *Focus on: invalidating related data after changes.*
- **Kent C. Dodds — Colocation** — https://kentcdodds.com/blog/colocation — *Focus on: colocating fetching with the component that needs it without creating waterfalls.*

**Search prompts:**
- `"react request waterfall vs parallel fetch"`
- `"tanstack query dependent queries enabled"`
- Ask an AI: *"Explain request waterfalls: when does fetching create an accidental sequential chain (each request waiting for the previous) versus legitimate dependent queries? How do I parallelize independent requests, and when is a dependent query (`enabled`) actually required? Contrast with the N+1 problem."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.9 — data fetching patterns and waterfalls.
Completed so far: Tracks 1–5, Modules 6.1–6.8.

Build me a React + Tailwind app (mock API with latency) with fetching-architecture problems: an
accidental request WATERFALL where independent resources are fetched sequentially (each component
waits for its parent's data even though it didn't need to), making the page slow; an N+1 pattern;
and a place where a genuinely dependent query is fetched in parallel and breaks (it needed the
first result). The page loads far slower than necessary.

Tell me only how to run it and how to read the Network timeline (sequential vs parallel bars). Do
not name the issues. I diagnose by looking at the waterfall shape. Make me parallelize the
independent requests, fix the N+1 with shared queries, and correctly sequence the truly dependent
one with `enabled`. Make me explain why the waterfall happened and the difference between
accidental waterfalls and necessary dependencies.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- An accidental waterfall: a child only starts fetching after its parent renders with data, even though the child's request didn't actually depend on the parent's result — serializing independent requests. Fix: hoist/parallelize so independent queries fire together.
- N+1: many components each fetching a related resource individually (fix via shared query keys / batching).
- A truly dependent query (needs an id from a first request) fired in parallel and erroring or fetching with `undefined`. Fix: gate it with `enabled: !!id` so it waits.
- Network timeline shows staircase (waterfall) bars before, parallel bars after (except the one legitimately dependent).

The principle: **fetch independent data in parallel and dependent data in sequence — accidental waterfalls come from coupling requests that didn't need coupling, while real dependencies must wait.** Reading the network timeline's shape (staircase vs parallel) tells you which you have, and `enabled`/query keys express the correct dependency graph.
</details>

---

## Module 6.10 — WebSockets & Real-Time Data

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **MDN — WebSockets API** — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API — *Focus on: the connection lifecycle (open/message/close/error), and that it's a persistent bidirectional channel.*
- **MDN — Writing WebSocket client applications** — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications — *Focus on: reconnection, and cleaning up on unmount.*
- **MDN — Server-Sent Events** — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events — *Focus on: when SSE (one-way) suffices vs WebSockets (two-way).*

**Search prompts:**
- `"websocket react useEffect cleanup reconnect"`
- `"websocket vs server-sent events vs polling"`
- Ask an AI: *"Explain the WebSocket lifecycle and how to integrate it with React: opening a connection in an effect, handling messages into state, cleaning up on unmount, and reconnecting with backoff. When should I use WebSockets vs Server-Sent Events vs polling? What are the common leak/duplicate-connection bugs?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.10 — WebSockets and real-time, build
challenge. Completed so far: Tracks 1–5, Modules 6.1–6.9. Raise the difficulty.

Give me a spec to build a real-time feature (a live chat, a presence indicator, or a live price
ticker) over WebSockets, with a mock WS server (or an echo/public test endpoint). Requirements:
open the connection correctly within the React lifecycle, route incoming messages into state,
clean up on unmount (no duplicate/leaked connections across re-renders), reconnect with backoff
on drop, and show connection status. Give me no solution code.

Review me strictly. If my effect opens a new socket on every render or never closes the old one,
do not tell me — ask me when the effect runs and what its cleanup should do. If reconnection
hammers the server, ask me about backoff. Make me explain the WS lifecycle, why cleanup matters
here specifically, and when SSE/polling would have been the better tool.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A single connection opened in an effect (correct deps so it isn't recreated needlessly), with handlers for `open`/`message`/`close`/`error`; messages routed into state.
- Cleanup that closes the socket on unmount and before re-running the effect — preventing duplicate/leaked connections (a very common bug when the effect re-runs).
- Reconnection with exponential backoff (not a tight retry loop hammering the server) and a max/jitter; connection-status UI.
- Consideration of whether the use case needed bidirectional WS at all, or whether SSE (server→client only) or polling would be simpler/cheaper.
- Common mistakes: opening a socket per render; no cleanup (leaks, duplicate message handling); reconnect storms with no backoff; updating state after unmount; not handling the closed/errored state.

The principle: **a WebSocket is a persistent, bidirectional connection whose lifecycle must be tied carefully to the React lifecycle — opened once, cleaned up on unmount, reconnected with backoff — or you leak connections and duplicate handlers.** Choosing WS vs SSE vs polling is a real architectural decision; persistent two-way state is what justifies the WS complexity.
</details>

---

## Module 6.11 — State Machines (Modeling With XState Concepts)

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Statecharts — The World of Statecharts (David Khourshid)** — https://statecharts.dev — *Focus on: states, events, transitions, and why explicit states prevent impossible combinations.*
- **XState — Introduction** — https://stately.ai/docs/xstate — *Focus on: the machine model (states, events, context, guards) — you can model the concepts even without the library.*
- **React docs — Reducer (revisit)** — https://react.dev/learn/extracting-state-logic-into-a-reducer — *A reducer is a state machine; relate the two.*

**Search prompts:**
- `"finite state machine ui impossible states"`
- `"statechart vs boolean flags react"`
- Ask an AI: *"Explain finite state machines / statecharts for UI: states, events, transitions, guards. Why does modeling a feature (like a media player, a checkout flow, or a fetch with retry) as explicit states eliminate the 'impossible state' bugs that boolean flags allow? Show me a machine vs the equivalent tangle of booleans."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.11 — state machines, build challenge.
Completed so far: Tracks 1–5, Modules 6.1–6.10.

Give me a spec for a feature notorious for boolean-flag spaghetti — e.g. a media player
(idle/loading/playing/paused/buffering/ended/error), a multi-step checkout with back/retry, or a
form submission with validation+network states. Describe the legal states and transitions. My
job: model it as an explicit state machine (you can use a reducer in the XState style, or XState
itself), making impossible states unrepresentable and every transition explicit. Give me no
solution code.

Review me strictly. If my model allows an illegal combination (e.g. playing AND error, or
loading AND success), or a transition that shouldn't exist, do not fix it — ask me to draw the
state diagram and point to the illegal edge. Make me explain how an explicit machine prevents the
bugs that scattered booleans invite.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A single explicit `state` (a discriminated union of named states) rather than a pile of independent booleans — so `playing && error` or `loading && success` simply cannot be represented.
- Events that drive transitions; transitions defined per-state (an event is only handled in states where it's legal); guards for conditional transitions; context for associated data.
- A drawable state diagram where every node and edge is intentional; illegal transitions are absent by construction.
- Common mistakes: modeling with booleans that allow contradictions; an event handled identically in all states (no real machine); side effects tangled into the model instead of as transition effects; states that should be distinct collapsed into flags.
- This connects directly to Module 5.10 (useReducer) and to TypeScript discriminated unions (Track 8).

The principle: **a state machine makes the legal states and transitions explicit and the illegal ones unrepresentable — replacing combinatorial boolean flags (which permit contradictory states) with a single, diagrammable model.** Complex async/interaction flows become correct by construction rather than by careful flag-juggling.
</details>

---

## Module 6.12 — Real-Time Collaboration: Operational Transforms

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Wikipedia — Operational transformation** — https://en.wikipedia.org/wiki/Operational_transformation — *Focus on: the consistency model and the convergence (diamond) property.*
- **OT visualizer** — https://operational-transformation.github.io — *Play with retain/insert/delete and concurrent ops.*
- **MDN — BroadcastChannel** — https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel — *Focus on: simulating two clients across tabs without a server.*
- **Joseph Gentle — CRDTs are the Future** — https://josephg.com/blog/crdts-are-the-future/ — *Context: OT vs CRDT tradeoffs.*

**Search prompts:**
- `"operational transform apply transform compose"`
- `"OT convergence diamond property concurrent edits"`
- Ask an AI: *"Explain operational transformation for collaborative text: operations as retain/insert/delete, `apply(doc, op)`, and the crucial `transform(opA, opB)` that makes two concurrent ops converge to the same document regardless of order (the diamond property). Walk me through transforming two concurrent inserts at the same position."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.12 — collaborative editing with operational
transforms, build challenge. Completed so far: Tracks 1–5, Modules 6.1–6.11. This is the track
capstone — make it genuinely hard.

Give me a starter: two editor panels side by side in one page (Client A and B) sharing edits over
BroadcastChannel (simulating the network), with an `ot.js` exposing stubbed `apply(doc, op)`,
`transform(op1, op2)`, and `compose(op1, op2)`, and the textareas wired to produce ops on change.
My job: implement OT so simultaneous edits in both panels CONVERGE to the same document. Give me
no solution code.

Review me strictly. Before code, make me explain the diamond/convergence property in my own
words. For `apply`, make me trace an op over a string by hand. For `transform`, make me work out
where two concurrent inserts at the same position end up and why. When the docs diverge after
simultaneous typing, ask me which function is wrong and how I can tell. Make me prove convergence
experimentally.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `apply(doc, op)`: walk the op components (retain N, insert S, delete N) over the document, producing the new string; the retain+delete counts must cover the whole document length.
- `transform(op1, op2)`: the hard one — given two ops authored against the SAME base document, produce op1′ and op2′ such that `apply(apply(doc, op1), op2′) === apply(apply(doc, op2), op1′)` (convergence / diamond property). Handle the cases of insert-vs-insert (tie-break by a consistent rule so both clients agree on ordering), insert-vs-delete, delete-vs-delete overlap, etc.
- `compose` to combine sequential ops; integrating transform on receiving a remote op against any pending local ops.
- Proving convergence: type "AAA" in A and "BBB" in B simultaneously and confirm both panels end identical.
- Common mistakes: a `transform` that doesn't preserve convergence (the docs diverge under concurrency); inconsistent tie-breaking (clients disagree on insert order); off-by-one in position shifting; not transforming pending local ops against incoming remote ops.

The principle: **OT keeps concurrent editors consistent by transforming each incoming operation against the operations applied since its base, guaranteeing all clients converge to the same document regardless of order — the diamond property.** `transform` is mathematically the crux; getting tie-breaking and position-shifting exactly right is what makes collaboration not corrupt the document.
</details>

---

## Module 6.13 — Pagination, Infinite Scroll & Virtualization

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **TanStack Query — Infinite Queries** — https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries — *Focus on: `useInfiniteQuery`, page params, and `getNextPageParam`.*
- **TanStack Virtual — Introduction** — https://tanstack.com/virtual/latest/docs/introduction — *Focus on: windowing, `estimateSize`, `overscan`.*
- **web.dev — Virtualize large lists** — https://web.dev/articles/virtualize-long-lists-react-window — *Focus on: the mental model and the three numbers (count, item size, container size).*

**Search prompts:**
- `"react virtualization windowing constant dom nodes"`
- `"useInfiniteQuery getNextPageParam"`
- `"pagination vs infinite scroll vs virtualization"`
- Ask an AI: *"Distinguish pagination, infinite scroll, and virtualization (windowing) — they solve different problems and combine. Explain list virtualization math: given scrollTop, item height, and container height, which item indices render? Why does the DOM node count stay constant while scrolling 100k rows?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.13 — pagination, infinite scroll,
virtualization, build challenge. Completed so far: Tracks 1–5, Modules 6.1–6.12.

Give me a spec to build a feed of 100,000 items that must scroll at 60fps and load incrementally:
(1) implement VIRTUALIZATION yourself from scratch (compute visible indices from scrollTop/item
height/container height; render only the visible window + overscan; keep total scroll height
correct), then (2) add INFINITE loading of pages as the user nears the end (IntersectionObserver
sentinel + `useInfiniteQuery`-style paging). Give me no solution code for the virtualization
math.

Review me strictly. Before coding, make me compute: at 52px/row and 100k rows, what's the total
height, and given scrollTop 5200 with a 600px container, which indices are visible? If my DOM node
count grows while scrolling, ask me what should stay constant. If infinite load double-fetches,
ask me about the sentinel guard. Make me explain the windowing math and how it composes with paging.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Virtualization: an outer scroll container of fixed height; an inner spacer of `totalItems * itemHeight` so the scrollbar is correct; compute `startIndex = floor(scrollTop / itemHeight)`, `endIndex = ceil((scrollTop + containerHeight) / itemHeight)`, render only `[start - overscan, end + overscan]`, absolutely positioned at `index * itemHeight`. DOM node count stays ~constant (visible window + overscan) regardless of total rows.
- Infinite loading: an IntersectionObserver sentinel near the end triggers the next page; a guard prevents double-fetch; new pages append to the dataset and the virtualizer recomputes total height.
- Re-virtualization on filter/resize (ResizeObserver for container height).
- Common mistakes: DOM nodes growing with scroll (not actually windowing); wrong total height (broken scrollbar); off-by-one in visible-range math; double-fetching on the sentinel; not handling variable item heights (acknowledge the harder case).

The principle: **virtualization renders only the visible window while faking full scroll height, so a 100k-row list costs the same DOM as a 20-row one — and it composes with infinite paging (load more data) and pagination (discrete pages), which are about data loading, not rendering.** The windowing math (indices from scroll position) is the core; the constant DOM node count is the proof it works.
</details>

---

## Module 6.14 — URL As State

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Next.js (App Router) + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **Next.js — useSearchParams / useRouter / usePathname** — https://nextjs.org/docs/app/api-reference/functions/use-search-params — *Focus on: reading and updating URL search params as state.*
- **MDN — URLSearchParams** — https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams — *Focus on: parsing and serializing query strings.*
- **web.dev — Stateful UX and the URL** — https://web.dev/articles/url-friendly — *Focus on: why shareable, restorable state belongs in the URL.*

**Search prompts:**
- `"url as state searchParams shareable filters react"`
- `"sync state to url query params next.js"`
- Ask an AI: *"Why should certain UI state (filters, search query, active tab, pagination) live in the URL rather than component state? What does putting it in the URL give you (shareable links, back/forward, refresh-survival)? How do I sync URL search params with the UI without infinite loops or losing other params?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 6.14 — URL as state. Completed so far:
Tracks 1–5, Modules 6.1–6.13. This closes the data/state track.

Scaffold me a Next.js (App Router) + Tailwind page (a filterable, searchable, paginated list)
where state that SHOULD live in the URL lives only in component state instead: refreshing loses
the user's filters/search/page; the back button doesn't undo a filter change; links aren't
shareable (a copied URL doesn't reproduce the view). Also plant a buggy attempt that DOES write to
the URL but clobbers unrelated params and causes a re-render loop.

Tell me only how to run it. Do not name the issues. I diagnose by testing refresh, back/forward,
and copy-link. Make me move the appropriate state into URL search params (reading via
`useSearchParams`, updating via the router without dropping other params or looping), keeping
genuinely ephemeral UI state local. Make me explain which state belongs in the URL and why.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- Filters/search/active-tab/page held only in component state: refresh resets them, back/forward doesn't traverse them, and copied links don't reproduce the view.
- A fix attempt that writes to the URL but rebuilds the query string from scratch, dropping other existing params (e.g. setting `?page=2` wipes `?q=...`). Fix: read current params, mutate one key, serialize back.
- A sync that creates a loop (URL update triggers a state update that triggers a URL update). Fix: treat the URL as the source of truth and derive state from it, rather than two-way syncing.
- Recognizing what should NOT go in the URL (transient UI like an open dropdown, unsaved input mid-edit).

The principle: **state that should be shareable, bookmarkable, and survive refresh/back-forward belongs in the URL — the URL is a legitimate, durable state container.** Read it as the source of truth and update individual params without clobbering others; keep only genuinely ephemeral UI state in component memory.
</details>

---

*End of Track 6 — State, Data & Async Architecture. 14 modules.*

*Next: Track 7 — Accessibility & Semantics in Practice, in its own file.*
