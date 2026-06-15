# TRACK 8 — ENGINEERING & TOOLING

*The senior layer. By now you can build almost anything; this track is about the engineering practices that make you trusted with a codebase rather than just a feature: how your code becomes a bundle, how the type system actually protects you, what to test and how, and how to ship safely. You build a bundler, write a custom lint rule, design a type-safe component, and reason about security and performance budgets. This is the difference between "writes React" and "owns the frontend."*

*Reminder: your root `CLAUDE.md` is loaded automatically by Claude Code. Do the Prep first. Never read the spoiler before you finish or give up.*

---

## Module 8.1 — Modules & Bundling: How Code Becomes A Bundle

**Difficulty:** ●●●○○ · **Format:** Investigation · **Stack:** Node + Vite/esbuild

### Prep — Read Before You Start

**Primary sources:**
- **MDN — JavaScript modules (revisit)** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules — *Focus on: static import graphs and why bundlers can analyze them.*
- **Vite — Why Vite / Features** — https://vite.dev/guide/why — *Focus on: dev (native ESM, no bundle) vs build (Rollup bundle), and why the two differ.*
- **esbuild — Architecture** — https://esbuild.github.io/faq/#why-is-esbuild-fast — *Focus on: what a bundler actually does — resolve, load, parse, transform, link, emit.*
- **webpack — Concepts** — https://webpack.js.org/concepts/ — *Focus on: entry, output, loaders, plugins, the dependency graph.*

**Search prompts:**
- `"how does a javascript bundler work dependency graph"`
- `"vite dev server vs build rollup difference"`
- `"esbuild vs webpack vs rollup"`
- Ask an AI: *"Explain what a JavaScript bundler does step by step: starting from an entry file, how it resolves imports, builds a dependency graph, and emits one or more bundles. Why does Vite serve unbundled native ESM in dev but bundle with Rollup for production? What is a 'module graph'?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.1 — modules and bundling, investigation.
Completed so far: Tracks 1–7.

Scaffold me a small multi-module project (a handful of ES modules importing each other, plus one
dynamic import and one third-party dependency) built with Vite. My job is to UNDERSTAND, not just
run it: inspect the dev behavior (native ESM, separate requests) vs the production build
(bundled output in dist), read the generated dependency graph / build output, and identify how my
source modules map to emitted chunks.

Tell me only how to run dev and build and where to look (network tab in dev, the dist folder and
build stats for prod). Do not explain the output for me. Make me trace: which file is the entry,
how an import becomes part of a chunk, what the dynamic import produced (a separate chunk), and how
the third-party dep was handled. Quiz me on why dev and prod load so differently and what a module
graph is.
```

### 🔒 SPOILER — What You Should Discover

<details>
<summary>Click only after you finish or give up</summary>

- In dev, Vite serves your source modules as native ESM over HTTP (you'll see separate requests per module in the Network tab), transforming on demand — no bundling, which is why dev start is instant.
- In the production build, Rollup walks the import graph from the entry, bundles modules into a small number of chunks, and emits hashed files into `dist` — you can map source modules to output chunks via the build output / a visualizer.
- The dynamic `import()` becomes its OWN chunk (a split point), loaded on demand — foreshadowing code-splitting (8.4).
- The third-party dependency is resolved from `node_modules`, often pre-bundled in dev and included/split in prod.
- The "module graph" is the directed graph of import relationships the bundler builds and traverses.

The principle: **a bundler starts at an entry, resolves and parses every import into a module graph, then links and emits that graph as chunks — and modern tools split dev (unbundled native ESM for speed) from prod (bundled for delivery).** Understanding the graph is the foundation for tree-shaking, code-splitting, and bundle analysis in the next modules.
</details>

---

## Module 8.2 — Build Your Own Bundler

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** Node

### Prep — Read Before You Start

**Primary sources:**
- **Ronen Amiel — Build Your Own Webpack (talk + repo)** — https://github.com/ronami/minipack — *Read minipack's annotated source (~200 lines). This is essentially what you build.*
- **MDN — `import`/`export` static structure** — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import — *Focus on: why imports are statically analyzable.*
- **Babel — @babel/parser & traverse** — https://babeljs.io/docs/babel-parser — *Focus on: parsing source to an AST and walking it to find imports.*
- **Node — `module` / resolution** — https://nodejs.org/api/modules.html — *Focus on: how paths resolve.*

**Search prompts:**
- `"minipack build your own bundler tutorial"`
- `"babel parser traverse find import declarations"`
- `"module dependency graph topological bundle"`
- Ask an AI: *"Walk me through building a minimal JS bundler: read the entry file, parse it to an AST (Babel), find its import declarations to discover dependencies, recursively build a dependency graph with unique ids, transform each module to CommonJS, then emit a single bundle with a small module runtime/`require` shim that wires modules together by id. Explain each stage's job."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.2 — build your own bundler, build challenge.
Completed so far: Tracks 1–7, Module 8.1. This is a hard capstone — make it rigorous.

Give me a starter: a tiny source app (entry module importing 2–3 other ES modules that import each
other), and a `bundler.js` with empty stubs: `createAsset(filename)` (parse a file, extract its
dependencies, transform code), `createGraph(entry)` (recursively build the dependency array), and
`bundle(graph)` (emit a single runnable bundle string). Babel parser/traverse/core are available.
My job is to implement all three so the emitted bundle runs in Node and produces the app's output.
Give me no solution code.

Review me like a senior. Before coding, make me explain why imports can be found statically and
what a module "id" is for. For `createAsset`, make me say what the AST gives me. For `bundle`, make
me explain the `require` shim and the module map — how does module A get module B by id at runtime?
When the bundle errors, ask me whether the bug is in graph-building or in the runtime wiring. After
it works, make me compare my bundler to what Rollup/esbuild add (tree-shaking, scope hoisting,
chunks).
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `createAsset`: read the file, parse to an AST, traverse to collect `ImportDeclaration` source values (the dependencies), transform the module code (e.g. to CommonJS via Babel), and return an asset `{ id, filename, dependencies, code }` with a unique incrementing id.
- `createGraph`: start at the entry asset; for each asset, resolve each dependency path to an absolute path, recurse, and record a `mapping` from the import string → the dependency's id. Build the full array of assets.
- `bundle`: emit a string containing a `modules` object keyed by id, each entry `[ function(require, module, exports){ <code> }, mapping ]`; plus a small runtime that defines `require(id)` to instantiate a module, wiring `require('./x')` to `require(mapping['./x'])`; finally `require(0)` to run the entry.
- Understanding: module ids decouple runtime wiring from file paths; the mapping translates import specifiers to ids; the runtime is a tiny module system.
- Common mistakes: not resolving relative paths to absolute (duplicate/missed modules); forgetting the per-module mapping so `require` can't find deps; not transforming ESM→CJS so `require`/`exports` don't exist; infinite recursion on circular deps without a visited guard.
- Comparison: real bundlers add tree-shaking (drop unused exports), scope hoisting, multiple chunks, code-splitting, and source maps.

The principle: **a bundler is parse → find imports → build a graph with ids → emit a module map plus a tiny `require` runtime that wires modules together.** Building one removes all the mystery from webpack/Rollup/Vite: everything they do is an optimization or feature layered on this core graph-and-runtime idea.
</details>

---

## Module 8.3 — Tree Shaking & Dead Code Elimination

**Difficulty:** ●●●●○ · **Format:** Investigation + broken project · **Stack:** Vite/Rollup

### Prep — Read Before You Start

**Primary sources:**
- **MDN — Tree shaking** — https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking — *Focus on: why static ESM enables removing unused exports.*
- **webpack — Tree Shaking** — https://webpack.js.org/guides/tree-shaking/ — *Focus on: `sideEffects: false`, and why side effects block elimination.*
- **Rollup — Tree-shaking** — https://rollupjs.org/introduction/#tree-shaking — *Focus on: how Rollup determines a binding is unused.*
- **web.dev — Reduce JavaScript payloads with tree shaking** — https://web.dev/articles/reduce-javascript-payloads-with-tree-shaking — *Focus on: import styles that defeat it.*

**Search prompts:**
- `"tree shaking sideEffects false package.json"`
- `"barrel file import defeats tree shaking"`
- `"why is my unused code in the bundle"`
- Ask an AI: *"Explain tree shaking: why static ES module imports let a bundler statically prove an export is unused and drop it, and why CommonJS can't be shaken. What defeats tree shaking — side effects, `import *`, barrel files, and missing `sideEffects: false`? How do I verify what actually made it into my bundle?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.3 — tree shaking and dead code elimination,
investigation. Completed so far: Tracks 1–7, Modules 8.1–8.2.

Scaffold me a Vite project that ships far more code than it uses: a big utility module where only
one function is imported but the whole thing lands in the bundle; an `import * as utils` that
defeats shaking; a barrel `index.js` re-exporting a heavy library so importing one helper pulls in
everything; a module with top-level side effects that blocks elimination; and a CommonJS dep that
can't be shaken. Include a bundle visualizer.

Tell me only how to run the build and open the bundle visualizer. Do not name the causes. I will
inspect the bundle, find the code that shouldn't be there, and trace WHY each piece survived. Make
me fix them (named imports, avoid `import *`, deep imports instead of the barrel, mark side-effect-
free, isolate real side effects) and re-measure the bundle. Make me explain why static ESM is what
makes shaking possible and what each pattern did to break it.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- A utility module where one named function is used but the whole module is included — often because of how it's imported or a missing `sideEffects` hint. Fix: ensure named imports + side-effect-free marking so unused exports drop.
- `import * as utils from './utils'` — namespace import that makes the bundler retain everything (it can't prove which members are unused). Fix: named imports.
- A barrel `index.js` re-exporting a heavy library; importing one helper through it pulls the whole library. Fix: deep/direct imports, or a barrel the bundler can shake.
- A module with top-level side effects (it runs code on import), so the bundler can't safely remove it. Fix: isolate the side effect, or mark the package `"sideEffects": false` when accurate.
- A CommonJS dependency that can't be tree-shaken (dynamic `require`, no static structure).
- The visualizer shows the bloat before and the shrink after.

The principle: **tree shaking works only because static ESM lets the bundler prove an export is never used — and side effects, namespace imports, barrels, and CommonJS each rob it of that certainty.** Shipping minimal JS is mostly about importing in shakeable ways and being honest about side effects.
</details>

---

## Module 8.4 — Bundle Analysis & Code Splitting

**Difficulty:** ●●●●○ · **Format:** Investigation · **Stack:** React + Vite

### Prep — Read Before You Start

**Primary sources:**
- **web.dev — Code splitting with dynamic import()** — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting — *Focus on: splitting at route/component boundaries to defer non-critical code.*
- **React docs — `lazy` and `Suspense`** — https://react.dev/reference/react/lazy — *Focus on: lazy-loading a component and its bundle.*
- **Vite — Build / manualChunks** — https://vite.dev/guide/build — *Focus on: chunking strategy and vendor splitting.*
- **rollup-plugin-visualizer** — https://github.com/btd/rollup-plugin-visualizer — *Focus on: reading the treemap to find the biggest contributors.*

**Search prompts:**
- `"react lazy suspense route code splitting"`
- `"bundle analyzer treemap find large dependency"`
- `"vite manualChunks vendor splitting"`
- Ask an AI: *"Explain code splitting: how `import()` and `React.lazy` create separate chunks loaded on demand, and how to split by route so the initial bundle stays small. How do I read a bundle treemap to find the biggest offenders, and when should I split a vendor chunk vs lazy-load a feature? What's the cost of over-splitting?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.4 — bundle analysis and code splitting,
investigation. Completed so far: Tracks 1–7, Modules 8.1–8.3.

Scaffold me a React + Vite multi-route app with a deliberately bloated initial bundle: a heavy
charting/date/markdown library imported eagerly at the top level but only used on one rarely-
visited route; all routes bundled into the main chunk (no route splitting); and a large component
loaded upfront that's only shown behind a tab/modal. Include the bundle visualizer.

Tell me only how to run the build, open the treemap, and measure the initial chunk size. Do not
tell me what's heavy. I will read the treemap, identify the largest contributors and what's loaded
that shouldn't be upfront, then split: lazy-load routes with `React.lazy`/`Suspense`, dynamic-
import the heavy library at its point of use, and defer the behind-a-tab component — re-measuring
the initial bundle each time. Make me explain what moved to which chunk and the load-time win, plus
when over-splitting hurts.
```

### 🔒 SPOILER — What You Should Find & Fix

<details>
<summary>Click only after you finish or give up</summary>

- A heavy library (charts/date/markdown) imported eagerly but used on one route — it bloats the initial chunk. Fix: dynamic `import()` at the point of use (or lazy-load the route that needs it) so it becomes a separate, on-demand chunk.
- No route-level splitting: every route's code is in the main bundle. Fix: `React.lazy(() => import('./Route'))` + `<Suspense>` so each route is its own chunk loaded on navigation.
- A large component shown only behind a tab/modal but loaded upfront. Fix: lazy-load it when the tab/modal opens.
- The treemap shows the initial chunk dominated by deferred-able code before, and a much smaller initial chunk + several lazy chunks after.
- The tradeoff: over-splitting creates many tiny requests and waterfalls; split at meaningful boundaries (routes, heavy features), not everything.

The principle: **code splitting keeps the initial bundle small by deferring non-critical code into chunks loaded on demand (`import()`/`React.lazy`), and the bundle treemap tells you exactly what to defer.** The goal is the smallest initial payload that renders the first view; everything else loads when it's actually needed — without fragmenting into so many chunks that loading becomes a waterfall.
</details>

---

## Module 8.5 — TypeScript: Narrowing & Discriminated Unions

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** TypeScript + React

### Prep — Read Before You Start

**Primary sources:**
- **TS Handbook — Narrowing** — https://www.typescriptlang.org/docs/handbook/2/narrowing.html — *Read fully. Focus on: type guards, control-flow narrowing, `typeof`/`in`/`instanceof`, and discriminated unions.*
- **TS Handbook — Unions and Discriminated Unions** — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#unions — *Focus on: a shared literal "tag" enabling exhaustive narrowing.*
- **TS Handbook — `never` and exhaustiveness checking** — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking — *Focus on: using `never` to force handling every case.*

**Optional course resources:**
- Matt Pocock — *Total TypeScript* — the narrowing / discriminated union exercises.

**Search prompts:**
- `"typescript discriminated union exhaustiveness never"`
- `"typescript narrowing type guard in operator"`
- Ask an AI: *"Explain TypeScript narrowing and discriminated unions: how a shared literal discriminant (`type: 'success' | 'error'`) lets TS narrow a union in each branch, how `in`/`typeof`/`instanceof` and custom type guards narrow, and how a `never` default forces exhaustive handling. Show me how this models 'impossible states' (tie-in to the state machine modules)."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.5 — TypeScript narrowing and discriminated
unions. Completed so far: Tracks 1–7, Modules 8.1–8.4.

Build me a TypeScript + React project that models async/UI state with TYPES that allow impossible
states and miss cases: a state typed as an object with optional `data`, `error`, and `loading`
booleans (so `loading && error && data` is representable and the UI mishandles it); a `switch` over
a union that silently misses a case (no exhaustiveness check, so adding a variant doesn't error);
and a spot where the code accesses `.data` without narrowing, hitting `undefined` at runtime.

Tell me only how to run it (and to watch both the type errors and the runtime behavior). Do not
name the bugs. I diagnose by reasoning about which states the type permits and which the code
fails to narrow. Make me refactor to a DISCRIMINATED UNION (`{status:'idle'} | {status:'loading'}
| {status:'success', data} | {status:'error', error}`) so impossible states don't typecheck, and
add a `never` exhaustiveness check. Explain how narrowing makes each branch safe.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- State modeled as `{ data?: T; error?: E; loading: boolean }` — allows contradictory combinations (`loading` true while `data` and `error` are both set), and forces unsafe optional access.
- A `switch`/`if` over a union with no exhaustiveness check, so a newly added variant is silently unhandled (no compile error).
- Accessing `.data` without narrowing (it's possibly `undefined`), causing a runtime crash the types didn't catch because the model was too loose.
- Fix: a discriminated union with a `status` discriminant; each branch carries only the data valid for that state (`success` has `data`, `error` has `error`), so impossible states are untypeable and TS narrows `.data`/`.error` safely per branch.
- A `default: const _exhaustive: never = state; ` (or a helper) that fails to compile if any case is unhandled — forcing exhaustiveness.

The principle: **a discriminated union plus exhaustiveness checking makes illegal states unrepresentable and forces you to handle every case — the type system enforcing the state-machine discipline from Tracks 5–6.** Narrowing then guarantees each branch only sees the data that's valid there, turning a class of runtime bugs into compile errors.
</details>

---

## Module 8.6 — TypeScript: Generics & Type-Level Programming

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** TypeScript

### Prep — Read Before You Start

**Primary sources:**
- **TS Handbook — Generics** — https://www.typescriptlang.org/docs/handbook/2/generics.html — *Focus on: generic functions, constraints (`extends`), and inference.*
- **TS Handbook — Conditional & Mapped Types** — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html — *Focus on: `T extends U ? X : Y`, `infer`, and mapped types.*
- **TS Handbook — Utility Types** — https://www.typescriptlang.org/docs/handbook/utility-types.html — *Focus on: how `Partial`, `Pick`, `Omit`, `Record`, `ReturnType` are built (then build your own).*
- **TS — `keyof`, indexed access, template literal types** — https://www.typescriptlang.org/docs/handbook/2/keyof-types.html — *Focus on: deriving types from other types.*

**Optional course resources:**
- Matt Pocock — *Total TypeScript* — the generics and type-transformation modules.
- Type Challenges — https://github.com/type-challenges/type-challenges — *Do a few "easy"/"medium" challenges.*

**Search prompts:**
- `"typescript generic constraint extends keyof"`
- `"typescript conditional type infer"`
- `"implement Pick Omit utility type from scratch"`
- Ask an AI: *"Teach me TypeScript generics with constraints and inference, then conditional types with `infer`, and mapped types. Walk me through reimplementing `Pick`, `Omit`, and `ReturnType` from scratch. Then show me a strongly-typed function (like a `get(obj, key)`) where the return type is derived from the key via `keyof` + indexed access."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.6 — generics and type-level programming, build
challenge. Completed so far: Tracks 1–7, Modules 8.1–8.5. This is hard — push me.

Give me a set of typed challenges to implement (types + runtime where relevant), no solution code:
(1) a `get(obj, key)` whose return type is `obj[key]`'s type via `keyof`+indexed access; (2)
reimplement `Pick<T,K>`, `Omit<T,K>`, and `ReturnType<F>` from scratch using mapped/conditional
types + `infer`; (3) a generic, fully-typed `pipe`/`compose` that infers types through the chain;
and (4) a `DeepReadonly<T>` mapped type. Provide failing type-level assertions (e.g. `Expect<Equal
<...>>`).

Review me like a senior. When a generic loses type info (returns `any`/widens), do not fix it —
ask me where inference broke and which constraint or `infer` I'm missing. For the utility-type
reimplementations, make me explain each in plain English first. For `pipe`, make me explain how
each function's output type must feed the next. Make me articulate when this complexity is worth it
vs over-engineering.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `get`: `function get<T, K extends keyof T>(obj: T, key: K): T[K]` — the constraint `K extends keyof T` and indexed access `T[K]` derive the precise return type.
- `Pick<T,K extends keyof T> = { [P in K]: T[P] }`; `Omit<T,K> = Pick<T, Exclude<keyof T, K>>` (or a mapped type with `as`); `ReturnType<F> = F extends (...a:any)=>infer R ? R : never` (conditional + `infer`).
- A typed `pipe`/`compose` using overloads or recursive generics so each function's input type matches the previous output — the hard part is threading inference through the tuple of functions.
- `DeepReadonly<T>`: a recursive mapped type applying `readonly` and recursing into object properties.
- Type-level assertions (`Expect<Equal<A,B>>`) passing.
- Common mistakes: missing `extends keyof` constraints (everything widens to `any`); forgetting `infer`; non-recursive deep types; over-complex types where a simple generic suffices.
- The judgment call: this power is for library/design-system boundaries; application code rarely needs deep type gymnastics.

The principle: **generics with constraints and inference, plus conditional/mapped types and `infer`, let you derive types from other types — so APIs stay precisely typed without manual annotation.** It's how the utility types and well-typed libraries work; the senior skill is wielding exactly enough of it to make an API safe, and no more.
</details>

---

## Module 8.7 — A Type-Safe Design System Component

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** TypeScript + React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **React TypeScript Cheatsheet — Components & Props** — https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example — *Focus on: typing props, children, and component generics.*
- **TS — Polymorphic components / `as` prop** — https://www.totaltypescript.com/pass-component-as-prop-correctly — *Focus on: the `as` prop pattern typed correctly.*
- **TS Handbook — Discriminated unions for props** — https://www.typescriptlang.org/docs/handbook/2/narrowing.html — *Focus on: mutually-exclusive prop combinations.*
- **MDN — `ComponentPropsWithoutRef` usage / forwardRef** — https://react.dev/reference/react/forwardRef — *Focus on: forwarding refs and inheriting native element props.*

**Search prompts:**
- `"react typescript polymorphic component as prop"`
- `"typescript mutually exclusive props discriminated union"`
- `"react extend html button props typescript"`
- Ask an AI: *"Show me how to build a type-safe Button/Box component in TypeScript: extending the native element's props (`ComponentPropsWithoutRef`), forwarding a ref, a variant prop as a union, mutually-exclusive props via a discriminated union (e.g. either `href` for a link OR `onClick` for a button, never both), and an optional polymorphic `as` prop. Why does each technique prevent a real misuse?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.7 — a type-safe design system component, build
challenge. Completed so far: Tracks 1–7, Modules 8.1–8.6.

Give me a spec to build ONE production-grade, type-safe component (a `Button` or `Card`) in
TypeScript + React + Tailwind that: extends the underlying native element's props (so all valid
HTML attributes pass through) and forwards a ref; exposes a typed `variant`/`size` union; uses a
DISCRIMINATED UNION so mutually-exclusive configurations are enforced at the type level (e.g. a
"link button" requires `href` and forbids `onClick`, a normal button is the reverse); and is
accessible (correct element, focus, disabled handling — tie in Track 7). Optionally support a
polymorphic `as`. Give me no solution code.

Review me like a senior. If my props allow a nonsensical combination (both `href` and `onClick`,
or a variant that doesn't exist) to typecheck, do not fix it — ask me to write the misuse and show
me it compiles, then make me tighten the types so it can't. Make me explain how each typing
technique blocks a real mistake, and verify the component is accessible.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Extending native props: `ComponentPropsWithoutRef<'button'>` (or `'a'`) so all valid attributes are typed and passed through; `forwardRef` to forward the ref to the DOM node.
- Variant/size as string-literal unions (`'primary' | 'secondary'`) so invalid variants don't typecheck; mapped to Tailwind classes.
- A discriminated union making mutually-exclusive props impossible: e.g. `({ href: string } & AnchorProps) | ({ onClick: ... } & ButtonProps)` so providing both — or the wrong one for the element — is a type error; the component renders `<a>` vs `<button>` accordingly.
- Accessibility: renders the correct element (`<a>` for navigation, `<button>` for actions), handles `disabled`/`aria-disabled`, keeps focus behavior correct (ties to Track 7).
- Optional polymorphic `as` typed so props follow the chosen element.
- Common mistakes: `any`/loose props that allow nonsensical combinations; not forwarding the ref; reinventing native attributes instead of extending them; a "link" that's a `<button>` (or vice versa); variant typos not caught.

The principle: **a great design-system component encodes its correct usage in its types — extending native props, forwarding refs, and using discriminated unions so misuse fails to compile — while staying accessible by rendering the right element.** The type system becomes documentation and a guardrail: teammates literally can't use the component wrong.
</details>

---

## Module 8.8 — Write A Custom ESLint Rule

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Node + ESLint (AST)

### Prep — Read Before You Start

**Primary sources:**
- **ESLint — Custom Rules** — https://eslint.org/docs/latest/extend/custom-rules — *Focus on: the rule object, `create(context)`, visitor methods on AST node types, and `context.report`.*
- **AST Explorer** — https://astexplorer.net — *Paste code, pick the ESPree parser, and see the AST node types you'll match.*
- **ESLint — Selectors** — https://eslint.org/docs/latest/extend/selectors — *Focus on: node selectors for targeting patterns.*
- **ESTree spec** — https://github.com/estree/estree — *Reference for node shapes.*

**Search prompts:**
- `"write custom eslint rule create context report"`
- `"ast explorer eslint node type visitor"`
- `"eslint rule autofix fixer"`
- Ask an AI: *"Walk me through writing a custom ESLint rule: the rule module shape (`meta` + `create(context)`), how `create` returns visitor functions keyed by AST node type, how to inspect a node and call `context.report` with a message and node, and how to add an autofix with a `fixer`. How do I use AST Explorer to find the node type for the pattern I want to flag?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.8 — write a custom ESLint rule, build
challenge. Completed so far: Tracks 1–7, Modules 8.1–8.7.

Give me a spec for a custom ESLint rule that enforces a real team convention — pick one, e.g.
"disallow `console.log` (but allow `console.warn`/`error`)", or "ban hardcoded hex colors in JSX
style props (must use a token)", or "require `alt` on a custom `<Image>` component". Give me a
starter with the rule file stub and a test file (valid + invalid cases using ESLint's RuleTester).
My job: implement `create(context)` with the right AST visitor(s) and `context.report`, plus an
autofix where sensible. Give me no solution code.

Review me like a senior. Before coding, make me open AST Explorer and state the exact node type and
the property path to the thing I'm checking. If my rule over-matches (flags allowed cases) or
under-matches, do not fix it — ask me what node my selector actually visits and what distinguishes
the valid from invalid AST. Make me explain how lint rules are just AST visitors, and how the
fixer rewrites source.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A rule module with `meta` (type, docs, `fixable`, messages) and `create(context)` returning a visitor object — e.g. `CallExpression(node)` for the console rule, checking `node.callee` is a `MemberExpression` with `object.name === 'console'` and `property.name === 'log'`, then `context.report({ node, messageId })`.
- Precise matching so allowed cases (`console.warn`) are NOT flagged — distinguishing valid vs invalid by the exact AST shape (verified in AST Explorer).
- An autofix via `fix(fixer)` that removes/replaces the offending node where it's safe (and omitting autofix where it isn't).
- Tests with `RuleTester`: valid cases that must pass and invalid cases with expected errors/output.
- Common mistakes: matching too broadly (flagging `console.error`); reading the wrong property path; an unsafe autofix that breaks code; not testing both valid and invalid cases; visiting a node type that doesn't actually contain the pattern.

The principle: **an ESLint rule is just an AST visitor — `create` returns handlers for node types, you inspect the matched nodes, and `context.report` (optionally with a fixer) flags or rewrites them.** Once you've seen the AST and written a rule, the entire linting/codemod ecosystem (and tools like Babel, Prettier, jscodeshift) becomes legible: it's all parse → visit → report/transform.
</details>

---

## Module 8.9 — Testing: Unit, Integration & The Testing Trophy

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Vitest + Testing Library

### Prep — Read Before You Start

**Primary sources:**
- **Testing Library — Guiding Principles & queries** — https://testing-library.com/docs/queries/about/ — *Focus on: querying by role/label/text (as users do), and the query priority order.*
- **Kent C. Dodds — The Testing Trophy** — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications — *Focus on: why integration tests give the best ROI.*
- **Kent C. Dodds — Testing Implementation Details** — https://kentcdodds.com/blog/testing-implementation-details — *Focus on: why testing internals makes brittle tests.*
- **Vitest — Guide** — https://vitest.dev/guide/ — *Focus on: the test runner basics and mocking.*

**Search prompts:**
- `"testing library query by role not test id"`
- `"testing implementation details brittle tests"`
- `"testing trophy integration unit e2e"`
- Ask an AI: *"Explain the Testing Trophy and why integration tests (rendering a component and interacting as a user) usually beat isolated unit tests for UI. Explain Testing Library's philosophy — query by role/label/text, not by implementation — and what 'testing implementation details' means and why it produces brittle tests. Show me a good vs bad test of the same component."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.9 — testing unit & integration. Completed so
far: Tracks 1–7, Modules 8.1–8.8.

Build me a React + Vitest + Testing Library project with a small feature (a form, or a filterable
list) and a test suite full of ANTI-PATTERNS: tests that query by `data-testid`/CSS classes/
component internals instead of user-facing roles/labels; tests asserting on state/implementation
details that break when you refactor without changing behavior; tests that mock so much the test no
longer proves anything; and a missing integration test for the actual user flow. Some tests pass
but are brittle; one gives false confidence.

Tell me only how to run the tests. Do not name the anti-patterns. I diagnose by asking, for each
test, "would this break if I refactored internals without changing behavior?" and "does this test
what the user experiences?" Make me rewrite them to query by role/label/text, test behavior not
internals, mock only the boundaries, and add the missing integration test. Explain the trophy and
why user-centric tests survive refactors.
```

### 🔒 SPOILER — The Planted Anti-Patterns

<details>
<summary>Click only after you finish or give up</summary>

- Queries by `data-testid`, class names, or component internals instead of accessible roles/labels/text — brittle and not user-representative. Fix: `getByRole`/`getByLabelText`/`getByText` (which also nudges accessibility).
- Assertions on internal state, props, or instance methods (implementation details) that break on harmless refactors. Fix: assert on rendered output/behavior the user sees.
- Over-mocking (mocking the very units under test, or all collaborators) so the test passes regardless of real behavior — false confidence. Fix: mock only true boundaries (network), test the integrated behavior.
- No integration test for the actual flow (fill form → submit → see result). Fix: add one that drives the component as a user would.
- A test that passes but asserts nothing meaningful.

The principle: **test behavior the way a user experiences it — query by role/label/text, assert on visible outcomes, mock only real boundaries — so tests survive refactors and actually catch regressions.** The Testing Trophy favors integration tests because they exercise real interactions for the best confidence-to-cost ratio; testing implementation details buys brittleness, not safety.
</details>

---

## Module 8.10 — What To Test: Coverage, Edge Cases & Confidence

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** React + Vitest + Testing Library

### Prep — Read Before You Start

**Primary sources:**
- **Testing Library — Common mistakes / async** — https://testing-library.com/docs/dom-testing-library/api-async/ — *Focus on: `findBy`/`waitFor` for async UI, and avoiding flaky tests.*
- **Kent C. Dodds — How to know what to test** — https://kentcdodds.com/blog/how-to-know-what-to-test — *Focus on: testing use cases, prioritizing by risk.*
- **Martin Fowler — Test Coverage** — https://martinfowler.com/bliki/TestCoverage.html — *Focus on: why 100% coverage is a poor goal and what coverage really tells you.*

**Search prompts:**
- `"what to test prioritize by risk use cases"`
- `"test coverage misleading 100 percent"`
- `"testing library async findBy waitFor flaky"`
- Ask an AI: *"How do I decide WHAT to test? Explain prioritizing by risk and use case rather than chasing coverage %, and why 100% coverage can still miss the bugs that matter. What edge cases get forgotten (empty states, errors, loading, boundary values, async races)? How do I write reliable async tests with `findBy`/`waitFor` instead of flaky ones?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.10 — what to test, build challenge. Completed
so far: Tracks 1–7, Modules 8.1–8.9.

Give me a spec for a component with real complexity and risk — e.g. an async data list with
loading/empty/error/success states, pagination, retry, and a filter — plus a starter with the
component built and NO tests (or only a trivial happy-path test that reports high coverage while
missing the important cases). My job: decide what's worth testing by RISK, and write a focused
suite covering the states and edge cases that actually matter (empty, error, loading, boundary
inputs, async race/retry), using reliable async queries. Give me no solution code.

Review me like a senior. If I chase coverage by testing trivial getters while ignoring the error
and empty states, do not let it slide — ask me which untested case would hurt a user most. If my
async tests are flaky (no `findBy`/`waitFor`), ask me what I'm racing. Make me justify each test by
the risk it mitigates and explain why coverage % is a weak proxy for confidence.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Tests prioritized by risk and use case: the states a user actually hits — loading, empty, error (and error→retry), success, and boundaries (first/last page, single item, max input) — not trivial internals.
- The frequently-forgotten cases covered: empty state, error state, the loading→loaded transition, retry after failure, and async races (a newer request superseding an older one).
- Reliable async testing with `findBy*`/`waitFor` (awaiting the UI settling) rather than synchronous assertions that race the update and flake.
- An explicit acknowledgment that high coverage from happy-path-only tests is false confidence — the error/empty/edge paths are where bugs live.
- Common mistakes: testing trivial code for coverage; skipping error/empty states; flaky async tests; not testing the retry/race; treating coverage % as the goal.

The principle: **test what's risky and user-facing — the empty, error, loading, boundary, and async cases — because that's where real bugs hide, and coverage percentage measures lines executed, not scenarios validated.** Confidence comes from exercising the paths that would actually hurt a user if they broke, with async tests that wait correctly instead of racing.
</details>

---

## Module 8.11 — CI: Pipelines, Caching & Speed

**Difficulty:** ●●●○○ · **Format:** Build challenge + broken project · **Stack:** GitHub Actions

### Prep — Read Before You Start

**Primary sources:**
- **GitHub Actions — Workflow syntax** — https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions — *Focus on: jobs, steps, triggers, and the `needs` dependency between jobs.*
- **GitHub Actions — Caching dependencies** — https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows — *Focus on: cache keys and restoring node_modules / build caches.*
- **GitHub Actions — Matrix / concurrency** — https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs — *Focus on: parallelism and cancelling superseded runs.*

**Search prompts:**
- `"github actions cache node_modules speed up"`
- `"github actions parallel jobs needs matrix"`
- `"ci pipeline fail fast lint test build"`
- Ask an AI: *"Explain a good frontend CI pipeline in GitHub Actions: which checks to run (install, lint, typecheck, test, build), how to parallelize independent jobs, how dependency/build caching cuts time, how `concurrency` cancels superseded runs, and why fast feedback matters. What makes a pipeline slow or flaky, and how do I fix it?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.11 — CI pipelines, caching, speed. Completed
so far: Tracks 1–7, Modules 8.1–8.10.

Give me a starter repo (a small frontend app) with a SLOW, badly-structured GitHub Actions workflow:
everything in one serial job; no dependency caching (fresh `npm install` every run); independent
checks (lint, typecheck, test, build) running sequentially; no concurrency control (superseded
pushes keep running); and a step ordering that fails late instead of fast. Give me the broken
workflow file.

Tell me only how to read the workflow and where runs show timing. Do not name the problems. I
diagnose by reading the YAML and the job timeline. Make me refactor it: cache dependencies (and
build artifacts), split independent checks into parallel jobs, add `concurrency` to cancel
superseded runs, order for fail-fast, and reuse install across jobs. Explain why each change speeds
feedback and what a cache key must capture.
```

### 🔒 SPOILER — The Planted Issues

<details>
<summary>Click only after you finish or give up</summary>

- One monolithic serial job doing install → lint → typecheck → test → build sequentially. Fix: split independent checks into parallel jobs (with a shared install/cache), so they run concurrently.
- No caching: every run does a cold `npm install`. Fix: cache the package manager store / `node_modules` keyed on the lockfile hash (and optionally cache build outputs) — a correct cache KEY captures the lockfile so it invalidates when deps change.
- No `concurrency` group: pushing twice leaves the old run consuming minutes. Fix: a `concurrency` block keyed on the branch with `cancel-in-progress`.
- Late failure: an expensive build runs before cheap lint/typecheck. Fix: order so fast checks fail first (or run in parallel and fail fast).
- Possibly a flaky test step with no retry/seed control.

The principle: **fast CI is parallel, cached, fail-fast, and cancels superseded runs — because the value of CI is quick feedback, and serial uncached pipelines waste minutes on every push.** A correct cache key (lockfile-based) and parallel independent jobs are the biggest wins; `concurrency` stops you paying for runs you no longer care about.
</details>

---

## Module 8.12 — Git Internals & Real Workflows

**Difficulty:** ●●●●○ · **Format:** Investigation + build challenge · **Stack:** Git (CLI)

### Prep — Read Before You Start

**Primary sources:**
- **Pro Git — Git Internals (Plumbing and Porcelain; Git Objects)** — https://git-scm.com/book/en/v2/Git-Internals-Git-Objects — *Read this chapter. Focus on: blobs, trees, commits as a content-addressed object graph; refs as pointers.*
- **Pro Git — Git Branching: Rebasing** — https://git-scm.com/book/en/v2/Git-Branching-Rebasing — *Focus on: rebase vs merge and what each does to history.*
- **Pro Git — Rewriting History (interactive rebase)** — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History — *Focus on: `rebase -i`, squashing, and the reflog safety net.*

**Search prompts:**
- `"git objects blob tree commit content addressable"`
- `"git rebase vs merge when to use"`
- `"git reflog recover lost commit"`
- Ask an AI: *"Explain Git's object model: how blobs, trees, and commits form a content-addressed DAG, and how branches/HEAD are just pointers. Then explain rebase vs merge (what history each produces), interactive rebase for cleaning up commits, and how the reflog lets me recover from 'mistakes' like a bad rebase or reset."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.12 — git internals and workflows. Completed so
far: Tracks 1–7, Modules 8.1–8.11.

Two parts.

Part 1 (investigation): give me a sequence of commands to run in a throwaway repo to SEE the object
model — make a commit, then use `git cat-file`/`git rev-parse`/`git ls-tree` to walk from the
commit to its tree to its blobs, and inspect `.git/refs` and HEAD. Do not explain the output; make
me narrate what each object is and how they point at each other.

Part 2 (build challenge): give me a messy-history scenario (a feature branch with WIP commits,
typos in messages, and a commit that belongs elsewhere) and have me clean it with interactive
rebase (squash/reword/reorder), then resolve a deliberate rebase conflict, then deliberately "lose"
a commit with a hard reset and RECOVER it via the reflog.

Review me strictly. When I'm unsure whether to rebase or merge, ask me what history I want to
produce. After recovery, make me explain why the commit was never actually gone.
```

### 🔒 SPOILER — What You Should Understand

<details>
<summary>Click only after you finish or give up</summary>

Part 1: `git cat-file -p <commit>` shows the commit points to a TREE and parent(s); the tree (`git ls-tree`) points to BLOBS (file contents) and subtrees; every object is named by the hash of its content (content-addressed). Branches and HEAD in `.git/refs`/`.git/HEAD` are just files containing a commit hash (pointers). Commits form a DAG via parent links.

Part 2:
- Interactive rebase (`git rebase -i`): `squash`/`fixup` to combine WIP commits, `reword` to fix messages, `reorder`/`drop` to clean history — producing a linear, readable branch history.
- Conflict resolution during rebase: fix the files, `git add`, `git rebase --continue`.
- Reset + recovery: `git reset --hard` moves the branch pointer but the old commit still exists as a dangling object; `git reflog` shows where HEAD has been, and `git reset --hard <reflog-hash>` (or branching from it) recovers it.
- Rebase vs merge: rebase rewrites commits onto a new base (linear history, don't do it to shared/pushed branches); merge preserves both histories with a merge commit.

The principle: **Git is a content-addressed DAG of objects (blobs/trees/commits) with refs as movable pointers — so "rewriting history" just creates new commits and moves pointers, and the reflog means almost nothing is ever truly lost.** Understanding the object model turns rebase, reset, and recovery from scary incantations into obvious pointer manipulation.
</details>

---

## Module 8.13 — Frontend Security: XSS, CSRF & CSP

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + a small server

### Prep — Read Before You Start

**Primary sources:**
- **OWASP — Cross Site Scripting (XSS)** — https://owasp.org/www-community/attacks/xss/ — *Focus on: stored/reflected/DOM XSS and how untrusted data becomes executable.*
- **MDN — Content Security Policy (CSP)** — https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP — *Focus on: restricting script sources as defense-in-depth.*
- **OWASP — CSRF & SameSite cookies** — https://owasp.org/www-community/attacks/csrf — *Focus on: how CSRF works and how SameSite/tokens defend.*
- **React docs — `dangerouslySetInnerHTML`** — https://react.dev/reference/react-dom/components/common#dangerously-setting-the-inner-html — *Focus on: why it's named "dangerously" and when it's an XSS hole.*

**Search prompts:**
- `"react dangerouslySetInnerHTML xss sanitize"`
- `"content security policy block inline script"`
- `"csrf samesite cookie token protection"`
- Ask an AI: *"Explain the main frontend security risks: XSS (how untrusted data injected into the DOM executes, including via `dangerouslySetInnerHTML`), CSRF (how a malicious site triggers authenticated requests, and how SameSite cookies + tokens stop it), and CSP (how it limits which scripts can run as defense-in-depth). For each, show a vulnerable pattern and its fix."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.13 — frontend security. Completed so far:
Tracks 1–7, Modules 8.1–8.12.

Build me a small React app (+ a minimal server) with REAL, demonstrable vulnerabilities (in a safe
local sandbox): a stored XSS where user-submitted content is rendered via `dangerouslySetInnerHTML`
without sanitization, so a `<script>`/`onerror` payload executes; a DOM XSS where a URL parameter is
injected into the page unsafely; an auth token kept in `localStorage` (XSS-exfiltratable); no
Content-Security-Policy; and a state-changing request with no CSRF protection. Include a benign
"attacker" payload that proves each hole.

Tell me only how to run it and trigger the payloads. Do not name the fixes. I will exploit each
vulnerability to SEE it, then fix it: sanitize/avoid `dangerouslySetInnerHTML`, never inject
untrusted data as HTML, move the token to an HttpOnly cookie, add a CSP, and add CSRF protection
(SameSite + token). Make me explain, for each, how untrusted data became dangerous and how the fix
closes it.
```

### 🔒 SPOILER — The Planted Vulnerabilities

<details>
<summary>Click only after you finish or give up</summary>

- Stored XSS: user content rendered with `dangerouslySetInnerHTML` (or via `innerHTML`) without sanitization, so an injected `<img onerror=...>`/`<script>` executes. Fix: render as text (React escapes by default), or sanitize (e.g. DOMPurify) if HTML is truly needed.
- DOM XSS: a URL/query parameter written into the DOM as HTML. Fix: treat it as text; never build HTML from untrusted input.
- Token in `localStorage`: any XSS can read and exfiltrate it. Fix: store the session in an `HttpOnly; Secure; SameSite` cookie (JS can't read it) — ties to Track 4 storage.
- No CSP: nothing limits which scripts run. Fix: a `Content-Security-Policy` restricting `script-src` (defense-in-depth that blunts injected scripts even if one slips through).
- CSRF: a state-changing endpoint relying only on the cookie, triggerable cross-site. Fix: `SameSite` cookies + an anti-CSRF token / origin checks.

The principle: **frontend security is fundamentally about untrusted data: XSS is untrusted data becoming executable (so escape/sanitize and avoid `dangerouslySetInnerHTML`), CSRF is the browser sending credentials on a forged request (so SameSite + tokens), and CSP is a safety net limiting what can run.** React escapes by default for a reason; the holes are where you opt out of that protection or trust input you shouldn't.

*Note: this module deals with security exploits in a deliberately vulnerable LOCAL sandbox for defensive learning — to understand and fix the holes. Never run these payloads against systems you don't own.*
</details>

---

## Module 8.14 — Performance Budgets & Shipping Fast Frontends

**Difficulty:** ●●●●○ · **Format:** Investigation + build challenge · **Stack:** React + Vite + Lighthouse CI

### Prep — Read Before You Start

**Primary sources:**
- **web.dev — Performance budgets 101** — https://web.dev/articles/performance-budgets-101 — *Focus on: setting budgets (size, metrics) and enforcing them.*
- **web.dev — Your first performance budget** — https://web.dev/articles/your-first-performance-budget — *Focus on: choosing budget thresholds.*
- **Lighthouse CI** — https://github.com/GoogleChrome/lighthouse-ci — *Focus on: asserting on Lighthouse scores / metrics in CI so regressions fail the build.*
- **web.dev — Core Web Vitals (revisit)** — https://web.dev/articles/vitals — *The metrics your budget targets.*

**Search prompts:**
- `"performance budget bundle size lighthouse ci"`
- `"fail build on bundle size regression"`
- `"lighthouse ci assert core web vitals"`
- Ask an AI: *"Explain performance budgets: setting limits on bundle size and Core Web Vitals, and enforcing them automatically (e.g. bundle-size checks and Lighthouse CI in the pipeline) so a regression fails the build. How do I pick budget thresholds, and how does treating performance as a CI gate change team behavior?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 8.14 — performance budgets, capstone. Completed
so far: ALL of Tracks 1–8 so far. This closes the curriculum — make it real.

Give me a starter: a React + Vite app that currently passes its tests but has REGRESSED on
performance — the bundle has bloated and Core Web Vitals slipped (pull in the issues you learned to
diagnose across the course: an eagerly-imported heavy lib, an unoptimized LCP image, a long
interaction task, layout shift). My job: establish performance BUDGETS (bundle size limits + Web
Vitals thresholds) and wire them into CI so this regression FAILS the build, then fix the app until
it passes the budget. Give me no solution code for the fixes.

Review me like a senior. Make me first measure (Lighthouse, bundle visualizer) and set defensible
budget numbers with justification. Wire up the enforcement (bundle-size check + Lighthouse CI
assertions) so the failing build is red. Then make me apply everything from the course — code
splitting, image optimization, breaking up the long task, reserving layout space — until it's green.
Make me explain why a budget enforced in CI is what actually keeps an app fast over time.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Measuring first: Lighthouse for LCP/INP/CLS, a bundle visualizer for size, establishing the current (bad) baseline.
- Setting budgets with justification: e.g. an initial-JS size ceiling and Core Web Vitals thresholds (LCP, INP, CLS "good" ranges) — numbers tied to user experience, not arbitrary.
- Enforcement in CI: a bundle-size assertion (e.g. size-limit / bundlesize) AND Lighthouse CI with `assert` on the metrics, configured so exceeding a budget FAILS the build (red).
- Fixing the regression using the whole course: code-split/lazy-load the heavy library (8.4), optimize/preload the LCP image (Track 4), break up or defer the long interaction task (Track 4), reserve space to kill CLS (Track 4) — until the build is green within budget.
- Common mistakes: setting budgets so loose they never fail; not actually failing the build (just reporting); fixing symptoms without re-measuring; arbitrary thresholds with no user-experience rationale.

The principle: **performance is only durable when it's a budget enforced in CI — a measured limit on bundle size and Core Web Vitals that fails the build on regression — because otherwise apps slowly rot as features pile on.** This capstone ties the whole curriculum together: you diagnose with the tools from every track and prevent regression with an automated gate, which is exactly how senior engineers keep a frontend fast at scale.
</details>

---

*End of Track 8 — Engineering & Tooling. 14 modules.*

---

## 🎓 Curriculum Complete

That's the whole Opus General Frontend Primer: **8 tracks, 124 modules**, from a semantic `<button>` to a CI-enforced performance budget.

- **Track 1 — HTML & Semantics** (15)
- **Track 2 — CSS & Layout** (18)
- **Track 3 — JavaScript & the Language** (17)
- **Track 4 — The Browser as a Runtime** (15)
- **Track 5 — React & Next.js** (19)
- **Track 6 — State, Data & Async Architecture** (14)
- **Track 7 — Accessibility & Semantics in Practice** (12)
- **Track 8 — Engineering & Tooling** (14)

**Recommended order:** 1 → 2 → 3 → 4 → 7 → 5 → 6 → 8. (Accessibility sits before React deliberately — you'll build accessible React components instead of retrofitting them.)

**How to work:** one module at a time. Do the Prep (read the primary sources — you're becoming a documentation reader, not a course-doer). Paste the module's prompt into Claude Code from inside that module's folder; the root `CLAUDE.md` loads automatically. Hunt the bug yourself; use `hint` to climb one rung; only say `I give up` when you mean it; type `exam me` when you're ready to be tested. Commit each finished module. After each track, do a couple of GreatFrontend machine-coding problems to convert the depth into interview speed. Don't read the spoilers early — the struggle is the point.

Build the thing. Then go get the senior role.
