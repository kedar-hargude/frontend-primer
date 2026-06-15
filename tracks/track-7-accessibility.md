# TRACK 7 — ACCESSIBILITY & SEMANTICS IN PRACTICE

*What separates pros from juniors, and an underpriced career advantage. Track 1 taught semantic HTML; this track makes you operate the way disabled users do — keyboard-only, screen-reader-driven — and build interactive components that actually work for them. Most developers cannot build an accessible combobox or modal from scratch. After this track, you can. Mix of vanilla (to learn the platform) and React + Tailwind (your real output).*

*Reminder: your root `CLAUDE.md` is loaded automatically. Do the Prep — including turning on a real screen reader — before each module. No spoilers early.*

---

## Module 7.1 — The Keyboard-Only Walk

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS/JS

### Prep — Read Before You Start

**Primary sources:**
- **WebAIM — Keyboard Accessibility** — https://webaim.org/techniques/keyboard/ — *Read fully. Focus on: tab order, focus visibility, keyboard traps, and what keys activate what.*
- **MDN — `tabindex`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex — *Focus on: `0` (in natural order), `-1` (focusable only programmatically), and why positive values are an anti-pattern.*
- **WCAG — 2.1.1 Keyboard & 2.4.7 Focus Visible** — https://www.w3.org/WAI/WCAG21/Understanding/keyboard — *Focus on: the requirement that everything be operable by keyboard and focus be visible.*

**Search prompts:**
- `"keyboard accessibility tab order focus trap"`
- `"tabindex 0 vs -1 vs positive"`
- `"focus visible outline accessibility"`
- Ask an AI: *"Teach me to audit a page with only a keyboard: what should Tab/Shift+Tab/Enter/Space/Escape/arrows do, what a keyboard trap is, why removing focus outlines is harmful, and how `tabindex` values (-1, 0, positive) change focus behavior. Give me a checklist to walk any UI."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.1 — the keyboard-only walk. Completed so far:
Tracks 1–6.

Build me a visually-decent interactive page (one HTML file, minimal CSS/JS) that is hostile to
keyboard users in several realistic ways: interactive controls that can't be reached or activated
by keyboard; a focus order that jumps around illogically; focus outlines removed for "design"
reasons so you can't see where you are; a custom widget that traps focus; and a control reachable
but not operable by Enter/Space.

Tell me only how to run it. I must navigate using ONLY the keyboard (mouse forbidden) and log
every failure in my own words: where focus went, what I couldn't reach, where I got stuck. Then I
fix them. Do not name the problems. Make me restore a logical tab order, visible focus, full
keyboard operability, and no traps — and explain what the platform gives keyboard users for free
when I use the right elements.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A `<div onclick>` "button" that isn't in the tab order and doesn't respond to Enter/Space (not keyboard-operable). Fix: a real `<button>` (or `tabindex="0"` + key handlers + role, but prefer native).
- `outline: none` (or `:focus { outline: none }`) globally, removing the focus indicator. Fix: a visible focus style (`:focus-visible`).
- A positive `tabindex` (e.g. `tabindex="5"`) forcing an illogical focus order out of sync with the visual/DOM order. Fix: rely on natural DOM order (`tabindex="0"`/none).
- A custom widget (modal/menu) that traps focus with no escape, or conversely lets focus escape when it should be contained.
- A control reachable by Tab but with no key handler, so Enter/Space do nothing.

The principle: **everything interactive must be reachable and operable by keyboard, in a logical order, with visible focus and no traps — and native elements provide all of that for free.** A keyboard walk surfaces exactly the failures that block users who can't use a mouse; most are fixed by using the right element instead of a styled div.
</details>

---

## Module 7.2 — The Screen Reader Walk

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**
- **WebAIM — Screen Reader Testing** — https://webaim.org/articles/screenreader_testing/ — *Focus on: how to actually run VoiceOver (Mac: Cmd+F5) or NVDA (Windows), and basic navigation commands.*
- **MDN — The accessibility tree** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/Accessibility_tree — *Focus on: that the screen reader consumes the accessibility tree your markup produces, not your CSS.*
- **WebAIM — Designing for Screen Reader Compatibility** — https://webaim.org/techniques/screenreader/ — *Focus on: what gets announced and what gets missed.*

**Search prompts:**
- `"voiceover nvda basic commands testing"`
- `"accessibility tree dom difference"`
- Ask an AI: *"Walk me through running a screen reader (VoiceOver on Mac or NVDA on Windows) for the first time: turning it on, navigating by element/heading/landmark, and what it announces for a link vs button vs unlabeled input vs image. What is the accessibility tree, and how is it different from the DOM?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.2 — the screen reader walk. Completed so far:
Tracks 1–6, Module 7.1.

Build me a small, visually-fine page (one HTML file, minimal CSS) that is a nightmare for screen
reader users in several distinct ways: controls announced with no name or the wrong name;
information conveyed only visually (color, position, icon) with no text equivalent; reading order
that doesn't match the visual order; an image and an icon-button with no accessible name; and
content that looks like a heading/list but isn't marked up as one (so it's invisible to
structural navigation).

Tell me only how to run it. I must navigate with a SCREEN READER (and keyboard) only, and log
exactly what it announces at each stop versus what a sighted user perceives. Then I fix the gaps.
Do not name the problems. Make me explain, for each fix, what the screen reader announced before
and after, and why the accessibility tree (not the CSS) is what it reads.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A control with no accessible name (icon-only button/link, or an input with no label) announced as "button"/"edit text" with no purpose.
- Information conveyed only by color/position/icon (e.g. a red dot meaning "error," an arrow meaning "expanded") with no text alternative — invisible to a screen reader.
- A DOM/reading order that diverges from the visual order (e.g. CSS reordering), so the screen reader narrates content in a confusing sequence.
- An image with missing/meaningless `alt`; an icon-button with no `aria-label`.
- Visual "headings"/"lists" built from styled `<div>`s/`<span>`s, so structural navigation (by heading/list) finds nothing.

The principle: **a screen reader narrates the accessibility tree your markup produces — names, roles, structure, and reading order — not your visual design.** Anything conveyed only visually, any unnamed control, and any fake-semantic structure simply doesn't exist for these users. Walking the page with a screen reader makes the gap between "looks done" and "is usable" unmistakable.
</details>

---

## Module 7.3 — Focus Management & Focus Traps

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WAI-ARIA APG — Dialog (Modal) pattern** — https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/ — *Focus on: moving focus into the dialog, trapping it while open, and returning it on close.*
- **MDN — `HTMLElement.focus()` and `:focus-visible`** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus — *Focus on: programmatic focus and `preventScroll`.*
- **MDN — `inert`** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/inert — *Focus on: making background content non-interactive when a modal is open.*

**Search prompts:**
- `"focus trap modal implementation accessibility"`
- `"return focus to trigger on close dialog"`
- `"inert attribute background modal"`
- Ask an AI: *"Explain focus management for overlays: when a modal opens, where should focus go; how do I trap Tab/Shift+Tab inside it; what does `inert` do to the background; and why must focus return to the triggering element on close? What are the common focus bugs in custom modals/menus?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.3 — focus management and focus traps.
Completed so far: Tracks 1–6, Modules 7.1–7.2.

Build me a React + Tailwind app with a custom modal and a custom dropdown menu that mismanage
focus: opening the modal leaves focus behind it (or on the trigger), so keyboard users are stuck
in the background; Tab escapes the open modal into the page behind it (no trap, background not
inert); closing the modal drops focus to the top of the page instead of returning to the trigger;
and the dropdown doesn't move focus into its items or restore it on close.

Tell me only how to run it. Test with keyboard only. Do not name the bugs. I diagnose by Tabbing
through an open overlay and noticing where focus goes. Make me implement correct focus management
— move focus in on open, trap it, make the background inert, return focus on close — and explain
why each step is required for keyboard/screen-reader users. (Prefer building it; note where native
`<dialog>` would give some of this free.)
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- On open, focus is not moved into the modal (stays on the trigger or body), so keyboard users have to Tab through the whole background to reach it.
- No focus trap: Tab/Shift+Tab from the modal's last/first focusable element escapes into the page behind it; the background isn't `inert`/`aria-hidden`, so it's still reachable.
- On close, focus is lost (drops to `<body>` top) instead of returning to the element that opened the modal — disorienting for keyboard/SR users.
- The dropdown doesn't move focus to its items (arrow-key navigation) or restore focus to the trigger on close.
- Note: native `<dialog>` + `showModal()` provides the trap, top layer, and Escape for free — a legitimate alternative.

The principle: **an accessible overlay moves focus in on open, traps focus inside while open (with the background inert), and returns focus to the trigger on close.** Without this, keyboard and screen-reader users get stranded behind or outside the overlay. Focus is a resource you must deliberately manage for any UI that appears on top of other UI.
</details>

---

## Module 7.4 — ARIA: Roles, States & Properties (And When NOT To Use It)

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WAI-ARIA APG — Read Me First / No ARIA is better than Bad ARIA** — https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/ — *Focus on: the first rule of ARIA — don't use it if a native element exists.*
- **MDN — ARIA roles / states and properties** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles — *Focus on: `aria-expanded`, `aria-selected`, `aria-current`, `aria-hidden`, `aria-label`/`aria-labelledby`/`aria-describedby`.*
- **W3C — Using ARIA (rules of ARIA use)** — https://www.w3.org/TR/using-aria/ — *Focus on: the five rules of ARIA.*

**Search prompts:**
- `"no aria is better than bad aria"`
- `"aria-label vs aria-labelledby vs aria-describedby"`
- `"aria-hidden focusable element bug"`
- Ask an AI: *"Teach me the rules of ARIA use, especially 'no ARIA is better than bad ARIA' and 'prefer native HTML.' Explain `aria-label` vs `aria-labelledby` vs `aria-describedby`, and the dangerous mistakes: redundant roles, `aria-hidden` on a focusable element, and inventing ARIA where a native element already works."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.4 — ARIA roles, states, properties, and
restraint. Completed so far: Tracks 1–6, Modules 7.1–7.3.

Build me a React + Tailwind app that MISUSES ARIA in realistic ways: redundant/incorrect roles on
native elements (e.g. `role="button"` on a `<button>`, or a wrong role that lies about an
element); `aria-hidden="true"` on an element that is still focusable (so SR users land on
"nothing"); a custom toggle/disclosure missing its state (`aria-expanded`) so SR users can't tell
if it's open; an icon button labeled with the wrong technique; and a place where someone built a
div+ARIA widget that should just be a native element.

Tell me only how to run it. Test with a screen reader. Do not name the bugs. I diagnose by what
the SR announces vs reality. Make me fix each — remove redundant/wrong ARIA, prefer native, add
the missing state attributes, label correctly — and explain "no ARIA is better than bad ARIA" and
when ARIA is genuinely required.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Redundant roles (`role="button"` on `<button>`, `role="list"` on `<ul>`) — noise or worse, a role that contradicts the element.
- `aria-hidden="true"` on an element that is still in the tab order: SR skips it but keyboard focus lands on it, announcing nothing ("focusable but hidden" trap). Fix: don't hide focusable content, or also remove it from tab order.
- A custom disclosure/accordion/toggle missing `aria-expanded` (and `aria-controls`), so its open/closed state is invisible to SR users.
- An icon button labeled wrong (e.g. visible text hidden with `display:none`, which removes it from the tree) instead of `aria-label` or a visually-hidden label.
- A div+ARIA reimplementation of something native (a fake checkbox/button/link) that should be the native element.

The principle: **ARIA augments semantics only when native HTML can't express them — and incorrect ARIA actively misleads, so "no ARIA is better than bad ARIA."** The frequent failures are lying roles, hiding focusable content, and omitting state (`aria-expanded`/`aria-selected`/`aria-current`) on custom widgets. Reach for native first; add ARIA precisely and only when needed.
</details>

---

## Module 7.5 — Accessible Forms

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **W3C WAI — Forms tutorial** — https://www.w3.org/WAI/tutorials/forms/ — *Read it fully. Focus on: labels, grouping, instructions, required fields, and error identification.*
- **WCAG — 3.3.1 Error Identification & 3.3.2 Labels or Instructions** — https://www.w3.org/WAI/WCAG21/Understanding/error-identification — *Focus on: programmatically associating errors with fields.*
- **MDN — `aria-describedby` for hints/errors** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby — *Focus on: linking a field to its hint/error text.*

**Search prompts:**
- `"accessible form error message aria-describedby"`
- `"required field aria-required vs required"`
- `"form validation error announce screen reader"`
- Ask an AI: *"Teach me accessible forms: associating labels, marking required fields, linking hint and error text to a field with `aria-describedby`, indicating invalid fields with `aria-invalid`, and announcing validation errors to screen-reader users (focus management or live regions). What does a screen reader user experience in a well-built vs broken form?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.5 — accessible forms. Completed so far:
Tracks 1–6, Modules 7.1–7.4.

Build me a React + Tailwind form (signup or checkout) with accessibility failures: fields whose
labels aren't programmatically associated; required fields indicated only with a visual asterisk;
hint text and error messages placed near a field but not LINKED to it (so SR users never hear
them); errors shown only in color; and on submit, validation errors that are not announced or
focused at all, so a SR user has no idea the form failed.

Tell me only how to run it. Test with a screen reader: fill it wrong, submit, and listen. Do not
name the bugs. I diagnose by what the SR conveys about each field and the errors. Make me fix
label association, required indication, hint/error linkage via `aria-describedby`, `aria-invalid`,
and error announcement/focus on submit — and explain what the SR user experiences before and
after.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Labels not associated (`for`/`id` mismatch, or label-as-plain-text), so fields are announced without a name.
- Required fields shown only with a visual `*` — no `required`/`aria-required`, so SR users don't know they're mandatory.
- Hint and error text sitting visually near the field but not linked via `aria-describedby`, so the SR never reads them when the field is focused.
- Errors conveyed by color/border only (no text, no `aria-invalid`).
- On submit with errors: no announcement and no focus moved to the first error — a SR user hears nothing and assumes success. Fix: move focus to the first invalid field (or summarize errors in a focused/`aria-live` region) and set `aria-invalid` + `aria-describedby` to the error text.

The principle: **an accessible form names every field, marks required ones programmatically, links hints and errors to their fields, indicates invalidity, and actively tells the user when validation fails.** A form that "shows" errors visually but doesn't associate or announce them leaves screen-reader users unable to complete it.
</details>

---

## Module 7.6 — Live Regions & Dynamic Announcements

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **MDN — ARIA live regions** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions — *Focus on: `aria-live` (polite/assertive), `aria-atomic`, `role="status"`/`role="alert"`, and the rule that the region must exist before content changes.*
- **WAI-ARIA APG — Alert pattern** — https://www.w3.org/WAI/ARIA/apg/patterns/alert/ — *Focus on: announcing dynamic, important messages.*
- **WCAG — 4.1.3 Status Messages** — https://www.w3.org/WAI/WCAG21/Understanding/status-messages — *Focus on: programmatically conveying status changes without moving focus.*

**Search prompts:**
- `"aria-live polite assertive role status alert"`
- `"live region not announcing dynamic content"`
- Ask an AI: *"Explain ARIA live regions: `aria-live='polite'` vs `'assertive'`, `role='status'` vs `role='alert'`, and `aria-atomic`. Why must the live region exist in the DOM BEFORE its content updates? What dynamic UI changes (search results count, toast notifications, form save status, cart updates) need announcing, and what are the common reasons announcements silently fail?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.6 — live regions and dynamic announcements.
Completed so far: Tracks 1–6, Modules 7.1–7.5.

Build me a React + Tailwind app with dynamic updates that are SILENT to screen readers: a
search/filter that updates a results count with no announcement; a toast/notification that
appears visually but is never announced; a "saved" status after an async action with no SR
feedback; and a buggy attempt where a live region is created at the same moment its content is set
(so nothing is announced because the region didn't pre-exist), plus an `assertive` region used for
trivial updates (rudely interrupting).

Tell me only how to run it. Test with a screen reader and listen for what changes are announced.
Do not name the bugs. I diagnose by acting and listening. Make me add correct live regions
(polite vs assertive, status vs alert), ensure they exist before updates, and explain why each
dynamic change needs announcing and why politeness level matters.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Dynamic content (results count, toast, "saved" status) updated with no live region, so SR users never learn it changed.
- A live region inserted into the DOM at the same instant its text is set — many SRs only announce changes to regions that already existed, so nothing is announced. Fix: render the (empty) live region up front; update its text later.
- An `aria-live="assertive"` / `role="alert"` used for non-urgent updates, rudely interrupting the user. Fix: `polite`/`role="status"` for routine updates; reserve assertive for genuinely urgent ones.
- Missing `aria-atomic` where the whole message should be re-read, or over-chatty regions announcing on every keystroke.

The principle: **screen readers don't notice visual changes; they announce updates to live regions — but only if the region pre-exists and is configured with the right politeness.** `polite`/`status` for routine updates, `assertive`/`alert` for urgent ones, and the region must be present before its content changes. Silent dynamic UI is invisible to SR users.
</details>

---

## Module 7.7 — Accessible Modal Dialog (Build From Scratch)

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WAI-ARIA APG — Dialog (Modal) pattern** — https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/ — *Read fully, including the keyboard interaction table and the example.*
- **MDN — `<dialog>`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog — *Focus on: `showModal()`, the top layer, `::backdrop`, and the behaviors it gives free.*
- **MDN — `inert`** — https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/inert — *Focus on: making the background non-interactive.*

**Search prompts:**
- `"accessible modal dialog aria pattern from scratch"`
- `"html dialog showModal vs custom modal accessibility"`
- Ask an AI: *"Walk me through every requirement for an accessible modal per the ARIA APG: role and labeling (`aria-modal`, `aria-labelledby`), focus moved in on open, focus trapped, Escape to close, focus returned on close, and the background made inert. Then contrast building it by hand vs using native `<dialog>` + `showModal()` — what does native give for free?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.7 — accessible modal from scratch, build
challenge. Completed so far: Tracks 1–6, Modules 7.1–7.6.

Give me a spec to build a fully accessible modal dialog in React + Tailwind meeting the ARIA APG
dialog pattern: correct role and accessible name, focus moved in on open, focus trapped while
open, Escape closes, focus returns to the trigger on close, background inert, and a screen reader
announces it as a dialog. Build TWO versions: one hand-rolled with a `<div role="dialog">`, and
one using native `<dialog>` + `showModal()` — then compare what each required. Give me no solution
code.

Review me strictly against the APG keyboard/focus table. If my trap leaks, Escape doesn't work, or
focus doesn't return, do not fix it — point me to the specific APG requirement I missed and make
me test it by keyboard and screen reader. Make me articulate exactly what native `<dialog>`
provided that I had to build by hand.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- `role="dialog"` + `aria-modal="true"`, an accessible name via `aria-labelledby` (pointing to the title), focus moved to the dialog (or its first focusable element) on open, a focus trap on Tab/Shift+Tab, Escape to close, focus returned to the triggering element on close, and the background made `inert`/`aria-hidden`.
- The native version: `<dialog>` + `showModal()` provides the top layer, `::backdrop`, automatic focus-in, focus trapping, Escape, and background inertness — far less code.
- A clear articulation of the gap: the hand-rolled version must manually implement the trap, inertness, escape, and focus return that native gives free.
- Common mistakes: leaky focus trap; Escape not wired; focus not returned (or returned to the wrong element); background still interactive; missing accessible name; not testing with an actual screen reader.

The principle: **a correct modal is a precise checklist — name, role, focus in/trap/return, Escape, inert background — and native `<dialog>` implements most of it for you.** Building both versions makes the value of native elements concrete: the platform encodes years of accessibility work you'd otherwise reimplement (usually incompletely).
</details>

---

## Module 7.8 — Accessible Combobox / Autocomplete (Build From Scratch)

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WAI-ARIA APG — Combobox pattern** — https://www.w3.org/WAI/ARIA/apg/patterns/combobox/ — *Read fully. Study the keyboard interaction table; this is the spec you implement.*
- **WAI-ARIA APG — Combobox with list autocomplete example** — https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-autocomplete-list/ — *Study the ARIA attributes and key handlers.*
- **MDN — `aria-activedescendant`** — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-activedescendant — *Focus on: virtual focus — why the input keeps DOM focus while a highlighted option is tracked by id.*

**Search prompts:**
- `"aria combobox pattern aria-activedescendant"`
- `"accessible autocomplete listbox keyboard"`
- Ask an AI: *"Walk me through building an accessible combobox per the ARIA APG: the roles (`combobox`, `listbox`, `option`), `aria-expanded`, `aria-controls`, `aria-activedescendant`, the full keyboard interaction (Down/Up/Enter/Escape/Home/End/typing), and announcing the result count. Why use `aria-activedescendant` (virtual focus) instead of moving real DOM focus to each option?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.8 — accessible combobox from scratch, build
challenge. Completed so far: Tracks 1–6, Modules 7.1–7.7. This is the accessibility capstone —
make it genuinely hard.

Give me a starter: a styled but ARIA-less and behavior-less combobox in React + Tailwind (an
input, a toggle, an empty listbox, a country dataset), with the full APG keyboard interaction
table pasted in a comment for reference. My job: implement complete ARIA (combobox/listbox/option
roles, `aria-expanded`, `aria-controls`, `aria-activedescendant`, `aria-selected`) and the FULL
keyboard interaction (Down/Up open + move, Enter select, Escape close, Home/End, typing to
filter), plus a live-region result count. Give me no solution code.

Review me strictly against the APG table. Before coding, make me read the keyboard table aloud and
list every ARIA attribute the input needs. When something fails, point me to the specific APG row.
Make me test with a real screen reader and explain why `aria-activedescendant` (virtual focus) is
used instead of moving DOM focus.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Roles/attributes: input has `role="combobox"`, `aria-expanded`, `aria-controls` (the listbox id), `aria-autocomplete`, and `aria-activedescendant` (id of the highlighted option); the listbox has `role="listbox"` and an accessible name; each option has `role="option"`, a unique id, and `aria-selected`.
- Keyboard per APG: Down/Up open the list and move the active option; Enter selects; Escape closes (and/or clears); Home/End jump; typing filters; Tab closes and commits.
- Virtual focus: DOM focus stays on the input; the "highlighted" option is tracked via `aria-activedescendant` (not real `.focus()`), so the user keeps typing while navigating options.
- A live region announcing "N results available" when the list opens/filters.
- Common mistakes: moving real focus to options (breaks typing and the SR model); missing `aria-expanded`/`aria-activedescendant`; incomplete keyboard table; no result-count announcement; `aria-selected` not maintained; not testing with a screen reader.

The principle: **the combobox is the hardest common widget because correctness means implementing the full APG ARIA + keyboard contract exactly — and `aria-activedescendant` (virtual focus) is the key idea: the input keeps focus for typing while a separate attribute tracks the highlighted option.** Building it to spec is the strongest proof you understand accessible interactive components from the platform up.
</details>

---

## Module 7.9 — Color Contrast & Visual Accessibility

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WCAG — 1.4.3 Contrast (Minimum) & 1.4.11 Non-text Contrast** — https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum — *Focus on: the 4.5:1 (normal text) and 3:1 (large text / UI components) ratios.*
- **WebAIM — Contrast Checker** — https://webaim.org/resources/contrastchecker/ — *Use it to test color pairs.*
- **WCAG — 1.4.1 Use of Color** — https://www.w3.org/WAI/WCAG21/Understanding/use-of-color — *Focus on: never conveying meaning by color alone.*

**Search prompts:**
- `"wcag contrast ratio 4.5 3 large text"`
- `"color blindness simulate ui testing"`
- `"information by color alone accessibility"`
- Ask an AI: *"Explain WCAG contrast requirements (4.5:1 normal text, 3:1 large text and UI components) and the 'don't rely on color alone' rule. How do I test contrast and simulate color-blindness? Give me examples of UI that fails for low-vision and color-blind users and how to fix each without redesigning."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.9 — color contrast and visual accessibility.
Completed so far: Tracks 1–6, Modules 7.1–7.8.

Build me a React + Tailwind UI (a dashboard or form) with realistic visual-accessibility failures:
body text and buttons below the contrast minimums; a focus indicator with insufficient contrast;
status conveyed ONLY by color (green = ok, red = error) with no icon/text; placeholder text used as
the only label at low contrast; and a chart/legend distinguishable only by hue (fails for
color-blind users).

Tell me only how to run it and how to check contrast (WebAIM checker, DevTools contrast tools) and
simulate color-blindness (DevTools rendering emulation). Do not name the failures. I diagnose by
measuring contrast and viewing under color-blind simulation. Make me fix contrast to meet 4.5:1 /
3:1, add non-color indicators (icon/text/pattern), and explain why color-alone and low contrast
exclude real users.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Body text below 4.5:1 and/or buttons/large text below 3:1 contrast against their backgrounds.
- A focus indicator (or its color) below the 3:1 non-text contrast minimum, so low-vision keyboard users can't see where they are.
- Status/meaning conveyed only by color (green/red dots or text) — invisible to color-blind users. Fix: add an icon, text label, or shape in addition to color.
- Placeholder-as-label at low contrast (fails both labeling and contrast).
- A chart/legend distinguished only by hue — fix with patterns, labels, or distinct shapes.

The principle: **low-vision and color-blind users need sufficient contrast (4.5:1 text, 3:1 large text/UI) and information that is never carried by color alone.** These are measurable, fixable failures — checked with a contrast tool and color-blindness simulation — that silently exclude a large fraction of users while the UI "looks fine" to the developer.
</details>

---

## Module 7.10 — Reduced Motion & User Preferences

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **MDN — `prefers-reduced-motion`** — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion — *Focus on: detecting the preference in CSS and JS and providing a reduced experience.*
- **WCAG — 2.3.3 Animation from Interactions & 2.2.2 Pause, Stop, Hide** — https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions — *Focus on: motion that can trigger vestibular disorders, and user control over animation.*
- **web.dev — prefers-reduced-motion** — https://web.dev/articles/prefers-reduced-motion — *Focus on: practical patterns for honoring the preference.*

**Search prompts:**
- `"prefers-reduced-motion css media query"`
- `"vestibular disorder animation accessibility"`
- Ask an AI: *"Explain `prefers-reduced-motion`: why large parallax/zoom/slide animations can cause real harm (vestibular disorders) and how to honor the user's OS preference in both CSS and JS. What's the difference between removing animation entirely vs replacing it with a subtle fade? Which motion is safe to keep?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.10 — reduced motion and user preferences,
build challenge. Completed so far: Tracks 1–6, Modules 7.1–7.9.

Give me a spec for a page rich in motion: page-transition slides, a parallax hero, animated
scroll-reveals, an auto-playing carousel, and a loading spinner. Requirement: it must fully honor
`prefers-reduced-motion` — disabling or replacing the potentially harmful motion with safe
alternatives (e.g. instant or gentle fades) while keeping essential feedback — detected in BOTH
CSS and JS (for JS-driven animations). Give me no solution code.

Review me strictly. If I disable ALL motion (including a harmless loading indicator) or, worse,
honor the preference in CSS but let a JS animation run anyway, do not fix it — ask me which motion
is actually harmful and where my JS reads the preference. Make me distinguish dangerous motion from
safe feedback and explain why this preference exists.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- CSS honoring `@media (prefers-reduced-motion: reduce)` to remove/replace large transforms (parallax, big slides/zoom) with instant changes or subtle fades.
- JS-driven animations also reading the preference via `window.matchMedia('(prefers-reduced-motion: reduce)')` (and listening for changes), since CSS-only handling misses JS animation.
- Keeping essential, non-vestibular feedback (a small loading spinner, focus transitions) while removing the harmful large-motion effects — not nuking all motion indiscriminately.
- An auto-playing carousel given a pause/stop control (2.2.2) regardless of preference.
- Common mistakes: honoring the preference in CSS but letting a JS/`requestAnimationFrame` animation run; removing all motion including harmless feedback; not listening for preference changes at runtime.

The principle: **large motion (parallax, zoom, big slides) can cause real physical harm to users with vestibular disorders, so `prefers-reduced-motion` must be honored in both CSS and JS — replacing dangerous motion with safe alternatives while preserving essential feedback.** Respecting user preferences is a core accessibility responsibility, not a nice-to-have.
</details>

---

## Module 7.11 — Accessible Data Tables (Complex)

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **W3C WAI — Tables tutorial** — https://www.w3.org/WAI/tutorials/tables/ — *Focus on: multi-level headers, the `headers`/`id` association, and `scope` for irregular tables.*
- **MDN — `<caption>`, `scope`, `headers`** — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/th — *Focus on: associating data cells with the right headers.*
- **WAI-ARIA APG — Grid pattern (for interactive tables)** — https://www.w3.org/WAI/ARIA/apg/patterns/grid/ — *Focus on: when a sortable/interactive table needs grid semantics and keyboard support.*

**Search prompts:**
- `"complex table headers id association accessibility"`
- `"sortable table aria-sort accessibility"`
- Ask an AI: *"Explain accessible complex tables: when `scope` is enough vs when you need explicit `headers`/`id` association (merged cells, multi-level headers). How do I make a sortable table accessible (`aria-sort`, announcing sort changes)? What does a screen reader user experience navigating a well-vs-poorly-built data table?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.11 — accessible complex data tables.
Completed so far: Tracks 1–6, Modules 7.1–7.10.

Build me a React + Tailwind app with a genuinely complex, interactive data table (multi-level or
merged headers, both row and column headers, and sortable columns) whose accessibility is broken:
header cells as styled `<td>`s; missing `scope` (or `scope` insufficient for the merged-header
case, where `headers`/`id` is required); no `<caption>`; and sorting that changes the data with no
`aria-sort` and no announcement, so SR users can't tell the table was re-sorted.

Tell me only how to run it. Navigate it with a screen reader's table commands. Do not name the
bugs. I diagnose by reading random cells in isolation and asking whether I can tell their row+column
context, and by sorting and listening. Make me fix the header semantics (scope vs headers/id),
caption, and sortable-column accessibility (`aria-sort` + announcement), and explain header
association.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Header cells as `<td>` (styled bold) instead of `<th>`, so they aren't headers to assistive tech.
- Missing `scope` on `<th>`, or `scope` used where the table's merged/multi-level headers actually require explicit `headers`/`id` association — so cells aren't tied to the right headers and read without context.
- No `<caption>` (no accessible name/summary for the table).
- Sortable columns that reorder data with no `aria-sort` state on the header and no announcement, leaving SR users unaware the order changed (and unsure of current sort).
- Possibly an interactive table that needs grid keyboard semantics but has none.

The principle: **in a complex table, every data cell must be programmatically associated with its row and column headers — `scope` for regular tables, explicit `headers`/`id` for merged/multi-level ones — and interactive features like sorting must expose state (`aria-sort`) and announce changes.** A cell read in isolation should still tell you what it means; sorting should be perceivable to non-visual users.
</details>

---

## Module 7.12 — Skip Links, Landmarks & Page-Level Navigation

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** React + Tailwind

### Prep — Read Before You Start

**Primary sources:**
- **WebAIM — Skip Navigation Links** — https://webaim.org/techniques/skipnav/ — *Focus on: the skip-link technique and the visible-on-focus pattern.*
- **W3C WAI — Page Regions (landmarks)** — https://www.w3.org/WAI/tutorials/page-structure/regions/ — *Focus on: landmark navigation and labeling multiple same-type landmarks.*
- **WCAG — 2.4.1 Bypass Blocks** — https://www.w3.org/WAI/WCAG21/Understanding/bypass-blocks — *Focus on: letting users skip repeated content.*

**Search prompts:**
- `"skip to main content link visible on focus"`
- `"landmark navigation multiple nav aria-label"`
- Ask an AI: *"Explain skip links and landmark navigation: why keyboard/SR users need a 'skip to main content' link, the correct visible-on-focus implementation, and how landmarks (`main`, `nav`, `banner`, `contentinfo`, `complementary`, `search`) let SR users jump between regions. How do I make multiple navs distinguishable, and why must there be exactly one `main`?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 7.12 — skip links and landmarks. Completed so
far: Tracks 1–6, Modules 7.1–7.11. This closes the accessibility track.

Build me a React + Tailwind page with a big repeated header/nav and landmark problems: no
skip-to-content link (so keyboard users must Tab through the whole nav on every page); a skip link
that exists but is hidden the wrong way (never becomes visible/focusable); major regions in plain
`<div>`s (no landmarks); two or more `<nav>`s with no labels (indistinguishable in the landmark
list); content that belongs in `main` sitting outside it; and possibly more than one `main`.

Tell me only how to run it. Test with keyboard (Tab from the top) and a screen reader's landmark
navigation. Do not name the bugs. I diagnose by trying to jump to main content and to enumerate
regions. Make me add a correct skip link (visible on focus), proper landmarks, labels for multiple
navs, exactly one `main`, and explain how SR/keyboard users navigate by region.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- No skip link, so keyboard users Tab through the entire repeated nav before reaching content (fails Bypass Blocks). Fix: a "Skip to main content" link as the first focusable element, targeting `#main`.
- A skip link hidden with `display:none`/`visibility:hidden` (removes it from the tab order entirely) instead of the visually-hidden-until-focused technique. Fix: position off-screen but focusable, revealed on `:focus`.
- Major regions as `<div>`s with no landmark roles. Fix: `<header>`/`<nav>`/`<main>`/`<aside>`/`<footer>` (or roles).
- Multiple `<nav>` elements with no `aria-label`, indistinguishable in the landmark list. Fix: label each (e.g. "Primary", "Footer").
- Content that should be in `<main>` sitting outside it, so "skip to main" lands wrong; or more than one `<main>` (must be exactly one).

The principle: **keyboard and screen-reader users navigate by skipping repeated blocks and jumping between landmarks, so a focusable skip link, proper landmark regions, labeled duplicate navs, and exactly one `main` are what make a page efficiently navigable.** Without them, every visit means Tabbing through the same header again and again, with no map of the page's structure.
</details>

---

*End of Track 7 — Accessibility & Semantics in Practice. 12 modules.*

*Next: Track 8 — Engineering & Tooling, in its own file.*
