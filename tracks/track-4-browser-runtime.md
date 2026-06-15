# TRACK 4 — THE BROWSER AS A RUNTIME

*Now that you know the language, learn the machine it runs on. This track is about the browser as an execution environment: how it turns your HTML/CSS/JS into pixels, when it does work and when it cannot, and the Web Platform APIs that let you cooperate with it instead of fighting it. This is where "why is my page slow/janky/leaking" stops being mysterious.*

*Reminder: paste the Universal Teaching Contract (your root `CLAUDE.md`) is loaded automatically by Claude Code. Do the Prep before every module. Never read the spoiler first.*

---

## Module 4.1 — The Critical Rendering Path

**Difficulty:** ●●○○○ · **Format:** Investigation · **Stack:** Vanilla HTML/CSS/JS

### Prep — Read Before You Start

**Primary sources:**

- **web.dev — Critical rendering path** — https://web.dev/articles/critical-rendering-path — *Read the full series. Focus on: bytes → DOM, CSS → CSSOM, render tree, layout, paint.*
- **MDN — Populating the page: how browsers work** — https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work — *Focus on: parsing, the render tree, and why CSS and synchronous JS block rendering.*
- **web.dev — Render-blocking resources** — https://web.dev/articles/render-blocking-resources — *Focus on: which resources block the first paint and how to unblock them.*

**Search prompts:**

- `"critical rendering path dom cssom render tree"`
- `"render blocking css javascript first paint"`
- `"async vs defer script tag difference"`
- Ask an AI: *"Walk me through the critical rendering path from receiving HTML bytes to the first paint: DOM construction, CSSOM construction, the render tree, layout, and paint. Why does CSS block rendering? Why does a synchronous `<script>` block parsing? What is the difference between `async` and `defer` on a script?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.1 — the critical rendering path,
investigation. Completed so far: Tracks 1–3.

Build me a page that paints SLOWLY on first load for structural reasons (not because of heavy
computation): render-blocking CSS in the wrong place, a synchronous script in the <head> that
blocks parsing, a stylesheet that delays first paint, fonts that block text rendering. The
page content is small; the delay is all about HOW resources are loaded.

Tell me how to run it and how to record a Performance trace / view the Network waterfall with
throttling. Do not tell me what is blocking. I will identify, from the waterfall and the
trace, which resources are blocking the render and reorder/attribute them (async/defer,
preload, move CSS, font-display) to paint sooner. Make me narrate the critical rendering path
and explain why each resource was blocking.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A synchronous `<script>` in the `<head>` (no `async`/`defer`) that blocks HTML parsing until it downloads and executes. Fix: `defer` (or move to the end of body).
- A large or slow CSS file that blocks first paint (render-blocking by nature) — mitigate by inlining critical CSS or splitting.
- A web font with default `font-display` causing invisible text (FOIT) until the font loads. Fix: `font-display: swap` and `preconnect`/`preload`.
- Possibly a stylesheet `@import` chain that serializes downloads.
- The Network waterfall shows the blocking serialization; the Performance trace shows a late first paint.

The principle: **the browser must build the DOM and CSSOM before it can paint, and render-blocking CSS plus parser-blocking synchronous JS delay that first pixel.** `defer`/`async`, resource hints, critical-CSS, and `font-display` are how you stop blocking the path to first paint. Understanding the path turns "the page is slow to appear" into a precise, fixable diagnosis.

</details>

---

## Module 4.2 — The DOM API From Scratch (No innerHTML)

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Introduction to the DOM** — https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction — *Focus on: the DOM as a live tree of nodes, and the document/element/node distinction.*
- **MDN — Document.createElement / Node.appendChild / Element APIs** — https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement — *Focus on: creating, configuring, and inserting nodes programmatically.*
- **MDN — DocumentFragment** — https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment — *Focus on: building a subtree off-document and inserting it once to avoid repeated reflows.*
- **MDN — `textContent` vs `innerHTML` vs `innerText`** — https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent — *Focus on: the security and performance differences.*

**Search prompts:**

- `"createElement appendChild vs innerHTML"`
- `"documentfragment batch dom insertion performance"`
- `"textContent vs innerHTML xss"`
- Ask an AI: *"Why is building DOM with `createElement`/`appendChild` safer than `innerHTML` string concatenation (XSS)? And why is inserting nodes one-by-one into the live document slower than building a `DocumentFragment` and inserting once? Explain what triggers reflow on each insertion."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.2 — the DOM API from scratch, build
challenge. Completed so far: Tracks 1–3, Module 4.1.

Give me a spec to build a small dynamic UI (e.g. a searchable, sortable list rendered from a
JS data array) using ONLY the imperative DOM API — `createElement`, `textContent`,
`setAttribute`, `append`, `DocumentFragment`, and event listeners. No `innerHTML`, no
frameworks, no template strings injected as HTML. The list must rebuild/update as data
changes.

Give me no solution code. Review me strictly. If I batch poorly (insert nodes one at a time
into the live document in a loop), do not tell me — ask me how many times the browser had to
reflow and whether a fragment could reduce that. If I reach for `innerHTML`, ask me what
happens if a data field contains `<script>`. Make me explain the create/append/fragment model
and the reflow implications.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Building elements with `createElement`, setting text via `textContent` (not `innerHTML`), attributes via `setAttribute`/properties, and composing with `append`.
- Using a `DocumentFragment` to assemble many rows off-document and inserting once, minimizing reflows — instead of appending each row directly to the live list in a loop.
- Clean update logic that does not destroy and rebuild more than necessary.
- Security: `textContent` prevents injected markup from executing; `innerHTML` with untrusted data is an XSS vector (foreshadows Track 8 security).
- Common mistakes: per-item live insertion (repeated layout); `innerHTML +=` in a loop (re-parses the whole list every time and breaks existing references/listeners); using `innerHTML` with user data.

The principle: **the DOM is a live tree, and every mutation to the live document can trigger layout — so you build off-document with a fragment and insert once.** The imperative API is verbose but explicit; it also avoids the XSS and performance footguns of `innerHTML`. This is the substrate every framework sits on top of.

</details>

---

## Module 4.3 — Reflow & Repaint

**Difficulty:** ●●●○○ · **Format:** Investigation + broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Reflow** — https://developer.mozilla.org/en-US/docs/Glossary/Reflow — *Focus on: what reflow (layout) is and what triggers it.*
- **web.dev — Avoid large, complex layouts and layout thrashing** — https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing — *Focus on: what forces synchronous layout and how to batch reads/writes.*
- **Paul Irish — What forces layout/reflow (gist)** — https://gist.github.com/paulirish/5d52fb081b3570c81e3a — *A reference list of every property/method that forces synchronous layout.*

**Search prompts:**

- `"what triggers reflow repaint browser"`
- `"forced synchronous layout offsetHeight"`
- Ask an AI: *"What is the difference between reflow (layout) and repaint? Give me a list of the DOM reads (like `offsetHeight`, `getBoundingClientRect`) that force a synchronous layout. Why does reading a layout property right after writing a style cause a forced reflow?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.3 — reflow and repaint, investigation +
debug. Completed so far: Tracks 1–3, Modules 4.1–4.2.

Build me a page with an interaction that is sluggish because it triggers excessive reflows:
code that, in a loop over many elements, READS a layout property and then WRITES a style on
each iteration (interleaved reads and writes forcing synchronous layout every time). Also
include a change that triggers a full-page reflow where a cheaper change would have sufficed.

Tell me how to run it and how to record a Performance trace and watch CPU throttling. Do not
tell me what is slow. I will find the forced-layout pattern in the trace (lots of
"Layout"/"Recalculate Style" work), and fix it by batching all reads then all writes (or using
transforms). Make me explain what forces a synchronous layout and why read-then-write in a
loop is pathological.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A loop over elements that reads a layout property (`offsetHeight`/`offsetWidth`/`getBoundingClientRect`) and then writes a style, per element — so each iteration invalidates layout and the next read forces a synchronous reflow. Fix: read ALL measurements first, then write ALL changes (batch), or avoid layout reads entirely.
- A change to a layout-affecting property (width/height/top) where a `transform` would have avoided layout.
- The Performance trace shows repeated "Layout" events interleaved with script (the layout-thrash signature).

The principle: **the browser batches layout until you force it by reading a layout-dependent value while changes are pending — interleaving reads and writes defeats the batching and triggers a reflow per iteration.** Separate the read phase from the write phase, prefer compositor-only changes, and the thrash disappears.

</details>

---

## Module 4.4 — Layout Thrashing At Scale

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **web.dev — Layout thrashing (revisit)** — https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing — *Focus on: the batching pattern and `requestAnimationFrame` for visual updates.*
- **MDN — `requestAnimationFrame`** — https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame — *Focus on: scheduling reads/writes relative to the frame.*
- **FastDOM (concept)** — https://github.com/wilsonpage/fastdom — *Read the README to see how a library formalizes read/write batching across a frame.*

**Search prompts:**

- `"layout thrashing read write batching fastdom"`
- `"requestAnimationFrame measure mutate phases"`
- Ask an AI: *"Explain how to eliminate layout thrashing in a real app: the measure phase vs the mutate phase, and how to schedule writes in `requestAnimationFrame`. Why does reading `scrollTop` or `getBoundingClientRect` during a write loop tank performance, and how does separating phases fix it?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.4 — layout thrashing at scale. Completed
so far: Tracks 1–3, Modules 4.1–4.3. Raise the difficulty.

Build me a realistic feature that thrashes layout badly: e.g. a "masonry" or "equal-height
columns" routine, or a scroll-driven effect, that measures each element's geometry and then
sets styles based on it, all interleaved over hundreds of elements — causing dozens of forced
reflows per interaction and visible jank under CPU throttle.

Tell me how to run it and trace it. Do not reveal the structure of the fix. I will identify
the interleaved measure/mutate pattern, restructure into a single measure pass followed by a
single mutate pass (scheduling writes in rAF where appropriate), and re-measure the trace.
Make me explain the measure/mutate separation and why it collapses many reflows into one.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A routine that, element by element, reads geometry (`getBoundingClientRect`/`offsetHeight`/`scrollTop`) and immediately writes a derived style — so every read after a write forces a fresh layout. With hundreds of elements this is dozens-to-hundreds of synchronous reflows.
- Fix: a two-phase approach — collect ALL measurements into an array first (read phase), then apply ALL style mutations (write phase), ideally scheduling the mutate phase in `requestAnimationFrame`. The whole operation then costs roughly one layout instead of N.
- The trace shows a wall of "Layout" events before the fix and a single layout after.

The principle: **batching is the whole game: read everything, then write everything, and let layout happen once.** Interleaving forces the browser to re-resolve layout repeatedly because each write invalidates the geometry the next read depends on. `requestAnimationFrame` aligns the write phase with the browser's frame so visual updates are smooth.

</details>

---

## Module 4.5 — The Event Loop In The Browser (Rendering & Frames)

**Difficulty:** ●●●●○ · **Format:** Investigation · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — The event loop (revisit, browser focus)** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop — *Focus on: where rendering fits relative to tasks and microtasks.*
- **web.dev — Optimize long tasks** — https://web.dev/articles/optimize-long-tasks — *Focus on: how long tasks block input and rendering, and how to break them up (`yield`, `scheduler.yield`, chunking).*
- **MDN — `requestIdleCallback`** — https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback — *Focus on: running low-priority work in idle time.*

**Search prompts:**

- `"long task main thread block input rendering"`
- `"break up long task setTimeout scheduler yield"`
- Ask an AI: *"In the browser, where does rendering happen relative to the event loop's tasks and microtasks? What is a 'long task' and why does it block both user input and the next frame? Show me strategies to break a long computation into chunks so the page stays responsive."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.5 — the browser event loop, long tasks,
and frames. Completed so far: Tracks 1–3, Modules 4.1–4.4.

Build me a page with a button that runs a heavy synchronous computation (e.g. processing a
large dataset) on the main thread. While it runs, the page is frozen: input is ignored, a
spinner does not spin, and a typed input lags. The work is correct but monolithic.

Tell me how to run it and how to record a Performance trace showing the long task. Do not tell
me the fix. I will confirm the main thread is blocked (long task, dropped frames, ignored
input), then break the work into chunks yielded across tasks/frames (or move toward a worker
in the next module) so the UI stays responsive. Make me explain why a long synchronous task
blocks rendering and input, and where yielding returns control to the loop.
```

### 🔒 SPOILER — What It Tests / The Fix Space

<details>
<summary>Click only after you finish or give up</summary>

- A single long synchronous loop on the main thread that prevents the event loop from processing input events or running the rendering steps — so the page is frozen, the spinner stalls, and input is dropped. The Performance trace shows one long task (>50ms, often much longer) with no frames painted during it.
- Fixes to discover: chunk the work and yield between chunks (`setTimeout(0)`, `await scheduler.yield()` where available, or a chunked `requestAnimationFrame`/`requestIdleCallback` loop) so the loop can render and handle input between chunks; or recognize that the truly correct fix is offloading to a Web Worker (next module).
- The key observation: yielding does not make the work faster — it makes the UI responsive by letting the event loop interleave rendering and input between chunks.

The principle: **the main thread runs JavaScript AND rendering AND input handling, one task at a time — so a long synchronous task starves all of them.** Breaking work into chunks that yield to the event loop keeps the page responsive; CPU-bound work that cannot be chunked belongs off the main thread entirely.

</details>

---

## Module 4.6 — requestAnimationFrame & Smooth Animation

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `requestAnimationFrame`** — https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame — *Focus on: the timestamp argument, that it fires before the next paint, and per-frame scheduling.*
- **MDN — Animating with JavaScript / timing** — https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame#examples — *Focus on: delta-time based animation so motion is frame-rate independent.*
- **web.dev — Animations and performance (revisit)** — https://web.dev/articles/animations-and-performance — *Focus on: keeping per-frame work small.*

**Search prompts:**

- `"requestAnimationFrame vs setTimeout animation"`
- `"delta time animation frame rate independent"`
- Ask an AI: *"Why is `requestAnimationFrame` better than `setInterval(fn, 16)` for animation? Explain frame-rate independence: why animation that moves a fixed pixels-per-frame runs at different speeds on 60Hz vs 120Hz displays, and how using the timestamp delta fixes it."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.6 — requestAnimationFrame and smooth
animation. Completed so far: Tracks 1–3, Modules 4.1–4.5.

Build me a JS-driven animation (a moving object, a progress meter, a particle or two) that is
implemented badly: it uses `setInterval`/`setTimeout` for timing so it stutters and tears, and
it moves a FIXED amount per tick so its speed depends on the display refresh rate / tab
throttling. It should look janky and behave inconsistently.

Tell me how to run it. Do not reveal the fix. I will diagnose the timing approach, convert to
a `requestAnimationFrame` loop, and make the motion frame-rate independent using the timestamp
delta. Make me explain why rAF aligns with the paint cycle and why delta-time animation is
necessary for consistent speed across devices.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Using `setInterval(fn, 16)` / `setTimeout` for animation timing — not synchronized to the display's paint, causing stutter, tearing, and drift; also keeps running (wastefully) in background tabs.
- Moving a FIXED number of pixels per tick, so speed scales with refresh rate (twice as fast on a 120Hz screen) and changes when the browser throttles timers.
- Fix: a `requestAnimationFrame(loop)` recursion; compute elapsed time from the timestamp argument (`delta = now - last`) and move by `velocity * delta`, making motion frame-rate independent; rAF also pauses in background tabs automatically.
- Possibly per-frame layout-triggering writes (tie-in to compositor lessons) — animate via transform.

The principle: **`requestAnimationFrame` schedules your update to run right before the browser's next paint, synchronizing animation with the display, and basing movement on elapsed time (delta) makes speed independent of frame rate.** Timer-based animation drifts, stutters, and runs at the wrong speed across devices.

</details>

---

## Module 4.7 — IntersectionObserver

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Intersection Observer API** — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API — *Focus on: the observer/entries model, `isIntersecting`, `threshold`, `rootMargin`, and why it is far cheaper than scroll listeners.*
- **MDN — IntersectionObserverEntry** — https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry — *Focus on: `intersectionRatio`, `boundingClientRect`, and `target`.*
- **web.dev — Lazy-loading with IntersectionObserver** — https://web.dev/articles/lazy-loading-images — *Focus on: the canonical lazy-load and infinite-scroll patterns.*

**Search prompts:**

- `"intersection observer vs scroll event performance"`
- `"intersection observer infinite scroll lazy load"`
- Ask an AI: *"Explain the Intersection Observer API: what problem does it solve that a `scroll` event listener creates (jank from running on every scroll)? Walk me through `threshold`, `rootMargin`, and `isIntersecting`, and show me the structure of a lazy-load and an infinite-scroll implementation."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.7 — IntersectionObserver, build challenge.
Completed so far: Tracks 1–3, Modules 4.1–4.6.

Give me a spec to build three things WITHOUT scroll-event listeners: (1) lazy-loading images
that only load as they near the viewport, (2) an infinite-scroll list that fetches the next
page when a sentinel element approaches the bottom, and (3) a "scroll spy" that highlights the
nav item for the section currently in view. Give me no solution code.

Review me strictly. If I reach for a `scroll` listener, do not tell me about the observer —
ask me how often my scroll handler runs and what that costs, then point me to a more efficient
primitive. If my infinite scroll double-fetches or my lazy-load triggers too late, ask me what
`rootMargin` and `threshold` are doing. Make me explain why IntersectionObserver is cheaper
than scroll math.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- An `IntersectionObserver` observing image elements; on `isIntersecting`, swap `data-src` → `src` and unobserve. A `rootMargin` (e.g. `200px`) preloads before the image is visible.
- Infinite scroll: observe a sentinel element at the list's end; when it intersects, fetch the next page; guard against double-fetch (unobserve during fetch or track a loading flag).
- Scroll spy: observe each section; update the active nav link based on which entries are intersecting (often using `intersectionRatio` or `rootMargin` to define the "active zone").
- Common mistakes: using scroll listeners + `getBoundingClientRect` (jank, forced layout); not unobserving after firing (repeated triggers); double-fetching in infinite scroll; wrong `threshold` so callbacks fire too often or never.

The principle: **IntersectionObserver lets the browser tell you, asynchronously and off the main thread, when elements cross visibility thresholds — replacing expensive per-scroll geometry math.** It is the right primitive for lazy-loading, infinite scroll, reveal-on-scroll, and visibility analytics.

</details>

---

## Module 4.8 — ResizeObserver & MutationObserver

**Difficulty:** ●●●○○ · **Format:** Build challenge + broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — ResizeObserver** — https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver — *Focus on: observing element size (not just window), `contentRect`/`borderBoxSize`, and the resize-loop warning.*
- **MDN — MutationObserver** — https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver — *Focus on: observing DOM changes (childList, attributes, subtree) and batched mutation records.*
- **MDN — ResizeObserver constructor** — https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver/ResizeObserver — *Focus on: choosing between Resize, Mutation, and Intersection observers.*

**Search prompts:**

- `"resizeobserver element size vs window resize"`
- `"mutationobserver watch dom changes"`
- `"resizeobserver loop limit exceeded error"`
- Ask an AI: *"When do I use ResizeObserver vs MutationObserver vs IntersectionObserver? Give me a concrete use case for each where the others would not work. Then explain the 'ResizeObserver loop completed with undelivered notifications' error and how a resize-triggered write can cause it."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.8 — ResizeObserver and MutationObserver.
Completed so far: Tracks 1–3, Modules 4.1–4.7.

Two parts.

Part 1 (build): give me a spec to build (a) a component that reflows its internal layout based
on its OWN element size using ResizeObserver (not window resize), and (b) a feature that
reacts to DOM changes it does not control — e.g. watching a container for added nodes with
MutationObserver and enhancing them. No solution code.

Part 2 (debug): give me code that misuses these — a ResizeObserver that writes a size-changing
style inside its own callback, creating an infinite resize loop / the "undelivered
notifications" warning; a MutationObserver that observes the wrong options so it never fires;
or a window-resize listener used where element-size observation was needed.

Do not reveal the bugs. Make me explain which observer is correct for which signal and why
the resize loop happens.
```

### 🔒 SPOILER — What It Tests / The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Part 1: a `ResizeObserver` on the component reading `entry.contentRect` to switch layouts based on its own width (the JS analog of container queries); a `MutationObserver` with the right `{ childList: true, subtree: true }` options reacting to inserted nodes.

Part 2 — likely bugs:

- A ResizeObserver whose callback writes a style that changes the observed element's size, triggering another resize → loop / "ResizeObserver loop completed with undelivered notifications." Fix: avoid feedback (write to a non-observed property, debounce, or guard).
- A MutationObserver `observe(target, options)` with missing/wrong options (e.g. forgetting `childList` or `subtree`) so it never reports the changes you care about.
- A `window.addEventListener('resize')` used to detect an ELEMENT's size change (fails when the element resizes without the window doing so).

The principle: **each observer watches a different signal — intersection (visibility), resize (element size), mutation (DOM structure/attributes) — and choosing the right one matters.** Observers are asynchronous and batched; the classic ResizeObserver footgun is causing the very resize you are observing, creating a feedback loop.

</details>

---

## Module 4.9 — Browser Storage: localStorage, sessionStorage, Cookies, IndexedDB

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Web Storage API** — https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API — *Focus on: localStorage vs sessionStorage, that values are strings, synchronous access, and the storage event.*
- **MDN — Using IndexedDB** — https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB — *Focus on: object stores, transactions, and that it is asynchronous and stores structured data.*
- **MDN — HTTP cookies** — https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies — *Focus on: `HttpOnly`, `Secure`, `SameSite`, expiry, and that cookies travel with every request.*
- **web.dev — Storage for the web** — https://web.dev/articles/storage-for-the-web — *Focus on: choosing the right storage mechanism and quotas.*

**Search prompts:**

- `"localStorage vs sessionStorage vs cookies vs indexeddb"`
- `"localStorage json parse stringify gotcha"`
- `"cookie httponly secure samesite"`
- Ask an AI: *"Compare localStorage, sessionStorage, cookies, and IndexedDB on: persistence, capacity, sync vs async, what data types they hold, and whether they travel to the server. For each, give one correct use case and one misuse. Why must you JSON.stringify/parse for Web Storage?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.9 — browser storage mechanisms. Completed
so far: Tracks 1–3, Modules 4.1–4.8.

Build me an app that persists data using the WRONG storage mechanisms and misuses the right
ones: an object stored in localStorage without serialization (so it becomes "[object
Object]"); large/structured data crammed into localStorage that should be in IndexedDB; data
that should survive only a session put in localStorage (or vice versa); something sensitive
put in localStorage that should be an HttpOnly cookie; a synchronous localStorage access in a
hot path; a JSON.parse with no error handling that throws on corrupt data.

Tell me only how to run it (and how to inspect storage in DevTools → Application). I diagnose
each: fix serialization, move data to the right store, correct persistence scope, and handle
parse errors. Make me explain the tradeoffs of each storage mechanism and why Web Storage
only holds strings.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Storing an object directly in localStorage (`setItem('user', userObj)`) → it stringifies to `"[object Object]"`. Fix: `JSON.stringify` on write, `JSON.parse` on read.
- A `JSON.parse(localStorage.getItem(...))` with no try/catch that throws when the stored value is missing/corrupt.
- Large or structured/queryable data in localStorage (5MB cap, synchronous, string-only) that belongs in IndexedDB (async, large, structured).
- Session-scoped data in localStorage (persists forever) or persistent data in sessionStorage (lost on tab close) — wrong persistence scope.
- A token/secret in localStorage (readable by any JS, XSS-exposed) that should be an `HttpOnly; Secure; SameSite` cookie.
- Synchronous localStorage reads/writes in a performance-sensitive loop (it blocks the main thread).

The principle: **each storage mechanism has a distinct contract — capacity, sync/async, data shape, persistence scope, and whether it reaches the server.** Web Storage is synchronous string KV; IndexedDB is async structured storage; cookies travel with requests and carry security flags. Most storage bugs are choosing the wrong mechanism or forgetting that Web Storage only holds strings.

</details>

---

## Module 4.10 — Fetch, Streams & The Network

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Using the Fetch API** — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch — *Focus on: that `fetch` only rejects on network failure (NOT on 4xx/5xx — you must check `response.ok`), reading the body, and request options.*
- **MDN — AbortController / AbortSignal** — https://developer.mozilla.org/en-US/docs/Web/API/AbortController — *Focus on: cancelling in-flight requests and timeouts.*
- **MDN — Response** — https://developer.mozilla.org/en-US/docs/Web/API/Response — *Focus on: `json()`/`text()`/`blob()` consuming the body once, and the ReadableStream body.*
- **MDN — Using readable streams** — https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Using_readable_streams — *Focus on: processing a response progressively.*

**Search prompts:**

- `"fetch does not reject on 404 response.ok"`
- `"abortcontroller fetch timeout cancel"`
- `"fetch body already read stream"`
- Ask an AI: *"What are the surprising parts of `fetch`? Why does it resolve (not reject) on a 404 or 500, and what must I check? Why can I only read the response body once? How do I cancel a fetch with AbortController and implement a timeout? How do I stream a large response progressively?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.10 — fetch, streams, and the network.
Completed so far: Tracks 1–3, Modules 4.1–4.9.

Build me a small app that talks to a (fake or public) API and has realistic fetch bugs: code
that treats any resolved fetch as success and so silently ignores 404/500 responses; a body
read twice (throws "body already read"); a request that cannot be cancelled, so a stale
response overwrites a newer one (race); no timeout, so a hung request hangs the UI forever;
and JSON parsing that assumes the response is always JSON.

Tell me only how to run it (make some endpoints return errors and one be slow). I diagnose and
fix: check `response.ok`, read the body once, add AbortController for cancellation + timeout,
and handle non-OK/non-JSON responses. Make me explain why fetch does not reject on HTTP errors
and why the body is a one-shot stream.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Treating a resolved `fetch` as success without checking `response.ok`, so 404/500 responses are processed as if they were data. Fix: `if (!response.ok) throw ...`.
- Reading the body twice (e.g. `response.json()` after `response.text()`), throwing because the body stream is already consumed.
- A search/autocomplete-style race: a slower earlier request resolves after a faster later one and overwrites fresh results. Fix: `AbortController` to cancel the previous request when a new one starts.
- No timeout: a hung request leaves the UI in a loading state forever. Fix: abort via `AbortSignal.timeout()` or a manual timer.
- `response.json()` on a response that might be empty/HTML/error text, throwing on parse. Fix: branch on status/content-type.

The principle: **`fetch` only rejects on network-level failure, so HTTP error statuses are successful promises you must inspect via `response.ok`; the body is a single-use stream; and without AbortController you cannot cancel, time out, or prevent stale responses from racing.** Robust networking is mostly about handling the cases the happy path ignores.

</details>

---

## Module 4.11 — Web Workers: Off The Main Thread

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Using Web Workers** — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers — *Focus on: the worker model, `postMessage`, the message event, and that workers have no DOM access.*
- **MDN — The structured clone algorithm** — https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm — *Focus on: how data is COPIED to a worker, and Transferable objects to move (not copy) large buffers.*
- **MDN — Worker** — https://developer.mozilla.org/en-US/docs/Web/API/Worker — *Focus on: module workers and error handling.*

**Search prompts:**

- `"web worker postMessage structured clone"`
- `"transferable objects arraybuffer worker"`
- `"web worker no dom access main thread offload"`
- Ask an AI: *"Explain Web Workers: why does moving CPU-bound work to a worker keep the UI responsive when chunking on the main thread does not? How does `postMessage` move data (structured clone vs Transferable), and what are the constraints (no DOM, message-passing only)?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.11 — Web Workers, build challenge.
Completed so far: Tracks 1–3, Modules 4.1–4.10. Raise the difficulty.

Take the heavy main-thread computation idea from Module 4.5 and give me a spec to move it OFF
the main thread into a Web Worker: the UI (a spinner, a typed input, a button) must stay
perfectly responsive while a genuinely heavy computation runs in the worker and posts its
result back. Include passing a large dataset to the worker and getting a large result back.

Give me no solution code. Review me strictly. If I try to touch the DOM from the worker, do
not just tell me it is forbidden — ask me what a worker can and cannot access and why. If I
copy a huge buffer where I could transfer it, ask me about structured clone vs Transferables.
Make me explain why a worker keeps the UI responsive when main-thread chunking only mitigates.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A worker script that receives the dataset via `onmessage`, does the heavy computation, and `postMessage`s the result back; the main thread stays free to render and handle input the entire time.
- Correct message-passing: data is COPIED via structured clone (so the worker has its own copy); for large `ArrayBuffer`s, using Transferables to MOVE ownership (zero-copy) instead of cloning.
- Understanding the constraints: no DOM, no `window`, communication only via messages; the worker is a separate global scope.
- Common mistakes: trying to access `document`/DOM in the worker; assuming shared memory (it is message-passing, not shared state, unless using SharedArrayBuffer with care); cloning a giant buffer when a transfer was appropriate; not handling worker errors.

The principle: **a Web Worker runs on its own thread, so CPU-bound work there never blocks the main thread's rendering and input — unlike chunking, which only interleaves work on the one main thread.** The cost is the message-passing boundary (copy or transfer) and no DOM access. Workers are how you do real computation without freezing the UI.

</details>

---

## Module 4.12 — Core Web Vitals: Measuring What Users Feel

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Vanilla / any

### Prep — Read Before You Start

**Primary sources:**

- **web.dev — Web Vitals** — https://web.dev/articles/vitals — *Focus on: LCP, INP, CLS — what each measures and the "good" thresholds.*
- **web.dev — Largest Contentful Paint** — https://web.dev/articles/lcp — *Focus on: what element counts as LCP and the four sub-parts of LCP time.*
- **web.dev — Interaction to Next Paint** — https://web.dev/articles/inp — *Focus on: why INP replaced FID and what makes an interaction slow.*
- **web.dev — Cumulative Layout Shift** — https://web.dev/articles/cls — *Focus on: what causes layout shifts and how to reserve space.*

**Search prompts:**

- `"core web vitals lcp inp cls thresholds"`
- `"cumulative layout shift causes fix"`
- `"largest contentful paint optimization"`
- Ask an AI: *"Explain LCP, INP, and CLS: what each measures, the good/needs-improvement/poor thresholds, and the top causes of a bad score for each. Give me a diagnostic checklist for improving each metric."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.12 — Core Web Vitals, investigation.
Completed so far: Tracks 1–3, Modules 4.1–4.11.

Build me a page that scores BADLY on all three Core Web Vitals in realistic ways: a slow LCP
(the main hero image/text appears late due to loading), poor INP (a click handler does heavy
synchronous work so the next paint after interaction is delayed), and visible CLS (content
jumps as images/ads/fonts load without reserved space).

Tell me how to run it and how to measure with Lighthouse and the Performance panel's Web
Vitals overlay (with throttling). Do not tell me the causes. I will measure each metric, trace
each to its root cause, and fix it (optimize the LCP resource, break up or defer the
interaction work, reserve space to stop shifts), then re-measure. Make me explain what each
metric captures about the user's actual experience.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- **LCP:** the largest element (hero image or headline) renders late — e.g. an unoptimized/eagerly-competing image, render-blocking resources delaying it, or the image not prioritized. Fix: optimize/preload the LCP image, remove render-blocking, set `fetchpriority="high"`.
- **INP:** a click handler runs a heavy synchronous task, so the paint after the interaction is delayed past the threshold. Fix: break up/defer the work (yield, or move to a worker), update UI optimistically.
- **CLS:** images/iframes/ads without width/height (or `aspect-ratio`) and late-loading fonts cause content to jump. Fix: reserve space with dimensions/`aspect-ratio`, use `font-display` carefully, avoid inserting content above existing content.
- Lighthouse and the Web Vitals overlay quantify each before/after.

The principle: **Core Web Vitals encode what users actually feel — how fast the main content appears (LCP), how responsive interactions are (INP), and how visually stable the page is (CLS).** Each has concrete, diagnosable causes; improving them is improving the felt experience, not gaming a score.

</details>

---

## Module 4.13 — Hunting Memory Leaks In A Real App

**Difficulty:** ●●●●○ · **Format:** Investigation · **Stack:** Vanilla JS (browser)

### Prep — Read Before You Start

**Primary sources:**

- **Chrome DevTools — Fix memory problems** — https://developer.chrome.com/docs/devtools/memory-problems — *Focus on: the three-snapshot technique, the allocation timeline, and reading retained size.*
- **MDN — Memory management (revisit)** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management — *Focus on: reachability as the leak criterion.*
- **Chrome DevTools — Memory panel reference** — https://developer.chrome.com/docs/devtools/memory — *Focus on: the Memory panel tools and detached-node detection.*

**Search prompts:**

- `"chrome devtools three snapshot memory leak technique"`
- `"detached dom node memory leak find"`
- Ask an AI: *"Teach me the three-snapshot technique for finding memory leaks in Chrome DevTools. How do I use 'objects allocated between snapshots' and the retained-size column to find what is leaking, and how do I identify a detached DOM tree being held by JavaScript?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.13 — hunting memory leaks, investigation.
Completed so far: Tracks 1–3, Modules 4.1–4.12. Raise the difficulty.

Build me an interactive page where I repeatedly mount and unmount a "view" (via a button or
route toggle). Plant realistic leaks so that repeated mount/unmount cycles grow memory that is
never reclaimed: listeners/intervals not cleaned up, detached DOM nodes held by JS references,
a cache that grows unbounded, closures retaining large data.

Tell me how to run it and walk me through taking THREE heap snapshots with many mount/unmount
cycles between them. Do not tell me where the leaks are. I will use the snapshot diff,
retained size, and the detached-nodes view to find each leak, then fix it and prove (with new
snapshots) that memory stabilizes. Make me explain what kept each object reachable and how my
fix made it collectable.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A listener or `setInterval` registered on mount but never removed on unmount, so the handler and everything its closure captures stay reachable across cycles.
- A detached DOM subtree: removed from the document but still referenced by a JS array/map/closure (shows as "Detached" nodes in the snapshot), keeping the whole subtree alive.
- An unbounded cache (`Map`/object) accumulating an entry per cycle. Fix: bound it (LRU) or use a `WeakMap`/`WeakRef` so entries die with their keys.
- A closure capturing a large dataset that outlives its need.
- Snapshots: retained size climbs monotonically across cycles before fixes; flat after.

The principle: **an object survives GC only while it is reachable from a root, so every leak is an accidental reference you forgot to release — a listener, a timer, a detached node, or an ever-growing cache.** The three-snapshot technique makes the growth visible and points you straight at what is still holding on.

</details>

---

## Module 4.14 — Service Workers & The Cache API

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Service Worker API** — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API — *Focus on: the lifecycle (install → activate → fetch), that it is a network proxy, and that it runs separately from the page.*
- **MDN — Using Service Workers** — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers — *Focus on: registration, the install/activate events, and intercepting `fetch`.*
- **MDN — Cache API** — https://developer.mozilla.org/en-US/docs/Web/API/Cache — *Focus on: storing request/response pairs and cache-first vs network-first strategies.*
- **web.dev — The Service Worker lifecycle** — https://web.dev/articles/service-worker-lifecycle — *Focus on: the update model, `skipWaiting`, and the "waiting" state that confuses everyone.*

**Search prompts:**

- `"service worker lifecycle install activate fetch"`
- `"cache first vs network first strategy"`
- `"service worker not updating skipWaiting"`
- Ask an AI: *"Explain the Service Worker lifecycle (install, activate, fetch) and why a service worker behaves like a network proxy. Walk me through cache-first vs network-first vs stale-while-revalidate strategies. Why does a new service worker get 'stuck waiting,' and what does `skipWaiting` do?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.14 — service workers and the Cache API,
build challenge. Completed so far: Tracks 1–3, Modules 4.1–4.13. Raise the difficulty.

Give me a spec to make a small static site work OFFLINE using a service worker: register it,
precache the app shell on install, and serve cached assets when the network is unavailable,
with a sensible strategy for HTML vs static assets vs API calls. Then have me deliberately ship
an "update" and observe the lifecycle. Give me no solution code.

Review me strictly. If my service worker caches everything forever (so updates never reach
users) or my fetch handler responds incorrectly when offline, do not fix it — ask me which
lifecycle phase my caching happens in, what happens to an old cache on activate, and why a new
worker is "waiting." Make me explain install/activate/fetch and the strategy I chose for each
resource type, and verify offline behavior in DevTools (Application → Service Workers, and
the offline checkbox).
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Registration from the page; an `install` handler that `caches.open(...)` and `addAll`s the app shell; an `activate` handler that cleans up OLD cache versions; a `fetch` handler that implements a strategy (cache-first for static assets, network-first or stale-while-revalidate for HTML/API).
- Versioned cache names so a new deploy can purge the old cache on activate.
- Understanding the lifecycle trap: a new service worker installs but stays in "waiting" until all old-version tabs close (or you call `skipWaiting()` + `clients.claim()`), which is why "my update isn't showing."
- Offline verification: DevTools → Application → Service Workers, tick "Offline," reload, and confirm cached responses serve.
- Common mistakes: caching everything indefinitely so updates never propagate; not cleaning old caches on activate; a fetch handler that breaks (returns nothing) on cache miss while offline; expecting the new SW to take over immediately without skipWaiting/claim.

The principle: **a service worker is a programmable network proxy with its own lifecycle, and the Cache API lets it serve responses with or without the network.** Offline-capable apps live or die on the install/activate/fetch model and on cache versioning — the update "waiting" behavior is the lifecycle protecting in-flight pages, not a bug.

</details>

---

## Module 4.15 — The Network Waterfall & Resource Loading

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Resource timing / Network monitor** — https://developer.mozilla.org/en-US/docs/Web/API/Performance_API/Resource_timing — *Focus on: the phases of a resource load (DNS, connect, TLS, TTFB, download).*
- **web.dev — Preload, prefetch, preconnect** — https://web.dev/articles/preload-critical-assets — *Focus on: `preload` for critical resources, `preconnect` for third-party origins, `prefetch` for next-navigation needs.*
- **web.dev — `fetchpriority`** — https://web.dev/articles/fetch-priority — *Focus on: hinting resource priority to the browser.*
- **Chrome DevTools — Network features reference** — https://developer.chrome.com/docs/devtools/network — *Focus on: reading the waterfall, the Timing tab, and priority column.*

**Search prompts:**

- `"network waterfall request chains critical resources"`
- `"preload vs prefetch vs preconnect difference"`
- `"resource loading priority browser fetchpriority"`
- Ask an AI: *"Teach me to read a network waterfall: what each phase of a request bar means (queueing, stalled, DNS, connect, SSL, TTFB, content download), and what a 'request chain' is. Then explain when to use preload, prefetch, preconnect, and fetchpriority, with a concrete example of each."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 4.15 — the network waterfall, investigation.
Completed so far: Tracks 1–3, Modules 4.1–4.14. This closes the browser track.

Build me a page with a poor resource-loading profile: critical resources discovered late
(request chains where the browser cannot find the important file until something else loads
first), a third-party origin (font/CDN) connected to late, a critical hero image loaded at low
priority, and resources downloaded that are not needed for the initial view. The page is small
but its loading is badly sequenced.

Tell me how to run it and how to read the Network waterfall with throttling (Timing tab,
priority column, the request-chain shape). Do not tell me the problems. I will read the
waterfall, identify the late-discovered and mis-prioritized resources, and fix the sequencing
with preload/preconnect/prefetch/fetchpriority, then compare waterfalls. Make me explain each
request phase and what a request chain costs.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A request CHAIN where a critical resource is only discoverable after another resource downloads and parses (e.g. CSS that references a font, or JS that triggers a fetch), so it starts far too late. Fix: `preload` the critical resource so the browser fetches it immediately.
- A third-party origin (fonts/CDN/API) where the DNS+TCP+TLS handshake happens late, delaying the first request to it. Fix: `preconnect` (or `dns-prefetch`) to warm the connection early.
- The LCP hero image loaded at default/low priority while less important resources compete. Fix: `fetchpriority="high"` (and `preload`).
- Resources for later navigations or below-the-fold content downloaded eagerly, competing with critical ones. Fix: defer/lazy-load, or `prefetch` only when appropriate.
- Waterfall before/after shows the critical path shortening.

The principle: **the browser can only fetch what it has discovered, so deep request chains and late-connected origins delay critical resources — and resource hints let you tell the browser what to fetch, connect to, or prioritize early.** Reading the waterfall turns loading performance from guesswork into a visible dependency graph you can reshape.

</details>

---

*End of Track 4 — The Browser as a Runtime. 15 modules.*

*Next: Track 5 — React & Next.js (the largest track), in its own file.*
