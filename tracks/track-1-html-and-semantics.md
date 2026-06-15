# The Opus General Frontend Primer — Prompts

*A first-principles, resistance-based curriculum for becoming an AI-proof, phenomenal frontend engineer. ~120 modules across 8 tracks, each ramping from absolute basics to genuinely hard.*

---

## What This Is

This is not a course. It is a sequence of **deliberate practice sessions**. In each one, an AI model scaffolds a real, running project — usually broken, sometimes empty — and then refuses to tell you what is wrong. You hunt the problem down yourself. The AI only helps when you ask for a hint, and only points to the exact line when you literally give up. Then it examines you on the underlying model and refuses to accept hand-wavy answers.

The goal is not to finish the modules. The goal is to become the kind of developer who can walk up to any broken frontend, observe the symptom, and reason their way to the cause from first principles — even for bugs they have never seen before. That is what makes you AI-proof: not knowing more facts than the model, but having a platform-level mental model the model cannot replace.

---

## How To Use This File

**1. Set up the repo once.**

```
frontend-primer/
  CLAUDE.md          ← the teaching contract (provided separately)
  .gitignore         ← contains: node_modules
  track-1-html/
    01-first-page/
    02-links-vs-buttons/
    ...
  track-2-css/
    ...
```

Put `CLAUDE.md` at the repo root. Every Claude Code session you launch from anywhere inside this repo will load it automatically. `git init` once at the root; each finished module is a commit. Add `node_modules` to `.gitignore` so it never gets tracked — no manual deleting.

**2. Per module, the loop is:**

- `cd` into the module's folder, create it if needed.
- Do the **Prep** reading first. Every time. Non-negotiable. The prep gives you just enough to *attempt* the session; the session gives the prep a place to live in your head.
- Launch Claude Code from inside the folder. It reads the root `CLAUDE.md` automatically.
- Paste the module's **Prompt**. The AI scaffolds the project.
- Run it, observe the symptom, and hunt. Struggle for real before reaching for a hint.
- Finish, get examined, log your wrong predictions, commit.

**3. Your three magic phrases (defined fully in `CLAUDE.md`):**

- `hint` — one rung up the hint ladder, no more.
- `I give up` — the only phrase that unlocks the exact line and answer.
- `exam me` — jump to the Socratic exam.

**4. Spoilers.** After each prompt there is a 🔒 **SPOILER** block listing the planted bugs. Do not read it before you start. It exists so you can check your work, or read it after `I give up`. Reading it first wastes the entire module.

---

## On Resources — Read The Docs, Not The Courses

You have spent your life doing courses. This curriculum is built to turn you into a documentation reader who learns from first principles. So every module's **Prep** leads with **primary sources**: MDN, web.dev, the actual specs, the actual source code. Read those. They are the real thing.

Where a course explains something exceptionally well, it is listed separately under **Optional course resources** — from your FrontendMasters library, Josh Comeau (CSS / React / Animations), Kevin Powell (CSS), and Matt Pocock (TypeScript). These are optional reinforcement, never the primary path. If you only ever read the primary sources, you will be fine. That is the point.

Each module also gives **Search prompts** — exact queries to paste into Google or a fresh AI chat to go deeper on a sub-topic on your own.

---

## The Gatekeeper (test out of what you already know)

You have 3 years of experience. Some early modules you can skip — but verify, do not assume. Before starting any track, paste this into a fresh Claude Code session in the repo:

```text
Follow the teaching contract in CLAUDE.md. Session type: Gatekeeper Exam.
I want to test out of [Track N], which covers: [paste that track's module titles].
Quiz me one question at a time, waiting for each answer: 8–12 questions spanning the
track's hardest ideas, plus 2 small live coding tasks I must actually write. Be strict —
partial credit does not pass. Verdict options only: "Skip the whole track", "Skip modules
N, N, N — do the rest", or "Do the full track." Justify the verdict by naming exactly
which answers exposed gaps.
```

---

## The Tracks

1. **HTML & Semantics** — the real foundation. Vanilla.
2. **CSS & Layout** — depth most developers never build. Vanilla-heavy.
3. **JavaScript & the Language** — closures, prototypes, async, the hard parts. Vanilla.
4. **The Browser as a Runtime** — rendering, the event loop, performance, Web APIs.
5. **React & Next.js** — your career core. The largest track.
6. **State, Data & Async Architecture** — the patterns that scale.
7. **Accessibility & Semantics in Practice** — what separates pros from juniors.
8. **Engineering & Tooling** — build, bundle, type, test, secure, ship.

Recommended order: 1 → 2 → 3 → 4 → 7 → 5 → 6 → 8. Accessibility (7) before React (5) is deliberate — it forces precision in your HTML that makes everything in React cleaner.

---

---

# TRACK 1 — HTML & SEMANTICS

*The foundation everyone skips. You cannot write good React if you write bad HTML. This track is almost entirely vanilla and almost entirely about meaning, not appearance.*

---

## Module 1.1 — Your First Semantic Page

**Difficulty:** ●○○○○ · **Format:** Build challenge · **Stack:** Vanilla HTML (zero CSS, zero JS)

### Prep — Read Before You Start

**Primary sources:**

- **MDN — HTML elements reference** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element — *Do not read it all. Skim the list top to bottom once, just to see the vocabulary of elements that exists. You are building a mental index of "what is available."*
- **MDN — HTML document structure** — https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Document_and_website_structure — *Focus on: the difference between `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, and what each one means.*
- **MDN — Getting started with HTML** — https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Getting_started — *Focus on: elements, attributes, nesting, and the anatomy of an element.*

**Optional course resources:**

- FrontendMasters — *Complete Intro to Web Development, v3* (Brian Holt) — the HTML section only.

**Search prompts:**

- `"semantic HTML vs div soup why it matters"`
- `"HTML5 sectioning elements when to use article vs section"`
- Ask an AI: *"What is the difference between `<section>` and `<div>`? When is each one the correct choice? Give me three concrete examples where choosing wrong causes a real problem."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.1 — semantic HTML, build challenge.
I have completed nothing in this curriculum yet. Treat me as someone who can write basic
tags but does not yet think in terms of meaning.

Give me a realistic content brief for a single web page — something with genuine structure:
a site header with navigation, a hero area, a main article made of several titled sections,
at least one figure with a caption, a pull quote, a small data table, a sidebar of related
links, and a footer. Describe the CONTENT in prose; do not give me any HTML.

My job: build the entire page in pure, semantic HTML. Zero CSS. Zero JavaScript. Every
piece of content must sit inside the element that correctly describes its MEANING, not its
appearance.

Do not accept my submission by glancing at it. Read my markup back to me the way a screen
reader would traverse it, region by region, and make me justify every element choice. If I
reach for a <div> or <span> where a meaningful element exists, do not tell me which one —
ask me what that content actually IS, and make me find the right element myself.

When the structure is correct, run the exam from the contract.
```

### 🔒 SPOILER — What This Module Is Really Testing

<details>
<summary>Click only after you finish or give up</summary>

This is a build, not a debug, so there are no planted bugs — but here is what a rigorous reviewer will push on:

- Did you use `<header>`/`<footer>` as document landmarks, and do you understand they can also appear inside an `<article>` with a different meaning?
- Is your heading hierarchy unbroken (one `<h1>`, then `<h2>`s, no skipping to `<h4>`)?
- Did you wrap the figure + caption in `<figure>`/`<figcaption>` rather than a `<div>` + `<p>`?
- Is the pull quote a `<blockquote>` (or `<q>`) and not styled text?
- Is the data table a real `<table>` with `<thead>`, `<tbody>`, `<th scope>`, and a `<caption>` — not a grid of divs?
- Is the nav a `<nav>` containing a `<ul>` of links, not a row of `<a>` tags floating in a div?
- Did you put exactly one `<main>` and not nest it?

The deeper lesson: **HTML is a description of meaning, and the browser, search engines, and assistive technology all consume that meaning.** Appearance is CSS's job. If you found yourself choosing elements based on how they look by default, that is the habit this module exists to break.

</details>

---

## Module 1.2 — Links Are Not Buttons, Buttons Are Not Divs

**Difficulty:** ●○○○○ · **Format:** Broken project · **Stack:** Vanilla HTML + minimal JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<a>` element** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a — *Focus on: what a link is FOR (navigation to a resource), and the `href` attribute.*
- **MDN — `<button>` element** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button — *Focus on: what a button is FOR (triggering an action), the `type` attribute, and why `type="button"` matters inside forms.*
- **MDN — Buttons (accessibility)** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/button_role — *Focus on: everything a native `<button>` gives you for free that a `<div role="button">` does not.*

**Optional course resources:**

- FrontendMasters — *Website Accessibility, v2* (Jon Kuperman) — the section on interactive elements.

**Search prompts:**

- `"div vs button accessibility keyboard"`
- `"when to use link vs button"`
- Ask an AI: *"List everything a native HTML `<button>` does automatically that I would have to manually recreate if I used a `<div onclick>` instead. Include keyboard, focus, screen reader, and form behavior."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.2 — links, buttons, and the difference.
Completed so far: Module 1.1.

Build me a small single-page interface (one HTML file, a little inline JS is fine) that
looks like a normal app toolbar and content area — something with navigation items, action
controls (save, delete, open a menu), and a couple of links to other pages and to an anchor
on the same page. Make it LOOK fine in a browser.

But construct the interactive elements WRONG in several realistic ways — the kind of wrong
that real developers ship when they style things by eye and reach for whatever element is
convenient. The page should appear to work for a mouse user but be broken in ways that only
reveal themselves under a keyboard, a screen reader, or specific user actions (like opening
a link in a new tab, or what happens on Enter vs Space).

Tell me only how to run it. Do not tell me what is wrong or how many problems there are.
I will navigate using only the Tab and Enter and Space keys first, then a mouse, then a
screen reader, and log every broken behavior I find. Make me explain, for each one, what the
correct element is and WHY — what the platform would have given me for free.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Typical defects this module plants (the AI chooses which, you should find them by behavior):

- A `<div onclick>` or `<span onclick>` used as a button — not focusable via Tab, does not respond to Enter/Space, invisible to screen readers as an actionable control.
- An `<a href="#">` used to trigger a JavaScript action instead of navigating — pollutes history, breaks "open in new tab," wrong semantics announced.
- A real `<button>` inside a `<form>` with no `type` attribute (defaults to `type="submit"`) that unexpectedly submits/reloads the page when clicked.
- A navigation item that is a styled `<button>` when it should be an `<a>` (it goes to another page, so it is navigation, not an action).
- A link styled to look like a button (fine) but missing a discernible accessible name (e.g. an icon-only link with no text or `aria-label`).
- A clickable card where the whole `<div>` has an onclick but no keyboard path in at all.

The principle: **`<a>` is for going somewhere; `<button>` is for doing something.** Native elements come with focusability, keyboard activation, correct roles, and form integration baked in. Every time you rebuild that on a `<div>`, you will rebuild it incompletely.

</details>

---

## Module 1.3 — Heading Hierarchy & The Document Outline

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Heading elements** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements — *Focus on: the "Usage notes" and "Accessibility concerns" sections. Why heading levels must not be chosen for visual size.*
- **W3C WAI — Headings** — https://www.w3.org/WAI/tutorials/page-structure/headings/ — *Focus on: how screen reader users navigate by heading, and why a broken hierarchy strands them.*
- **MDN — `<hgroup>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/hgroup — *Focus on: the correct way to pair a heading with a subheading/tagline.*

**Search prompts:**

- `"why heading levels matter screen reader navigation"`
- `"skipping heading levels h1 to h3 accessibility"`
- Ask an AI: *"How do screen reader users navigate a page by headings? What exactly breaks for them when a page jumps from an h2 to an h4, or has three h1s? Walk me through their experience."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.3 — heading hierarchy and the document
outline. Completed so far: Modules 1.1–1.2.

Build me a content-rich page (one HTML file, minimal CSS allowed just so it is readable)
that is a documentation article or a long blog post — lots of titled sections and
subsections. Make the headings WRONG in realistic ways: the kind of mistakes developers
make when they pick heading tags based on how big they want the text to look, rather than
on the document's logical structure.

The page should look visually plausible. The damage is structural and invisible until you
navigate by headings. Tell me how to open it, and tell me to install a "headings outline"
browser extension OR to run document.querySelectorAll on the headings to print the outline.

Do not tell me what is wrong. I will produce the heading outline, decide whether it tells a
coherent story of the document, and fix it. Make me explain what a correct outline looks
like and why visual size must never drive heading choice.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Skipped levels: an `<h2>` followed directly by an `<h4>` because the author wanted smaller text.
- Multiple `<h1>` elements scattered through the body for visual emphasis.
- A heading used for a non-heading purpose (e.g. an `<h3>` wrapping a styled label or a price).
- Real headings demoted to styled `<p class="heading">` so they vanish from the outline entirely.
- A subtitle/tagline marked up as its own `<h2>` instead of being grouped with the title (should be inside `<hgroup>` or be a `<p>`).
- The page's visible title not being the `<h1>` (e.g. the site logo is the h1 and the article title is an h2).

The principle: **headings are the table of contents the browser builds from your page.** Screen reader users jump heading-to-heading the way you skim with your eyes. Choose levels by logical depth, style size with CSS, and the outline will be a faithful map of your content.

</details>

---

## Module 1.4 — Lists, Definition Lists, And Choosing The Right One

**Difficulty:** ●●○○○ · **Format:** Build challenge · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<ul>`, `<ol>`, `<li>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul — *Focus on: ordered vs unordered semantics, and that order MEANING (not visual numbering) decides which to use.*
- **MDN — `<dl>`, `<dt>`, `<dd>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dl — *Focus on: the description list, its name-value pairing model, and its legitimate uses (metadata, glossaries, key-value displays).*
- **W3C WAI — Lists** — https://www.w3.org/WAI/tutorials/page-structure/content/#lists — *Focus on: how assistive technology announces lists ("list, 5 items") and why that count matters.*

**Search prompts:**

- `"when to use description list dl dt dd"`
- `"ordered vs unordered list semantics"`
- Ask an AI: *"Give me five real UI patterns (navigation, breadcrumbs, a definition glossary, a stats panel, a step-by-step recipe) and tell me which list element each should use and why."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.4 — lists and definition lists, build
challenge. Completed so far: Modules 1.1–1.3.

Give me a content brief containing several different list-like structures: a primary
navigation menu, a set of breadcrumbs, a recipe's ordered steps, a glossary of terms with
definitions, a product's spec sheet (key-value pairs), and a tag cloud. Describe them in
prose only — give me no HTML.

My job: mark each one up using the correct list element (<ul>, <ol>, or <dl>) — or argue
that it should not be a list at all. Zero CSS needed beyond what makes it legible.

For each structure, before you accept it, ask me what assistive technology will ANNOUNCE
when it reaches that list, and make me justify why I chose that element over the others.
Push hard on the spec sheet and the glossary — those are where the description list earns
its place and where most developers wrongly reach for a <ul> or a <table>.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Navigation and breadcrumbs should be a `<ul>`/`<ol>` of links inside a `<nav>` (breadcrumbs are arguably ordered).
- The recipe steps are an `<ol>` — order is meaningful, reordering changes the recipe.
- The glossary and the spec sheet are the `<dl>` cases — name/value pairs where `<dt>` is the term and `<dd>` is the description. Most people wrongly use a table or nested `<ul>`.
- A tag cloud is a `<ul>` of links; the visual weighting is pure CSS.
- Watch for using a `<ul>` purely to get the bullet styling on something that is not a list, or stripping list semantics with `list-style: none` and forgetting that the list role still matters.

The principle: **a list element tells assistive tech "these items belong together, and here is how many."** Choose by the relationship between the items — sequence, set membership, or name-value pairing — not by the bullets you want.

</details>

---

## Module 1.5 — Tables For Tabular Data Done Right

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<table>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table — *Focus on: the full anatomy — `<caption>`, `<thead>`, `<tbody>`, `<tfoot>`, `<th>`, `<td>`.*
- **MDN — `<th>` and the `scope` attribute** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/th — *Focus on: `scope="col"` vs `scope="row"` and why it is essential for screen reader users to know which header a cell belongs to.*
- **W3C WAI — Tables tutorial** — https://www.w3.org/WAI/tutorials/tables/ — *Read the whole thing. Focus on: simple tables, tables with two header levels, and the `headers`/`id` association for complex tables.*

**Search prompts:**

- `"accessible data table scope headers id"`
- `"table caption thead tbody when required"`
- Ask an AI: *"How does a screen reader user read a data table cell by cell? What does the `scope` attribute change about what they hear? Show me a before/after."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.5 — accessible data tables. Completed
so far: Modules 1.1–1.4.

Build me a page with a genuine data table — something like a pricing comparison, a sports
standings table, or a financial summary with row headers AND column headers. Make it look
acceptable visually. Then break the table's SEMANTICS in several realistic ways — the kind
of mistakes developers make when they think of a table as a visual grid rather than a
structured dataset.

The table should display fine to a sighted mouse user. The damage is in how the data is
(not) associated for assistive technology and in misuse of table structure. Tell me how to
run it and suggest I explore it with a screen reader's table-navigation commands.

Do not name the problems. I will inspect the markup and navigate the table with assistive
tech, then fix it so that any cell, read in isolation, still tells you which row and column
it belongs to. Make me explain how header association actually works.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- `<th>` elements with no `scope` attribute, so cells are not associated with their headers.
- Header cells marked up as `<td>` (styled bold) instead of `<th>`, so they are not headers at all to assistive tech.
- A table with both row and column headers but no `scope` (or wrong scope), leaving cell associations ambiguous.
- Missing `<caption>` — no accessible name/summary for the table.
- No `<thead>`/`<tbody>` separation.
- Layout abuse: using a `<table>` for visual layout of non-tabular content (or the reverse — a "table" of `<div>`s with `display: table` for real tabular data, losing all semantics).
- A complex table (merged cells / two header levels) that needs `headers`/`id` association but only has `scope`.

The principle: **a table is a coordinate system of data, and every data cell must be locatable by its headers.** `scope` (for simple tables) and `headers`/`id` (for complex ones) are how the browser tells assistive tech "this number is Q3 revenue for the EMEA region."

</details>

---

## Module 1.6 — Input Types & What The Platform Gives You Free

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<input>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input — *Skim every `type`. Focus on: `email`, `tel`, `url`, `number`, `date`, `search`, `password`. Each type changes the keyboard on mobile, the validation, and sometimes the UI.*
- **web.dev — Learn Forms: input types** — https://web.dev/learn/forms/ — *Focus on: how the right input type improves mobile keyboards, autofill, and built-in validation.*
- **MDN — `inputmode` and `autocomplete`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete — *Focus on: the autocomplete tokens (`email`, `name`, `street-address`, `one-time-code`) that let browsers autofill correctly.*

**Optional course resources:**

- FrontendMasters — search for a forms-focused workshop in your library; the input-types segment.

**Search prompts:**

- `"html input types mobile keyboard autocomplete"`
- `"input type number vs text inputmode"`
- Ask an AI: *"For each of these fields — email, phone, zip code, credit card, search box, birthday, one-time passcode — what is the correct input type, inputmode, and autocomplete value, and what does each one change for the user?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.6 — input types and platform-provided
form behavior. Completed so far: Modules 1.1–1.5.

Build me a realistic signup or checkout form (one HTML file, zero JavaScript) with a wide
variety of fields: name, email, phone, password, a numeric quantity, a date, a website URL,
a search box, a postal code, and a one-time code. Make every field use a generic
type="text" or otherwise use the WRONG input type / missing attributes — the way a
developer does when they treat all inputs as interchangeable text boxes.

The form should look normal. The losses are invisible on desktop with a keyboard: wrong
mobile keyboards, no autofill, no built-in validation, no semantic meaning. Tell me to test
it by imagining (or actually using) a phone, and by trying to submit bad data.

Do not tell me what is suboptimal. I will go field by field, identify the correct type and
attributes, and explain what each correct choice gives the user that I am currently throwing
away. Reject any answer where I cannot say what the BROWSER does differently.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Email field as `type="text"` — loses the `@`-aware mobile keyboard and built-in email validation.
- Phone as `type="text"` — should be `type="tel"` for the numeric phone keypad.
- Numeric quantity as `type="text"` — should be `type="number"` with `inputmode="numeric"`.
- Postal/OTP code as `type="text"` with no `inputmode="numeric"` and (for OTP) no `autocomplete="one-time-code"`.
- Date as a text field instead of `type="date"`.
- URL field as text instead of `type="url"`.
- Search box as text instead of `type="search"`.
- Missing `autocomplete` tokens everywhere, so browser autofill cannot help.
- Password field as `type="text"` (visible) or missing `autocomplete="current-password"`/`"new-password"`.

The principle: **the input type is a contract with the browser about what kind of data this is.** Honor it and you get the right keyboard, autofill, validation, and assistive-tech semantics for free. Ignore it and you reimplement all of that worse.

</details>

---

## Module 1.7 — Form Validation Without JavaScript

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML (+ tiny CSS for `:invalid`)

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Client-side form validation** — https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation — *Focus on: `required`, `minlength`, `maxlength`, `min`, `max`, `pattern`, `step`, and the `:valid`/`:invalid` pseudo-classes.*
- **MDN — Constraint Validation API** — https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation — *Focus on: how the browser validates before submit, and `setCustomValidity`. (You will NOT use JS here, but understand what the platform is doing.)*
- **web.dev — Learn Forms: validation** — https://web.dev/learn/forms/validation/ — *Focus on: built-in validation vs custom, and the order the browser checks constraints.*

**Search prompts:**

- `"html5 form validation required pattern minlength"`
- `"css invalid valid pseudo class styling"`
- Ask an AI: *"How does HTML5 constraint validation work without any JavaScript? Walk me through what happens when a user submits a form with a `required` empty field and a `pattern` mismatch — what does the browser do and in what order?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.7 — native constraint validation, no
JavaScript. Completed so far: Modules 1.1–1.6.

Build me a registration form (one HTML file). You may add a little CSS to style validity
states, but ZERO JavaScript. The form has rules: some fields are mandatory, a username has
a length range and an allowed-character pattern, a password has a minimum length, an age
field has a numeric range, two fields must match a format, etc. Describe these rules to me
as the product requirements.

Then build the form so it FAILS to enforce these rules — it relies on validation attributes
that are missing, wrong, or contradictory, so the browser lets bad data through or blocks
good data. The form should look complete. Tell me only how to run it.

I will try to submit valid and invalid data, discover where the native validation is broken,
and fix it using only HTML attributes (and CSS for feedback). If I try to reach for
JavaScript, refuse — make me find the platform attribute that already does the job. Make me
explain what the browser checks and when.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- A "required" field missing the `required` attribute (so it submits empty).
- A `pattern` regex that is subtly wrong (e.g. not anchored, or rejecting valid input).
- `minlength`/`maxlength` missing or swapped, or set on a `type="number"` (where `min`/`max`/`step` apply instead).
- `type="number"` with no `min`/`max`/`step`, so out-of-range or decimal values pass.
- A `pattern` on a field whose `type` already validates differently, causing a conflict.
- Relying on `placeholder` as if it were a constraint (it does nothing).
- A field that should match another (confirm email/password) — note this genuinely needs JS or a server; a good answer recognizes the limit of pure HTML here.
- Missing the `novalidate`-free baseline, or a `formnovalidate` button silently disabling validation.

The principle: **the browser has a full validation engine built in.** `required`, `pattern`, `min`/`max`/`step`, `minlength`/`maxlength` declare your constraints and the browser enforces them, styles them via `:invalid`, and blocks submission — no script required. Reach for JS only for cross-field rules the platform genuinely cannot express.

</details>

---

## Module 1.8 — Labels, Fieldsets, And Form Association

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<label>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label — *Focus on: explicit association via `for`/`id` vs implicit wrapping, and why clicking a label focuses its control.*
- **MDN — `<fieldset>` and `<legend>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/fieldset — *Focus on: grouping related controls (especially radio groups) and how `<legend>` is announced.*
- **W3C WAI — Labeling controls** — https://www.w3.org/WAI/tutorials/forms/labels/ — *Focus on: visible labels, hidden-but-present labels, and grouping.*

**Search prompts:**

- `"label for id vs wrapping input accessibility"`
- `"fieldset legend radio group screen reader"`
- Ask an AI: *"Why must every form control have an associated label? What exactly does a screen reader announce for an input with a `for`/`id` label vs one with only a placeholder vs one wrapped in a label? And when do I need a fieldset and legend?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.8 — labels, fieldsets, and form
association. Completed so far: Modules 1.1–1.7.

Build me a form with grouped controls: a set of radio buttons for a choice, a group of
checkboxes, and several text inputs — for example a shipping form with a "delivery speed"
radio group and a "gift options" checkbox group. Make it look organized.

Break the LABELING and GROUPING in realistic ways — the kind of mistakes that happen when
developers position labels visually with CSS rather than associating them semantically, and
when they forget that a set of radios is a single question. The form should look labeled to
a sighted user but be confusing or broken for screen reader and click-target users.

Tell me only how to run it. I will click on label text to see if it focuses the control,
navigate with a screen reader, and find every association failure. Make me fix them with
correct HTML and explain what assistive tech announces in each case.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Labels that are just `<span>`/`<p>` text near the input with no `for`/`id` link — clicking them does nothing, screen reader announces an unlabeled field.
- A `<label for="...">` whose `for` does not match any input `id` (typo or mismatch).
- Placeholder used as the only label (disappears on input, low contrast, not a real label).
- A radio-button group with no `<fieldset>`/`<legend>`, so the overall question ("Delivery speed") is never announced — only the individual options.
- Duplicate `id`s breaking label association.
- A checkbox group similarly ungrouped.
- A visually-hidden label done wrong (`display:none` removes it from the accessibility tree entirely; the correct visually-hidden technique keeps it present).

The principle: **a label is a programmatic bond between text and a control, not just text sitting nearby.** `for`/`id` (or wrapping) creates that bond — giving a bigger click target and a spoken name. `<fieldset>`/`<legend>` bonds a group of controls into one labeled question.

</details>

---

## Module 1.9 — Images: alt, figure, Decorative vs Meaningful

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<img>` and the `alt` attribute** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img — *Focus on: when `alt` should describe, when it should be empty (`alt=""`), and that a missing `alt` is different from an empty one.*
- **W3C WAI — Images tutorial** — https://www.w3.org/WAI/tutorials/images/ — *Read the decision tree. Focus on: informative, decorative, functional, and complex images, and what `alt` each one needs.*
- **MDN — `<figure>` and `<figcaption>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figure — *Focus on: when a caption is appropriate and how it differs from `alt`.*

**Search prompts:**

- `"alt text decorative vs informative images"`
- `"empty alt attribute vs missing alt"`
- Ask an AI: *"Give me the W3C decision tree for writing alt text. Then quiz me: for a company logo that links home, a decorative divider, a chart, and a product photo, what alt does each need and why?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.9 — images and alternative text.
Completed so far: Modules 1.1–1.8.

Build me a content page with many kinds of image: a logo that links to the homepage, a
purely decorative background flourish, an informative product photo, a chart/graph that
conveys data, an icon inside a button, and a photo with a caption. Make the page look fine.

Handle the alt text and figure markup WRONG across these images in realistic ways — the
mistakes developers make when they either skip alt entirely, stuff filenames into it, or
describe decorative images that should be silent. Tell me only how to run it.

I will go through every image with a screen reader (or by reading the alt values) and decide,
per image, whether the current alternative text is correct — describing meaningful images,
silencing decorative ones, and conveying the FUNCTION of functional images. Make me apply
the decision tree and justify each call.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- The logo-link with `alt="logo"` instead of describing its function (e.g. `alt="Acme, home"`).
- A decorative image with descriptive `alt` (noise) instead of `alt=""` (silenced).
- An informative product image with missing or filename `alt` (`alt="IMG_2931.jpg"`).
- A chart with `alt="chart"` instead of conveying the data, or no longer/structured description for a complex image.
- An icon-only button where the icon `<img>` has empty alt AND the button has no other accessible name → unlabeled control.
- A captioned photo where the same text is duplicated in both `alt` and `<figcaption>` (redundant) — often the `alt` should be empty or different from the caption.
- A meaningful image set as a CSS background (so it is invisible to assistive tech and unprintable) when it should be a real `<img>`.

The principle: **`alt` answers "what would a sighted user lose if this image failed to load?"** Meaningful images describe; decorative images go silent with `alt=""`; functional images (links/buttons) describe their action. A missing `alt` is a bug; an intentional empty `alt` is a decision.

</details>

---

## Module 1.10 — Native Interactive Elements: details, dialog, and friends

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML (+ minimal JS for `<dialog>`)

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `<details>` and `<summary>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details — *Focus on: the built-in disclosure widget — open/close state, keyboard support, all free.*
- **MDN — `<dialog>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog — *Focus on: `showModal()` vs `show()`, the backdrop, focus trapping, and `Escape`-to-close that you get for free.*
- **MDN — `<progress>` and `<meter>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress — *Focus on: when a native element already exists for a UI pattern you were about to build from scratch.*

**Search prompts:**

- `"html dialog element showModal focus trap"`
- `"details summary accordion accessibility"`
- Ask an AI: *"What does the native `<dialog>` element give me for free that a `<div role=dialog>` modal requires me to build manually? Cover focus trapping, the backdrop, the top layer, Escape handling, and inertness of the background."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.10 — native interactive elements, build
challenge. Completed so far: Modules 1.1–1.9.

Give me a spec for a small page that needs: an FAQ accordion (several expandable
question/answer pairs), a confirmation modal triggered by a button, and a file-upload
progress indicator. Describe the behavior as requirements. Give me no code.

My job: build all three using NATIVE HTML elements wherever one exists — not div-based
reimplementations. Minimal JS is allowed only where the platform genuinely requires it (e.g.
calling dialog.showModal()).

Review me like a senior. For each widget, if I reach for a <div> + ARIA + custom JS where a
native element exists, do not tell me the element — ask me what behaviors I am about to
hand-build (keyboard, focus, state, escape) and whether the platform already provides them.
Make me discover <details>, <dialog>, and <progress> myself, and explain what each hands me
for free.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

The "right" answers:

- The accordion should be `<details>`/`<summary>` pairs — open/close state, keyboard toggling, and correct semantics with zero JS. Most people build a `<div>` + `aria-expanded` + click handler that is worse.
- The modal should be a `<dialog>` opened with `showModal()` — you get the top layer, the `::backdrop`, automatic focus into the dialog, focus trapping, `Escape`-to-close, and background inertness. A hand-rolled `<div role="dialog">` requires you to build every one of those, usually incompletely.
- The progress indicator should be `<progress>` (or `<meter>` for a static measurement), which is announced correctly and styleable.

Watch for: forgetting that `<dialog>` needs `showModal()` (not just the `open` attribute) to get the modal behaviors; reimplementing focus trapping by hand; using `<details>` but breaking it by hijacking the click.

The principle: **before building an interactive widget, ask whether the platform already ships one.** Native elements come with keyboard, focus, and state management that took the browser vendors years to get right. Your reimplementation will not match them.

</details>

---

## Module 1.11 — The `<head>`, Metadata, And SEO Fundamentals

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — What's in the head? Metadata** — https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/The_head_metadata_in_HTML — *Focus on: `<title>`, `<meta charset>`, `<meta name="viewport">`, `<meta name="description">`, and `<link>` for stylesheets/icons.*
- **MDN — `<meta>` viewport** — https://developer.mozilla.org/en-US/docs/Web/HTML/Viewport_meta_tag — *Focus on: what the viewport meta tag does for mobile rendering and what breaks without it.*
- **Google Search Central — Title links & meta descriptions** — https://developers.google.com/search/docs/appearance/title-link — *Focus on: how the `<title>` and meta description feed the search result snippet.*

**Search prompts:**

- `"meta viewport tag mobile rendering"`
- `"title tag meta description seo best practice"`
- Ask an AI: *"What is every element that belongs in the `<head>`, and what does each one do? Explain specifically what breaks visually and for SEO if the viewport meta, charset, and title are missing."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.11 — document metadata and SEO basics.
Completed so far: Modules 1.1–1.10.

Build me a multi-section marketing page whose <head> is broken or impoverished in realistic
ways — the kind of neglect that happens when developers focus only on the <body>. The page
content is fine; the head is the problem. Symptoms should include things I can actually
observe: bad mobile rendering, a useless browser tab title, mojibake from wrong encoding,
and a page that would produce a terrible search snippet.

Tell me how to run it, and tell me to view it on a narrow viewport and to look at the tab.
Do not tell me what is wrong in the head. I will diagnose each symptom, trace it to the
missing or wrong metadata, and fix it. Make me explain what each head element does and which
consumer (browser, search engine, social card) relies on it.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Missing `<meta name="viewport" content="width=device-width, initial-scale=1">` → page renders zoomed-out and tiny on mobile.
- Missing or wrong `<meta charset="utf-8">` (or it appears too late in the head) → special characters render as mojibake.
- Empty, generic, or missing `<title>` → useless browser tab and search result heading.
- Missing `<meta name="description">` → search engine invents a poor snippet.
- `<html>` missing the `lang` attribute → wrong pronunciation by screen readers, wrong translation prompts.
- Stylesheet `<link>` or favicon `<link>` malformed.
- Charset/viewport placed after other tags that depend on them.

The principle: **the `<head>` is where you talk to the browser, search engines, and social platforms — none of whom read your `<body>` the way a human does.** The viewport tag controls mobile layout, charset controls text decoding, title and description control your search presence, and `lang` controls localization and pronunciation.

</details>

---

## Module 1.12 — Structured Data & Open Graph

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **Open Graph protocol** — https://ogp.me — *Read the whole (short) spec. Focus on: `og:title`, `og:description`, `og:image`, `og:type`, `og:url`.*
- **Schema.org — Getting started** — https://schema.org/docs/gs.html — *Focus on: what structured data is and the `Article`, `Product`, and `BreadcrumbList` types.*
- **Google Search Central — Intro to structured data** — https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data — *Focus on: JSON-LD as the recommended format and what rich results it can unlock.*

**Search prompts:**

- `"open graph tags social media preview"`
- `"json-ld structured data schema.org article"`
- Ask an AI: *"What is the difference between Open Graph meta tags and Schema.org JSON-LD structured data? Which consumers use each, and what does each unlock? Show me both for a blog article."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.12 — structured data and social cards,
build challenge. Completed so far: Modules 1.1–1.11.

Give me a spec for an article page that needs to (a) produce a rich, correct preview card
when shared on social media and (b) be eligible for a rich result in search via structured
data. Describe the article's metadata (title, author, publish date, hero image, description)
as content. Give me no code.

My job: add the correct Open Graph meta tags AND a Schema.org JSON-LD block for an Article.
Then I will validate it (you tell me which validator URLs to paste it into).

Review me strictly. If my OG tags are incomplete or my JSON-LD has the wrong type or missing
required properties, do not fix it — point me at the validator output and the spec and make
me reconcile them. Make me explain which consumer reads OG tags vs JSON-LD and why both
exist.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A complete Open Graph set: `og:title`, `og:type` (`article`), `og:url`, `og:image` (with absolute URL and ideally dimensions), `og:description`. Twitter Card tags (`twitter:card`, etc.) are a reasonable addition.
- A valid JSON-LD `Article` (or `BlogPosting`) with `headline`, `author` (as a `Person`/`Organization` object), `datePublished`, `image`, `publisher`.
- Common mistakes to catch: relative image URLs in OG (they must be absolute); JSON-LD `@type` wrong or `@context` missing; duplicating but disagreeing values between OG and JSON-LD; putting JSON-LD in the body without a `<script type="application/ld+json">` wrapper.
- Validation: the JSON-LD against Google's Rich Results Test; the OG tags against a social card debugger.

The principle: **search engines and social platforms cannot understand your prose, so you describe your content to them in machine-readable formats.** Open Graph powers share previews; JSON-LD powers search rich results. Both are metadata layered on top of your human-readable HTML.

</details>

---

## Module 1.13 — Encoding, Entities, And Internationalization Basics

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — HTML character references (entities)** — https://developer.mozilla.org/en-US/docs/Glossary/Character_reference — *Focus on: when you must escape `<`, `>`, `&`, and when named/numeric entities are needed.*
- **W3C — Character encodings: Essential concepts** — https://www.w3.org/International/articles/definitions-characters/ — *Focus on: bytes vs code points vs characters, and why declaring UTF-8 matters.*
- **MDN — `lang` and `dir` attributes** — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/lang — *Focus on: marking language for parts of a page, and `dir="rtl"` for right-to-left scripts.*

**Search prompts:**

- `"utf-8 encoding mojibake html charset"`
- `"html entities ampersand less than escaping"`
- `"lang dir attribute internationalization"`
- Ask an AI: *"Explain the difference between a byte, a Unicode code point, and a grapheme. Why does declaring `<meta charset='utf-8'>` matter, and what exactly is mojibake? When must I use HTML entities instead of raw characters?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.13 — encoding, entities, and i18n.
Completed so far: Modules 1.1–1.12.

Build me a page with international content: text in multiple languages including at least one
right-to-left script, some mathematical/currency symbols, code snippets that contain angle
brackets and ampersands, and some emoji. Introduce realistic bugs: characters that render as
garbage, entities that should be escaped but are not (breaking the markup), a snippet whose
literal < is being parsed as a tag, and language/direction that is not marked.

Tell me how to run it. I will spot the rendering and parsing failures, trace each to its
encoding or entity or i18n cause, and fix them. Make me explain what the browser is doing
when it decodes bytes into characters, and why unescaped < breaks a code sample.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Missing/wrong charset declaration → accented and non-Latin characters become mojibake.
- A code snippet containing a literal `<div>` that the browser parses as a real element (must be escaped to `&lt;div&gt;` or wrapped appropriately).
- An unescaped `&` that starts an invalid entity (e.g. `Tom & Jerry` where the parser chokes), needing `&amp;`.
- Right-to-left text not wrapped with `dir="rtl"` (or `lang`), so punctuation and ordering render wrong.
- A foreign-language passage with no `lang` attribute → screen readers mispronounce it.
- Over-escaping (using entities where raw UTF-8 characters are fine and clearer).

The principle: **the browser receives bytes and must be told how to turn them into characters (UTF-8), and must be told which characters are markup vs content (entities).** Encoding mismatches corrupt text; unescaped reserved characters corrupt structure; missing `lang`/`dir` corrupts pronunciation and layout for international users.

</details>

---

## Module 1.14 — Responsive Images: srcset, sizes, and `<picture>`

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Responsive images** — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images — *The core text. Focus on: the resolution-switching problem (`srcset` + `sizes`) vs the art-direction problem (`<picture>` + `<source>`).*
- **MDN — `<picture>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture — *Focus on: `<source>` with `media` and `type`, and how the browser picks the first match.*
- **web.dev — Serve responsive images** — https://web.dev/articles/serve-responsive-images — *Focus on: `srcset` density vs width descriptors, and how `sizes` tells the browser the layout width before CSS loads.*

**Search prompts:**

- `"srcset sizes width descriptor explained"`
- `"picture element art direction vs srcset"`
- `"responsive images avif webp fallback"`
- Ask an AI: *"Explain the difference between using `srcset`/`sizes` for resolution switching and `<picture>` for art direction. When do I need each? Walk me through how the browser chooses a source, and what the `sizes` attribute is actually telling it."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.14 — responsive images, build challenge.
Completed so far: Modules 1.1–1.13.

Give me a spec for a page with two distinct image needs: (1) a hero photo that should load
an appropriately-sized file depending on the viewport and pixel density (resolution
switching), and (2) a featured image that should be CROPPED DIFFERENTLY on mobile vs desktop
(art direction) and should prefer a modern format (AVIF/WebP) with a JPEG fallback.

Give me public image URLs at a few sizes to work with, and the layout's CSS widths. Give me
no <img>/<picture> code.

My job: implement (1) with srcset + sizes and (2) with <picture> + <source>. Then I will
verify in DevTools Network which file the browser actually downloads at different viewport
sizes and pixel ratios.

Review me strictly. If the browser downloads the wrong file, do not tell me why — ask me
what `sizes` is promising and whether it matches my real layout. Make me explain how the
browser's source selection works and why `sizes` must be accurate.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- For resolution switching: `srcset` with width descriptors (`image-480.jpg 480w, image-800.jpg 800w, ...`) plus an accurate `sizes` attribute describing the rendered width at each breakpoint. A wrong `sizes` (e.g. `100vw` when the image is actually half-width) makes the browser download a too-large file.
- For art direction: `<picture>` with `<source media="(max-width: 600px)" srcset="...">` for the cropped mobile version, and `<source type="image/avif">` / `type="image/webp"` before the `<img>` JPEG fallback.
- Common mistakes: forgetting the fallback `<img>` inside `<picture>` (required); putting `type` and `media` sources in the wrong order; using density descriptors (`2x`) when width descriptors are needed; a `sizes` value that does not match the CSS.
- Verification: in DevTools, change viewport and device pixel ratio and watch which file downloads.

The principle: **`srcset`/`sizes` solves "same image, right size for this screen"; `<picture>` solves "genuinely different image or format for this context."** `sizes` is a promise to the browser about layout width made BEFORE CSS is parsed — so the browser can pick a file early. Lie in `sizes` and it picks wrong.

</details>

---

## Module 1.15 — Landmarks & The Document's Regions

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML

### Prep — Read Before You Start

**Primary sources:**

- **MDN — ARIA landmark roles** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles#3._landmark_roles — *Focus on: how native elements (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) map to landmark roles automatically.*
- **W3C WAI — Page Regions** — https://www.w3.org/WAI/tutorials/page-structure/regions/ — *Focus on: how assistive tech users jump between landmarks, and why exactly one `<main>` matters.*
- **MDN — `<nav>` and labeling multiple navs** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/nav — *Focus on: using `aria-label` to distinguish multiple navigation regions on one page.*

**Search prompts:**

- `"aria landmarks navigation screen reader"`
- `"multiple nav elements aria-label distinguish"`
- Ask an AI: *"What are ARIA landmark regions, how do screen reader users navigate between them, and which native HTML elements create them automatically? If a page has two `<nav>` elements, how do I make them distinguishable?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 1.15 — landmark regions. Completed so far:
Modules 1.1–1.14. This module closes the HTML track, so push me harder.

Build me a full page layout — primary nav, a secondary in-page nav, a main content area, a
complementary sidebar, a search region, and a footer — but construct the regions WRONG: too
much wrapped in generic <div>s, missing or duplicated landmarks, multiple unlabeled <nav>s
that are indistinguishable, content that belongs in <main> sitting outside it, and so on.

The page looks fine visually. The damage is that an assistive-tech user cannot efficiently
jump to "the main content" or tell the two navigations apart. Tell me how to run it and
suggest I use a landmark-navigation tool or a screen reader's landmark list.

Do not name the problems. I will produce the landmark map of the page, judge whether a
stranger could navigate it by regions alone, and fix it. Make me explain what each landmark
communicates and why a single, correct <main> is non-negotiable.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

Likely defects:

- Major regions wrapped in `<div class="...">` instead of `<header>`/`<nav>`/`<main>`/`<aside>`/`<footer>`, so no landmarks exist.
- Zero or multiple `<main>` elements (there must be exactly one per page).
- Two or more `<nav>` elements with no `aria-label`, so the landmark list shows "navigation, navigation" with no way to tell them apart.
- Content that is the page's primary content sitting outside `<main>` (e.g. in a sibling div), so "skip to main" lands in the wrong place.
- A search area not marked with `role="search"` (or a `<search>` element) so it is not reachable as a landmark.
- A footer or header nested where its landmark meaning changes unexpectedly (e.g. a `<footer>` inside an `<article>` is the article's footer, not the page's).

The principle: **landmarks are the page's regions, and assistive-tech users navigate between them the way you glance around a webpage.** Native sectioning elements create them for free; multiple same-type landmarks need labels; and exactly one `<main>` marks where the unique content begins so "skip to content" works.

</details>

---

*End of Track 1 — HTML & Semantics. 15 modules.*

*Next: Track 2 — CSS & Layout, in its own file.*