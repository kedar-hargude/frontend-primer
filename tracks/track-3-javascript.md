# TRACK 3 — JAVASCRIPT & THE LANGUAGE

*The hard parts of the language itself, learned by breaking them. This is the track that makes you AI-proof: when you understand closures, the prototype chain, the event loop, and how `this` is bound, you can reason about any JavaScript behavior from first principles instead of pattern-matching. Pure vanilla — no framework hides the mechanics here.*

---

## Module 3.1 — Types, Coercion & Equality

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — JavaScript data types and data structures** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures — *Focus on: the seven primitives vs objects, and `typeof`'s quirks (including `typeof null`).*
- **MDN — Equality comparisons** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness — *Focus on: `==` vs `===` vs `Object.is`, and the abstract vs strict comparison algorithms.*
- **MDN — Type coercion** — https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion — *Focus on: how `+`, `==`, and template literals coerce, and the ToPrimitive/ToNumber/ToString steps.*
- **ECMAScript spec — Abstract Equality Comparison** — https://tc39.es/ecma262/#sec-islooselyequal — *Skim the actual algorithm `==` runs.*

**Search prompts:**

- `"javascript == vs === coercion rules"`
- `"why typeof null is object"`
- `"javascript truthy falsy values list"`
- Ask an AI: *"Walk me through the abstract equality (`==`) algorithm step by step. Then quiz me on tricky comparisons: `0 == ''`, `null == undefined`, `[] == ![]`, `NaN === NaN`. For each, explain the coercion that happens."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.1 — types, coercion, and equality.
Completed so far: Tracks 1–2. For JS, treat me as a working developer with gaps in the
language fundamentals.

Build me a single-file JS program (HTML page that logs to the screen, or a Node script) — a
small "data processing" routine: parsing user inputs, summing values, comparing things,
building messages. Plant realistic bugs caused by type coercion and equality confusion:
string/number addition going wrong, a `==` comparison passing when it should not (or vice
versa), a falsy check that rejects a valid `0` or empty string, `NaN` propagating silently,
and an object/array compared by value when it is actually compared by reference.

Tell me only how to run it. The outputs will be wrong in subtle ways. I will trace each wrong
value to the coercion or comparison that caused it and fix it. Make me explain, for each, what
JavaScript actually did under the hood — which conversion ran and why.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- `"5" + 3` producing `"53"` (string concatenation) where numeric addition was intended — a value came from an input/dataset as a string.
- A `==` comparison that coerces unexpectedly (e.g. `if (value == false)` matching `0`, `""`, `[]`).
- A truthiness guard (`if (count)`) that wrongly rejects a legitimate `0` or `""`.
- `NaN` produced by a bad parse and then silently flowing through arithmetic; `NaN === NaN` being false breaks a check.
- Comparing two objects/arrays with `===` expecting value equality, but getting reference inequality.
- `parseInt` without a radix, or `Number()` vs `parseInt` confusion on inputs like `"08"` or `"12px"`.

The principle: **JavaScript silently converts types in arithmetic and loose comparisons, following precise (if surprising) algorithms.** `===` avoids coercion; `==` invokes the abstract equality steps; objects compare by reference, never by value. Most "the number is wrong" bugs are a string that was never converted, and most "the comparison is wrong" bugs are coercion doing exactly what the spec says.

</details>

---

## Module 3.2 — Scope, Hoisting & The Temporal Dead Zone

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Scope** — https://developer.mozilla.org/en-US/docs/Glossary/Scope — *Focus on: global, function, and block scope.*
- **MDN — Hoisting** — https://developer.mozilla.org/en-US/docs/Glossary/Hoisting — *Focus on: how `var`, `let`, `const`, and function declarations are hoisted differently.*
- **MDN — `let` and the temporal dead zone** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz — *Focus on: why accessing a `let`/`const` before its declaration throws.*
- **MDN — `var`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var — *Focus on: function-scoping and why `var` in loops causes classic bugs.*

**Search prompts:**

- `"var let const hoisting difference temporal dead zone"`
- `"var in for loop closure bug"`
- Ask an AI: *"Explain hoisting precisely for `var`, `let`, `const`, function declarations, and function expressions. What is the temporal dead zone and why does it exist? Then show me the classic `var`-in-a-for-loop bug and explain why `let` fixes it."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.2 — scope, hoisting, and the TDZ.
Completed so far: Tracks 1–2, Module 3.1.

Build me a JS program with several functions and loops that misbehave due to scoping and
hoisting: a value that is `undefined` when the developer expected a number (hoisting), a
reference error that looks like it should not happen (TDZ), and the famous loop where every
callback logs the same final value because of `var` scoping. The program should run but
produce wrong/surprising output or a confusing error.

Tell me only how to run it. I will diagnose each issue by reasoning about which scope each
variable lives in and when it is initialized, then fix them. Make me explain what the engine
does during the "creation phase" of a scope and why `let` in a loop creates a fresh binding
per iteration.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A `var` read before its assignment returning `undefined` (declaration hoisted, initialization not) where the developer expected a value.
- A `let`/`const` accessed before declaration throwing a ReferenceError (TDZ) — surprising if the dev thinks all declarations hoist identically.
- The classic `for (var i...) { setTimeout(() => log(i)) }` printing the final `i` every time, because one function-scoped `i` is shared. Fix: `let i` (per-iteration binding) or an IIFE.
- A function expression assigned to a `var`/`const` called before the assignment line (only declarations hoist; the assignment does not).
- A variable accidentally global because it was assigned without declaration (or `var` leaking out of a block).

The principle: **declarations are processed before code runs, but `var` initializes to `undefined` while `let`/`const` stay uninitialized in a temporal dead zone until their line executes.** `var` is function-scoped; `let`/`const` are block-scoped, and `let` in a loop creates a new binding each iteration — which is exactly why closures over the loop variable behave differently.

</details>

---

## Module 3.3 — Closures

**Difficulty:** ●●●○○ · **Format:** Build challenge + broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Closures** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures — *Read the whole page. Focus on: lexical scoping, what a closure "captures," and the practical patterns (private state, factories, memoization).*
- **MDN — IIFE** — https://developer.mozilla.org/en-US/docs/Glossary/IIFE — *Focus on: the historical use of IIFEs to create scope before `let`.*

**Optional course resources:**

- FrontendMasters — *JavaScript: The Hard Parts* (Will Sentance) — the closures section is the best closure teaching anywhere.

**Search prompts:**

- `"javascript closure lexical scope captured variable"`
- `"closure private state counter factory"`
- Ask an AI: *"Explain closures from first principles: what does it mean that a function 'closes over' its lexical scope, and what exactly is captured — the value or the variable? Show me a counter factory, and explain why two counters made by the same factory do not share state."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.3 — closures. Completed so far:
Tracks 1–2, Modules 3.1–3.2.

Two parts.

Part 1 (build): give me a spec to implement, using closures, (a) a counter factory that
produces independent counters with private state, (b) a `once(fn)` that runs fn at most once
and caches the result, and (c) a simple `memoize(fn)`. No classes — closures only. Review me
strictly; make me explain what each returned function captured and why instances do not leak
into each other.

Part 2 (debug): then give me code where closures cause BUGS — event handlers in a loop all
referencing the same captured variable, a memoization cache that is accidentally shared or
never hit, a "private" variable that leaks. I diagnose and fix.

Do not reveal the bugs. Make me explain, in both parts, what variable each closure holds a
live reference to.
```

### 🔒 SPOILER — What This Module Tests / The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Part 1 (build) — a rigorous reviewer checks: each counter has its own `count` variable in its own closure (calling the factory twice gives independent state); `once` captures a `called` flag and a cached result; `memoize` captures a cache object keyed by arguments. The key understanding: a closure captures the VARIABLE (binding), not a snapshot of the value.

Part 2 (debug) — likely planted bugs:

- Loop handlers (with `var`) all closing over the same variable, so they all see the final value. Fix: `let` or an IIFE capturing the current value.
- A `memoize` whose cache is declared INSIDE the returned function (recreated every call, never hits) instead of in the closure.
- A "private" variable exposed by accidentally returning a reference to it (objects/arrays leak by reference).
- A closure holding a large object alive longer than expected (a memory-retention foreshadow).

The principle: **a closure is a function bundled with a live reference to the variables in the scope where it was defined.** It captures bindings, not values — which is the source of both its power (private, persistent state) and its classic bugs (everyone shares the one loop variable).

</details>

---

## Module 3.4 — `this`, call, apply, bind

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `this`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this — *Focus on: the four binding rules (default, implicit, explicit, `new`) and how arrow functions differ (lexical `this`).*
- **MDN — `Function.prototype.bind`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind — *Focus on: creating a bound function and partial application.*
- **MDN — `call` and `apply`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call — *Focus on: invoking with an explicit `this` and arguments.*

**Optional course resources:**

- FrontendMasters — *JavaScript: The Hard Parts* — the `this` and prototype sections.

**Search prompts:**

- `"javascript this binding rules implicit explicit new"`
- `"arrow function this lexical vs regular"`
- `"this undefined in callback method lost context"`
- Ask an AI: *"Give me the precedence order of `this` binding rules: default, implicit (method call), explicit (call/apply/bind), and `new`. Then explain why arrow functions ignore all of them. Quiz me with a method passed as a callback that loses its `this`."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.4 — `this` and explicit binding.
Completed so far: Tracks 1–2, Modules 3.1–3.3.

Build me a program with objects that have methods, callbacks, and event handlers where `this`
is WRONG in realistic ways: a method passed as a callback loses its object and `this` becomes
undefined/global; an arrow function is used where a dynamic `this` was needed (or vice versa);
a `setTimeout` callback's `this` is not what the developer expected; a borrowed method called
on the wrong object.

Tell me only how to run it. The symptoms are `undefined is not a function`, wrong values, or
operations hitting the wrong object. I will determine, for each call site, which binding rule
applied and fix it using `bind`/`call`/`apply` or an arrow function as appropriate. Make me
state the binding rule at each call site and why arrow functions break (or save) the pattern.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A method extracted and passed as a callback (`element.addEventListener('click', obj.method)`) so the implicit binding is lost and `this` is the element or `undefined`. Fix: `obj.method.bind(obj)` or an arrow wrapper.
- An arrow function used as an object METHOD, so `this` is the enclosing lexical scope (often the module/window), not the object.
- A regular `function` used as a callback where an arrow was needed to preserve the surrounding `this`.
- `setTimeout(obj.method, 1000)` losing context.
- A borrowed array-like method (`Array.prototype.slice.call(arguments)`) demonstrating explicit binding — or misusing `call`/`apply` argument forms.

The principle: **`this` is determined by HOW a function is called, not where it is defined — with four rules in precedence order (new > explicit > implicit > default).** Arrow functions opt out entirely and inherit `this` lexically. Lost-context bugs happen when a method is detached from its object; `bind`/`call`/`apply` or arrows reattach the intended `this`.

</details>

---

## Module 3.5 — Prototypes & The Prototype Chain

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Object prototypes** — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes — *Focus on: `[[Prototype]]`, `__proto__` vs `prototype`, and how property lookup walks the chain.*
- **MDN — Inheritance and the prototype chain** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain — *Read the whole thing. Focus on: how `new`, constructor functions, and `class` all build prototype links.*
- **MDN — `Object.create`, `Object.getPrototypeOf`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create — *Focus on: creating objects with an explicit prototype.*

**Optional course resources:**

- FrontendMasters — *JavaScript: The Hard Parts* — prototype chain section.

**Search prompts:**

- `"javascript prototype chain property lookup"`
- `"prototype vs __proto__ difference"`
- `"class syntactic sugar prototype"`
- Ask an AI: *"Explain the prototype chain: when I access `obj.foo`, how does the engine search for it? What is the difference between an object's `__proto__` and a function's `prototype` property? Show me how `class extends` desugars to prototype links."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.5 — prototypes and the prototype chain,
build challenge. Completed so far: Tracks 1–2, Modules 3.1–3.4. Raise the difficulty.

Give me a spec to build a small inheritance hierarchy TWICE: once with constructor functions
and manual prototype wiring (`Object.create`, setting `.prototype`, calling the parent
constructor), and once with `class`/`extends`. For example: a base `Shape` with an `area()`
method, and `Circle`/`Rectangle` subclasses. Then have me prove the two are equivalent by
inspecting the prototype chain of instances.

Give me no solution code. Review me strictly. If my constructor-function version has a broken
chain (methods not found, wrong constructor, parent not initialized), do not fix it — make me
walk the chain with `Object.getPrototypeOf` and find where the link is missing. Make me
explain how the `class` version produces the SAME chain.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Constructor-function version: methods on `Shape.prototype`; `Circle.prototype = Object.create(Shape.prototype)`; resetting `Circle.prototype.constructor = Circle`; calling `Shape.call(this, ...)` inside `Circle` to run the parent constructor; subclass methods overriding correctly.
- `class` version: `class Circle extends Shape { constructor(){ super(...) } }` producing the identical chain (`instance → Circle.prototype → Shape.prototype → Object.prototype → null`).
- Proving equivalence by walking `Object.getPrototypeOf` repeatedly and checking method resolution.
- Common mistakes: forgetting `Object.create` (using `new Shape()` as the prototype, which runs the constructor prematurely); not calling the parent constructor; forgetting to fix `.constructor`; thinking `prototype` and `__proto__` are the same thing.

The principle: **property lookup walks the prototype chain until it finds the property or hits `null`, and `class` is syntactic sugar over exactly this mechanism.** Constructor functions, `Object.create`, and `class extends` are three ways to wire the same `[[Prototype]]` links. Understanding the chain demystifies inheritance, method resolution, and "why is this method undefined."

</details>

---

## Module 3.6 — Objects, Property Descriptors, Getters & Setters

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Property descriptors / `Object.defineProperty`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty — *Focus on: `writable`, `enumerable`, `configurable`, and `value` vs `get`/`set`.*
- **MDN — Getters and setters** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get — *Focus on: accessor properties and computed values.*
- **MDN — `Object.freeze`, `Object.seal`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze — *Focus on: immutability levels and what each prevents.*

**Search prompts:**

- `"object defineproperty writable enumerable configurable"`
- `"javascript getter setter computed property"`
- `"object.freeze shallow vs deep"`
- Ask an AI: *"Explain the three attributes of a data property descriptor (`writable`, `enumerable`, `configurable`) and what each controls. Then explain accessor properties (get/set). Why does `Object.freeze` only freeze one level deep, and what does that mean for nested objects?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.6 — property descriptors, getters,
setters. Completed so far: Tracks 1–2, Modules 3.1–3.5.

Build me a program modeling some "configured" objects where behavior is surprising because of
property descriptors and accessors: a property that silently refuses to update (non-writable)
without throwing; a property that does not show up in `Object.keys`/`for...in` (non-
enumerable) so a loop misses it; an object that was "frozen" but whose nested data still
mutates (shallow freeze); a getter/setter pair where the setter has a bug so reads and writes
disagree.

Tell me only how to run it. The bugs manifest as data that won't change, loops that skip
fields, "immutable" data that mutates, and computed values that drift. I diagnose each via the
property descriptors (`Object.getOwnPropertyDescriptor`) and fix them. Make me explain what
each descriptor attribute controls and why shallow freeze is shallow.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A property defined with `writable: false`, so assignments silently do nothing in non-strict mode (or throw in strict) — the value "won't update."
- A property with `enumerable: false`, so `Object.keys`/`for...in`/spread skip it and a serialization or loop misses the field.
- An `Object.freeze`d object whose nested object/array still mutates because freeze is shallow.
- An accessor pair where the setter stores to the wrong backing field (or the getter reads a different one), so writes don't reflect in reads.
- A `configurable: false` property that cannot be redefined or deleted, surprising later code.

The principle: **every property has hidden attributes that govern whether it can change, be seen by iteration, or be reconfigured — and accessor properties replace a stored value with get/set functions.** "It won't update" / "the loop skips it" / "it's still mutable" bugs are these attributes doing exactly what they declare. `Object.freeze` only locks the top level because it does not recurse.

</details>

---

## Module 3.7 — Arrays & Iteration Methods

**Difficulty:** ●●○○○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Array** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array — *Focus on: `map`, `filter`, `reduce`, `find`, `some`, `every`, `flatMap`, and which methods mutate vs return new arrays.*
- **MDN — `Array.prototype.reduce`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce — *Focus on: the accumulator model and why `reduce` can express map/filter/group.*
- **MDN — Copying methods and mutating methods** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#copying_methods_and_mutating_methods — *Focus on: the mutation table (`sort`, `reverse`, `splice` mutate; `slice`, `toSorted`, `with` do not).*

**Search prompts:**

- `"javascript reduce explained accumulator"`
- `"array mutating vs non-mutating methods sort reverse"`
- Ask an AI: *"Implement `map`, `filter`, and `groupBy` using only `reduce`, and explain the accumulator pattern. Then list which array methods mutate the original array and which return a new one, and why mixing them up causes bugs."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.7 — arrays and iteration, build
challenge. Completed so far: Tracks 1–2, Modules 3.1–3.6.

Give me a spec for a data-transformation challenge over a dataset (say, a list of orders or
users): produce several derived results — a filtered+mapped list, a grouping, a sum/aggregate,
a "find first match", and a flattened structure — using array methods idiomatically and
WITHOUT mutating the source data. Then ask me to reimplement `map`, `filter`, and a `groupBy`
using only `reduce`.

Give me no solution code. Review me strictly. If I accidentally mutate the source (e.g.
`sort` in place), or reach for an imperative loop where a method reads cleaner, or my `reduce`
implementations are off, do not fix them — ask me whether that method mutates, and what my
accumulator is at each step. Make me explain reduce's signature and the mutation distinction.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Idiomatic, non-mutating transforms: `filter().map()`, `reduce` for grouping/aggregation, `find`/`some`/`every` for searches, `flatMap`/`flat` for flattening.
- Reimplementations: `map` as `reduce((acc, x) => [...acc, fn(x)], [])`; `filter` similarly with a conditional push; `groupBy` building an object keyed by a selector.
- The mutation trap: `sort()`/`reverse()`/`splice()` mutate the source — a clean answer copies first (`[...arr].sort()` or `toSorted()`).
- Common mistakes: mutating the source array and causing spooky action elsewhere; using `forEach` + external accumulator where `reduce`/`map` is clearer; `reduce` with no initial value misbehaving on empty arrays; off-by-one or wrong accumulator shape.

The principle: **`reduce` is the universal iteration primitive — map, filter, and group are all special cases of accumulation — and a sharp mental model of which methods mutate prevents a whole class of shared-state bugs.** Functional, non-mutating transforms keep data flow predictable.

</details>

---

## Module 3.8 — The Event Loop: Microtasks vs Macrotasks

**Difficulty:** ●●●●○ · **Format:** Broken project + investigation · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — The event loop** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop — *Focus on: the call stack, the task queue, run-to-completion, and the microtask checkpoint.*
- **MDN — Microtasks guide** — https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide — *Focus on: when microtasks (promises, `queueMicrotask`) run relative to macrotasks (`setTimeout`).*
- **Jake Archibald — Tasks, microtasks, queues and schedules** — https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/ — *The definitive write-up; work through the examples.*

**Optional course resources:**

- Jake Archibald — "In The Loop" (JSConf 2018, YouTube) — the canonical event-loop talk.

**Search prompts:**

- `"javascript microtask vs macrotask order"`
- `"promise then vs setTimeout execution order"`
- `"event loop run to completion call stack"`
- Ask an AI: *"Explain the event loop: call stack, macrotask queue, microtask queue, and the rule that all microtasks drain before the next macrotask. Then predict the log order of a program that mixes synchronous code, `setTimeout(0)`, `Promise.resolve().then`, and `queueMicrotask`, and explain each step."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.8 — the event loop, micro vs macrotasks.
Completed so far: Tracks 1–2, Modules 3.1–3.7. Raise the difficulty.

Build me a program whose console output order is SURPRISING and, in one place, actually buggy:
it mixes synchronous logs, `setTimeout(…, 0)`, resolved promises with `.then`,
`queueMicrotask`, and maybe an `await`. Somewhere, code assumes an ordering that does not hold
(e.g. it reads a value expecting a promise callback to have run already, but a setTimeout ran
first, or vice versa), producing a real bug.

First make me PREDICT the exact log order before running. Then I run it and reconcile. Then I
find and fix the ordering-assumption bug. Do not reveal the order or the bug. Make me explain
the event loop step by step — when the stack empties, when microtasks drain, when the next
macrotask runs — and why my prediction was wrong where it was.
```

### 🔒 SPOILER — The Planted Bugs / What It Tests

<details>
<summary>Click only after you finish or give up</summary>

- The core exercise: predicting that synchronous code runs first; then ALL microtasks (promise `.then`, `queueMicrotask`) drain; then ONE macrotask (`setTimeout`) runs; then microtasks drain again; etc. A nested `.then` queued during the microtask drain still runs before the next macrotask.
- The planted bug: code that depends on a `setTimeout(0)` callback having completed before a promise callback (or the reverse), so it reads stale/undefined state. The fix is to sequence the dependency correctly (await the right thing, or move logic into the correct queue).
- A common trap: assuming `setTimeout(0)` runs "immediately" or before microtasks.
- Possibly an `await` that yields to the microtask queue mid-function, reordering logs.

The principle: **JavaScript runs each task to completion, then drains the entire microtask queue, then renders/handles the next macrotask.** Promises and `queueMicrotask` are microtasks (high priority, drained fully); `setTimeout` is a macrotask (one per loop turn). Ordering bugs come from assuming an interleaving the loop does not actually produce.

</details>

---

## Module 3.9 — Promises From Scratch

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Using promises** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises — *Focus on: states (pending/fulfilled/rejected), chaining, and error propagation.*
- **MDN — Promise** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise — *Focus on: the executor, `then`/`catch`/`finally`, and the static combinators (`all`, `race`, `allSettled`, `any`).*
- **Promises/A+ specification** — https://promisesaplus.com — *Read the whole spec. It is short and it is exactly what you will implement.*

**Search prompts:**

- `"implement promise from scratch promises/a+"`
- `"promise then returns new promise chaining"`
- Ask an AI: *"Walk me through implementing a Promise that conforms to Promises/A+: the three states, the executor with resolve/reject, a `then` that returns a NEW promise, how resolution handles a thenable, and why callbacks must run asynchronously (as microtasks). Quiz me on the chaining behavior."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.9 — implement Promises from scratch,
build challenge. Completed so far: Tracks 1–2, Modules 3.1–3.8. This is a hard capstone for
the async portion — make it rigorous.

Give me a starter (one file) with a test harness: a set of assertions that exercise a
`MyPromise` — resolving, rejecting, async resolution, `then` chaining where each `then`
returns a new promise, returning a value vs returning a promise from `then`, error
propagation through `catch`, and `MyPromise.all`. The `MyPromise` class is an empty stub. The
tests should fail.

My job: implement `MyPromise` (executor, state machine, `then`/`catch`/`finally`, async
callback scheduling via microtask, thenable resolution) plus `all`. Give me no solution code.
Review me like a senior. When a chaining test fails, do not tell me the cause — ask me what
`then` must RETURN and when its callback is allowed to run. Make me explain why callbacks must
be asynchronous and how a returned promise gets adopted.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A state machine with `pending → fulfilled/rejected`, transitioning only once.
- An executor called immediately with `resolve`/`reject`.
- `then` returning a NEW promise so chains work; handlers stored if pending, scheduled if already settled.
- Callbacks scheduled asynchronously (via `queueMicrotask`/`setTimeout`) — never run synchronously, per spec.
- The resolution procedure: if a handler returns a thenable/promise, the next promise ADOPTS its state (chaining promises); if it returns a value, the next promise fulfills with it; if it throws, the next rejects.
- `catch` as `then(undefined, onRejected)`; `finally` running regardless and passing through.
- `MyPromise.all` resolving with an ordered results array and rejecting on the first rejection.
- Common mistakes: `then` not returning a new promise (chains break); running callbacks synchronously; not handling the returned-promise adoption; resolving more than once; losing error propagation.

The principle: **a promise is a state machine whose `then` returns a new promise, enabling chaining, with callbacks always deferred to the microtask queue.** Building one cements why async ordering works the way Module 3.8 showed, and why `then`-returning-a-promise is what makes `.then().then()` and `await` compose.

</details>

---

## Module 3.10 — async/await & Error Handling

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `async function`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function — *Focus on: how `async` wraps a return in a promise and how `await` pauses on a promise.*
- **MDN — `await`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await — *Focus on: that `await` yields to the microtask queue and unwraps the resolved value (or throws on rejection).*
- **MDN — `Promise.all`/`allSettled`/`race`/`any`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all — *Focus on: concurrent vs sequential awaiting.*

**Search prompts:**

- `"async await sequential vs parallel promise.all"`
- `"await in for loop performance sequential"`
- `"async await try catch error handling"`
- Ask an AI: *"Explain how `async`/`await` maps onto promises. Why does `await`ing inside a `for` loop run things sequentially, and how do I parallelize with `Promise.all`? How does error handling work — what does `await` do on a rejected promise, and what does an unhandled rejection look like?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.10 — async/await and error handling.
Completed so far: Tracks 1–2, Modules 3.1–3.9.

Build me a program that fetches/awaits several async operations (use a fake async API with
delays and occasional failures) and is buggy in realistic async ways: independent requests
awaited one-by-one in a loop so the whole thing is needlessly slow; an error that is swallowed
because of where (or whether) try/catch sits; an unhandled promise rejection; a function that
returns before its async work finishes because an `await` was forgotten; and a `Promise.all`
that fails the whole batch when one item could have been tolerated.

Tell me only how to run it (it should be observably slow and/or produce wrong results/console
errors). I diagnose and fix: parallelize the independent awaits, place error handling
correctly, and choose the right combinator. Make me explain why sequential awaits are slow and
what `await` does to control flow and errors.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Independent async calls `await`ed sequentially inside a `for...of`, making total time the SUM of latencies. Fix: kick them all off and `await Promise.all(...)` for the sum→max speedup.
- A `try/catch` that does not wrap the `await` (or an async error thrown outside any catch), so the error is swallowed or becomes an unhandled rejection.
- A missing `await` so a function resolves/returns before the async work completes, yielding `undefined`/a pending promise.
- A `Promise.all` where one rejection rejects the whole batch when `Promise.allSettled` was the right tool.
- A fire-and-forget promise with no `.catch`, producing an unhandled rejection warning.

The principle: **`await` pauses an async function until a promise settles, unwrapping the value or throwing on rejection — but awaiting independent operations in sequence forfeits concurrency.** Parallelize with `Promise.all`/`allSettled`, place `try/catch` around the actual `await`, and never forget the `await` that makes your function actually wait.

</details>

---

## Module 3.11 — Modules: ESM vs CommonJS

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS / Node

### Prep — Read Before You Start

**Primary sources:**

- **MDN — JavaScript modules** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules — *Focus on: `import`/`export` (named vs default), module scope, and that modules are deferred and run once.*
- **Node.js — Modules: ECMAScript modules** — https://nodejs.org/api/esm.html — *Focus on: ESM vs CommonJS interop, `.mjs`/`type: module`, and why you cannot mix them naively.*
- **MDN — `import` (static) and dynamic `import()`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import — *Focus on: static analysis (enabling tree-shaking) vs runtime dynamic import.*

**Search prompts:**

- `"esm vs commonjs import require difference"`
- `"named export vs default export"`
- `"circular dependency javascript modules"`
- Ask an AI: *"Explain the difference between ES modules (`import`/`export`) and CommonJS (`require`/`module.exports`): loading semantics (static vs dynamic), live bindings vs value copies, and why mixing them causes interop pain. What happens with a circular dependency in each system?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.11 — modules, ESM vs CommonJS. Completed
so far: Tracks 1–2, Modules 3.1–3.10.

Build me a small multi-file project that has realistic module bugs: a default import that
should be named (or vice versa) so a value is `undefined`; a circular dependency where one
module reads another's export before it is initialized; a mix of `require` and `import` that
breaks; and a shared module that everyone imports but that has surprising single-execution
behavior (a side effect that runs once, or a "singleton" that is accidentally not shared).

Tell me only how to run it (and which runtime/flags). I diagnose each: fix the import/export
mismatch, break or reorder the cycle, resolve the interop, and explain the single-evaluation
behavior. Make me explain how modules are resolved and executed, and the difference between
ESM live bindings and CommonJS value copies.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A `import thing from './x'` (default) where `x` only has a NAMED export (or the reverse), so `thing` is `undefined`.
- A circular import where module A imports B and B imports A, and one reads the other's export at module-evaluation time before it is assigned — getting `undefined`. Fix: defer the access, restructure, or use a function.
- Mixing `require()` and ESM `import` in a way the runtime rejects, or relying on CommonJS `__dirname` in ESM.
- A module with a top-level side effect (e.g. registering something) that the developer expects to run per-import but runs ONCE (modules are evaluated a single time and cached).
- A "singleton" duplicated because two different specifiers resolve to two module instances.

The principle: **modules are resolved, evaluated once, and cached; ESM uses static, hoisted imports with live bindings while CommonJS uses dynamic `require` returning a value copy.** Import/export mismatches, circular-dependency ordering, and interop friction all flow from these semantics. Static ESM is also what makes tree-shaking possible (foreshadowing Track 8).

</details>

---

## Module 3.12 — Generators & Iterators

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Iterators and generators** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators — *Focus on: the iterator protocol (`next()` returning `{value, done}`), the iterable protocol (`Symbol.iterator`), and `function*`/`yield`.*
- **MDN — `function*`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function* — *Focus on: lazy evaluation, two-way communication via `yield`, and `yield*` delegation.*
- **MDN — Symbol.iterator** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator — *Focus on: making a custom object iterable so `for...of` and spread work on it.*

**Search prompts:**

- `"javascript generator function yield lazy"`
- `"custom iterable symbol.iterator for...of"`
- Ask an AI: *"Explain the iterator and iterable protocols. Then show me how a generator (`function*`/`yield`) implements an iterator lazily, including infinite sequences. How does data flow BOTH directions through `yield` (passing a value into `.next()`)?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.12 — generators and iterators, build
challenge. Completed so far: Tracks 1–2, Modules 3.1–3.11.

Give me a spec to build: (1) a custom iterable object (e.g. a range or a linked list) that
works with `for...of` and spread by implementing `Symbol.iterator`; (2) a generator that
produces an INFINITE sequence (e.g. fibonacci) consumed lazily with a `take(n)` helper; and
(3) a generator that receives values back via `.next(value)` to demonstrate two-way
communication. Give me no solution code.

Review me strictly. If my custom iterable does not work with `for...of`, do not tell me why —
ask me what `Symbol.iterator` must return and what shape `next()` produces. If my infinite
generator hangs the page, ask me what "lazy" means and who controls advancing. Make me explain
the iterator protocol and how a generator pauses at `yield`.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A custom iterable implementing `[Symbol.iterator]()` returning an object with a `next()` that yields `{value, done}` — enabling `for...of`, spread, destructuring.
- An infinite generator (`function* fib()` with `while(true) yield`) that does not hang because the CONSUMER controls advancement; a `take(gen, n)` that pulls exactly n values.
- Two-way communication: `const x = yield value` receiving what the caller passes to `.next(x)`.
- `yield*` delegation to another iterable.
- Common mistakes: `Symbol.iterator` returning the wrong shape; forgetting `done: true` to terminate; trying to eagerly materialize an infinite sequence (hang); misunderstanding that the first `.next()` argument is ignored.

The principle: **the iterator protocol is a pull-based contract (`next()` → `{value, done}`), and generators implement it with pausable functions that yield lazily and can receive values back.** This is the machinery behind `for...of`, spread, and (conceptually) async iteration — and lazy evaluation lets you model infinite or expensive sequences safely.

</details>

---

## Module 3.13 — Proxies & Reflect

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Proxy** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy — *Focus on: the trap handlers (`get`, `set`, `has`, `deleteProperty`, etc.) and the target/handler model.*
- **MDN — Reflect** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect — *Focus on: why `Reflect` methods pair with proxy traps to perform the default operation correctly.*
- **MDN — Proxy `get`/`set` handlers** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get — *Focus on: the arguments each trap receives and what it must return.*

**Search prompts:**

- `"javascript proxy get set trap reactive"`
- `"reflect vs direct property access proxy"`
- Ask an AI: *"Explain JavaScript Proxies: what is a target, a handler, and a trap? Show me a Proxy that logs every property access and one that validates writes. Why do trap implementations call the corresponding `Reflect` method instead of doing the operation directly? (This is roughly how Vue's reactivity works.)"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.13 — Proxy and Reflect, build challenge.
Completed so far: Tracks 1–2, Modules 3.1–3.12.

Give me a spec to build: (1) a validation proxy that enforces rules on writes (e.g. an object
where `age` must be a positive integer, throwing on violation); (2) a logging/observation
proxy that records every get and set (the seed of a reactivity system); and (3) a "negative
array index" proxy where `arr[-1]` returns the last element. Use `Reflect` for the default
behavior inside traps. Give me no solution code.

Review me strictly. If my trap forgets to actually perform (or return) the underlying
operation, do not tell me — ask me what the `get`/`set` trap must RETURN and what happens if
it does not. Make me explain why `Reflect.get`/`Reflect.set` exist and why reimplementing the
default by hand inside a trap is error-prone.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A `set` trap that validates then calls `Reflect.set(target, prop, value, receiver)` and returns its boolean result (returning falsy/throwing on invalid).
- A `get`/`set` logging proxy that records access then delegates to `Reflect` — the conceptual basis of fine-grained reactivity (Vue 3, MobX, Valtio).
- A `get` trap handling negative indices (`arr[-1]` → `target[target.length - 1]`) and otherwise `Reflect.get`.
- Correct use of the `receiver` argument for proper `this`/prototype behavior.
- Common mistakes: not returning a value from `get`; not returning `true` from `set` (causes a TypeError in strict mode); reimplementing default behavior by hand and breaking getters/inheritance; infinite recursion by accessing the proxy inside its own trap instead of the target.

The principle: **a Proxy intercepts fundamental operations on an object via trap functions, and `Reflect` provides the exact default behavior for each trap so you can wrap rather than replace it.** This is the mechanism behind modern reactivity systems — observing reads to track dependencies and writes to trigger updates.

</details>

---

## Module 3.14 — Memory, References & Garbage Collection

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Vanilla JS (browser)

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Memory management** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management — *Focus on: reachability, the mark-and-sweep algorithm, and what keeps an object alive.*
- **MDN — WeakMap / WeakSet / WeakRef** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap — *Focus on: weak references that do not prevent collection, and their use for caches/metadata.*
- **Chrome DevTools — Memory: Heap snapshots** — https://developer.chrome.com/docs/devtools/memory-problems — *Focus on: taking heap snapshots, comparing them, and spotting detached DOM nodes and growing retained size.*

**Search prompts:**

- `"javascript memory leak closure detached dom"`
- `"weakmap vs map garbage collection"`
- `"chrome devtools heap snapshot detached nodes"`
- Ask an AI: *"Explain JavaScript garbage collection (reachability + mark-and-sweep). Then list the four classic memory-leak patterns in the browser: forgotten timers/listeners, detached DOM references, closures retaining large objects, and unbounded caches. How does a WeakMap avoid the cache leak?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.14 — memory, references, GC. Completed so
far: Tracks 1–2, Modules 3.1–3.13. Raise the difficulty.

Build me a small interactive page (mount/unmount a "widget" repeatedly via a button) that
LEAKS memory in realistic ways: an event listener or interval that is never removed; a
reference to a removed (detached) DOM node held in a JS array/closure; a cache that grows
without bound; a closure that retains a large object longer than needed.

Tell me how to run it and how to take and compare heap snapshots in DevTools (mount/unmount
many times between snapshots). Do not tell me where the leaks are. I will use heap snapshots
and the detached-nodes / retained-size views to FIND each leak, then fix it (cleanup
listeners/timers, drop references, bound or weaken the cache). Make me explain what kept each
object reachable and how the fix made it collectable.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- An `addEventListener` (or `setInterval`) added on mount and never removed on unmount, so the handler — and everything its closure references — stays reachable.
- A detached DOM node: the element is removed from the document but a JS reference (in an array, map, or closure) keeps it (and its subtree) alive — visible as "Detached" nodes in a heap snapshot.
- An unbounded cache (`Map`/object) that accumulates entries forever. Fix: bound it (LRU) or use a `WeakMap` keyed by objects so entries are collectable when keys die.
- A closure capturing a large object that outlives its usefulness.
- Snapshots show retained size growing across mount/unmount cycles; after fixes, it stabilizes.

The principle: **an object is garbage-collected only when it is unreachable from any root, so leaks are accidental reachability — a listener, a timer, a stray reference, or a cache that never lets go.** `WeakMap`/`WeakRef` hold references that do not prevent collection. Heap snapshots make the invisible visible: you can see exactly what is still pointing at memory you meant to free.

</details>

---

## Module 3.15 — Functional Patterns: Currying, Composition, Immutability

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Closures (revisit for partial application)** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures — *Focus on: how closures enable currying and partial application.*
- **MDN — Spread syntax & structuredClone** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax — *Focus on: non-mutating updates and the limits of shallow copies; `structuredClone` for deep copies.*
- **MDN — `Function.prototype.bind` (partial application)** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind — *Focus on: presetting arguments.*

**Search prompts:**

- `"javascript currying partial application explained"`
- `"function composition pipe compose"`
- `"immutable update nested object spread"`
- Ask an AI: *"Explain currying vs partial application with examples. Then implement `compose` and `pipe`. Then show me how to update a deeply nested object immutably with spread, and explain why shallow spread is not enough for nested data and when `structuredClone` helps."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.15 — functional patterns, build challenge.
Completed so far: Tracks 1–2, Modules 3.1–3.14.

Give me a spec to implement: (1) a `curry(fn)` that lets a multi-arg function be called one
argument at a time; (2) `compose(...fns)` and `pipe(...fns)`; and (3) a set of immutable
update helpers for nested state (update a deep field without mutating the original). Then a
small task that uses all three together to transform some data. Give me no solution code.

Review me strictly. If my `curry` mishandles arity, my `compose`/`pipe` apply functions in
the wrong order, or my "immutable" update accidentally mutates a nested object (shallow copy
trap), do not fix it — ask me how many arguments have arrived, which direction data flows
through the composition, and which levels of the object I actually copied. Make me explain
why shared references make shallow immutability a lie.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `curry`: collects args until arity is satisfied, then invokes — using closures to accumulate.
- `compose` (right-to-left) vs `pipe` (left-to-right): both reduce over the function list, threading the result; getting the direction right is the test.
- Immutable nested update: spreading at EVERY level along the path being changed (`{...state, user: {...state.user, name}}`), not just the top — otherwise nested objects are shared by reference and "immutable" updates mutate the original. `structuredClone` for deep copies when appropriate (and its limits with functions/DOM).
- Common mistakes: shallow spread that shares nested references (mutation leaks to the "old" state — directly relevant to React state bugs in Track 5); compose/pipe reversed; curry not handling extra/fewer args.

The principle: **closures make currying and composition natural, and true immutability requires copying every level along the path you change, because objects are shared by reference.** These patterns underpin predictable state management — the same shallow-copy trap that bites here is the root of countless React "my state didn't update" bugs.

</details>

---

## Module 3.16 — Regular Expressions

**Difficulty:** ●●●○○ · **Format:** Build challenge + broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Regular expressions guide** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions — *Focus on: character classes, quantifiers, anchors, groups, and flags (`g`, `i`, `m`, `s`, `u`).*
- **MDN — Groups and backreferences** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Groups_and_backreferences — *Focus on: capturing vs non-capturing groups, named groups, and lookahead/lookbehind.*
- **MDN — `RegExp.prototype.exec` / `String.matchAll`** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec — *Focus on: the stateful `lastIndex` with the `g` flag and why it bites.*

**Search prompts:**

- `"regex capturing vs non-capturing group"`
- `"regex global flag lastIndex stateful bug"`
- `"regex lookahead lookbehind explained"`
- Ask an AI: *"Teach me regex anchoring, quantifiers (greedy vs lazy), capturing/non-capturing/named groups, and lookahead/lookbehind. Then explain the stateful `lastIndex` gotcha with the `g` flag — why reusing a `/g` regex across calls can skip matches."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.16 — regular expressions. Completed so
far: Tracks 1–2, Modules 3.1–3.15.

Two parts.

Part 1 (build): give me a spec for several real parsing/validation tasks — extract all hashtags
from text, validate a specific ID format, parse key=value pairs, reformat dates — to implement
with regex. Review me strictly: make me explain greedy vs lazy quantifiers, why I anchored (or
should have), and capturing vs non-capturing groups.

Part 2 (debug): then give me code with regex BUGS — a validation regex that is unanchored and
accepts garbage; a greedy quantifier that over-matches; a `/g` regex reused across calls whose
`lastIndex` causes it to skip matches; a special character not escaped. I diagnose and fix.

Do not reveal the bugs. Make me explain what each pattern matches step by step and the
`lastIndex` statefulness.
```

### 🔒 SPOILER — The Planted Bugs / What It Tests

<details>
<summary>Click only after you finish or give up</summary>

- An unanchored validation regex (no `^`/`$`) that matches a valid substring inside invalid input, so garbage passes.
- A greedy quantifier (`.*`) that over-matches across delimiters where a lazy `.*?` or a negated class `[^"]*` was needed.
- A `/g` regex stored and reused with `.test()`/`.exec()` across calls: `lastIndex` advances and persists, so subsequent calls start mid-string and skip or miss matches. Fix: don't reuse a `/g` regex statefully (or reset `lastIndex`, or use `matchAll`).
- An unescaped special character (`.`/`(`/`[`) treated as a metacharacter when a literal was intended.
- A capturing group used where non-capturing `(?:...)` was meant (off-by-one in match indices).

The principle: **a regex is a tiny state machine; anchoring controls where it may match, quantifiers control how much, groups control what you capture, and the `g` flag makes the object stateful via `lastIndex`.** Most regex bugs are not "wrong pattern" but "right pattern, wrong assumptions" — unanchored, greedy, or statefully reused.

</details>

---

## Module 3.17 — Event Delegation & The DOM Event Model

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Introduction to events** — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events — *Focus on: the event object, `target` vs `currentTarget`, and listener basics.*
- **MDN — Event bubbling and capture** — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Event_bubbling — *Focus on: the three phases (capture, target, bubble), `stopPropagation`, and `preventDefault`.*
- **MDN — Event delegation** — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Event_bubbling#event_delegation — *Focus on: one listener on a parent handling events from many children, and why it survives dynamic content.*

**Search prompts:**

- `"event delegation javascript target currenttarget"`
- `"event bubbling capturing phases stopPropagation"`
- `"event listener not working on dynamically added elements"`
- Ask an AI: *"Explain the three phases of DOM event propagation (capture, target, bubble), and the difference between `event.target` and `event.currentTarget`. Then explain event delegation: why attaching one listener to a parent handles clicks on children — even children added later — and how `stopPropagation` and `preventDefault` differ."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 3.17 — event delegation and propagation.
Completed so far: Tracks 1–2, Modules 3.1–3.16. This closes the JS track.

Build me an interactive list/board UI (vanilla JS) with bugs rooted in the event model:
listeners attached to each item individually so dynamically-added items have no handler; a
handler that uses `event.target` where it needed `currentTarget` (or vice versa) and so acts
on the wrong element when an inner element is clicked; a `stopPropagation` that breaks an
ancestor behavior; a missing `preventDefault` on a link/form so the default action fires; and
a delegated handler that fails to identify which child was clicked.

Tell me only how to run it. I diagnose each via the event flow — which phase, which element is
`target` vs `currentTarget`, what propagation is happening — and fix it, converting to proper
delegation where appropriate. Make me explain bubbling and the target/currentTarget
distinction.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Per-item listeners that miss items added AFTER setup (no handler on new elements). Fix: delegate — one listener on the container, identify the item via `event.target.closest('.item')`.
- A handler reading `event.target` (the deepest clicked element, possibly an inner icon/span) when it needed `event.currentTarget` (the element the listener is bound to) — so it operates on the wrong node.
- A `stopPropagation()` that prevents a needed ancestor handler (e.g. an outside-click-to-close) from ever firing.
- A click handler on a link or a submit on a form missing `preventDefault()`, so the browser navigates/reloads.
- A delegated handler that does not narrow to the right child (clicks on the container background also trigger it) because it lacks a `closest`/`matches` guard.

The principle: **events propagate through capture → target → bubble, `target` is what was clicked while `currentTarget` is what is listening, and delegation leverages bubbling so one parent listener handles many (even future) children.** Most "the handler fired on the wrong thing" or "new items don't work" bugs are target/currentTarget confusion or the absence of delegation.

</details>

---

*End of Track 3 — JavaScript & the Language. 17 modules.*

*Next: Track 4 — The Browser as a Runtime, in its own file.*