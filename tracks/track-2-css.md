# TRACK 2 — CSS & LAYOUT

*The depth most developers never build. This track is vanilla-heavy on purpose: you cannot reason about Tailwind or styled-components until you can reason about the cascade, the box model, and the layout algorithms underneath them. It ramps from the box model to building your own constraint solver.*

---

## Module 2.1 — The Box Model & box-sizing

**Difficulty:** ●○○○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — The box model** — https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model — *Focus on: content, padding, border, margin; the difference between the standard (`content-box`) and alternative (`border-box`) box models.*
- **MDN — `box-sizing`** — https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing — *Focus on: exactly what `border-box` changes about how `width` is computed.*
- **CSS Spec — Box model** — https://www.w3.org/TR/CSS22/box.html — *Skim. Focus on: the formal definition of how widths and the box edges relate.*

**Optional course resources:**

- Josh Comeau — *CSS for JS Developers* — the "Rendering Logic" module on the box model.
- Kevin Powell — any of his box-model explainer videos on YouTube.

**Search prompts:**

- `"box-sizing border-box vs content-box difference"`
- `"why does width 100% plus padding overflow"`
- Ask an AI: *"With the default `content-box`, if I set `width: 200px; padding: 20px; border: 5px solid`, what is the total rendered width of the element and why? How does `box-sizing: border-box` change that calculation?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.1 — the box model and box-sizing.
Completed so far: Track 1 (HTML). For CSS, treat me as a beginner who has written styles
but never reasoned about box math.

Build me a single HTML/CSS file: a card grid or a form layout where elements OVERFLOW their
containers, widths do not add up the way a naive developer expects, and a "100% width" thing
spills past its parent. The visual symptoms should be obvious — things sticking out,
horizontal scrollbars, misaligned columns — but the cause should require understanding how
the browser computes the size of a box.

Tell me only how to run it. Do not tell me what is wrong. I will measure boxes in the
DevTools box-model diagram, figure out why the math breaks, and fix it. Make me explain, for
each fix, exactly what the browser includes in an element's rendered width under each box
model.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Elements with `width: 100%` PLUS `padding` and/or `border` under the default `content-box`, so total width exceeds 100% and overflows the parent.
- A fixed-width container holding fixed-width children whose padding pushes the total past the container.
- No global `box-sizing: border-box` reset, so every width calculation is "off" by padding+border.
- A `width: 100vw` element causing horizontal scroll because `vw` includes the scrollbar width.
- Percentage padding being computed against the parent's WIDTH (even vertical padding), surprising the developer.

The fix is usually the classic `*, *::before, *::after { box-sizing: border-box; }` reset, plus understanding why. The principle: **under `content-box`, `width` sizes only the content, and padding/border are added on top; under `border-box`, `width` is the visible box edge-to-edge.** Overflow bugs are almost always the box model doing exactly what it was told.

</details>

---

## Module 2.2 — Specificity & The Cascade

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Specificity** — https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity — *Focus on: the (a,b,c) scoring of ID, class, and type selectors; where inline styles and `!important` sit.*
- **MDN — The cascade** — https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade — *Focus on: the order the browser resolves conflicts — origin, specificity, then source order.*
- **MDN — `:where()` and `:is()`** — https://developer.mozilla.org/en-US/docs/Web/CSS/:where — *Focus on: `:where()` having zero specificity, a modern tool for taming the cascade.*
- **CSS Cascade spec** — https://www.w3.org/TR/css-cascade-4/ — *Skim the "Cascading" section for the formal ordering.*

**Optional course resources:**

- Josh Comeau — *CSS for JS Developers* — the cascade and specificity lessons.
- Kevin Powell — his specificity videos.

**Search prompts:**

- `"css specificity calculation id class element"`
- `"why is my css being overridden cascade"`
- `"css :where() zero specificity"`
- Ask an AI: *"Walk me through CSS specificity scoring with the (inline, id, class, type) model. Then tell me: when two rules have equal specificity, what decides the winner? Where do `!important`, inline styles, and `:where()` fit?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.2 — specificity and the cascade.
Completed so far: Track 1, Module 2.1.

Build me an HTML/CSS file — a themed component library page (buttons, cards, alerts in
variants) — where styles do NOT apply the way you would expect. A color you set is ignored;
a "more specific" rule loses to a "less specific" one; an !important creates a trap; a later
rule that should win does not. The page should look subtly wrong — wrong colors, wrong
states — in ways that trace back to cascade and specificity conflicts.

Tell me only how to run it. I will use the DevTools Styles panel (which shows
struck-through overridden rules) to see what is winning and why, then fix the conflicts —
WITHOUT carpet-bombing everything with !important. Make me compute the specificity of the
competing selectors out loud and explain which cascade rule actually decided each conflict.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A low-specificity rule (single class) being overridden by a higher-specificity one (ID, or chained selectors) the developer forgot about.
- An `!important` declaration somewhere that traps a value so nothing can override it normally — forcing a cascade dead-end.
- Two equal-specificity rules where the EARLIER one is winning because the "intended" rule appears earlier in source order (or vice versa).
- An inline `style` attribute beating all the stylesheet rules.
- An overly-specific selector (`nav ul li a.link`) that cannot be cleanly overridden by a sensible `.link--active`.
- A rule that looks like it should match but does not (typo, wrong combinator), so a less specific fallback wins.

Fixes should lower specificity, fix source order, or use `:where()` to neutralize specificity — not pile on `!important`. The principle: **the cascade resolves conflicts by origin, then specificity, then source order.** When a style "isn't working," the browser is applying a rule that won by these deterministic rules — DevTools shows you exactly which.

</details>

---

## Module 2.3 — Inheritance & Custom Properties

**Difficulty:** ●●○○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Inheritance** — https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance — *Focus on: which properties inherit by default, and the `inherit`, `initial`, `unset`, `revert` keywords.*
- **MDN — Using CSS custom properties** — https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties — *Focus on: how custom properties cascade and inherit, `var()` fallbacks, and scoping a variable to a selector.*
- **MDN — `@property`** — https://developer.mozilla.org/en-US/docs/Web/CSS/@property — *Focus on: typed custom properties, defaults, and animatability.*

**Optional course resources:**

- Josh Comeau — *CSS for JS Developers* — custom properties lessons.

**Search prompts:**

- `"css custom properties inheritance scope"`
- `"css variables var fallback theming"`
- Ask an AI: *"How do CSS custom properties interact with inheritance and the cascade? Show me how to build a theme (light/dark) using only custom properties scoped to a class, and explain why a variable set on a parent flows to children."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.3 — inheritance and custom properties,
build challenge. Completed so far: Track 1, Modules 2.1–2.2.

Give me a spec for a small component set (a button with variants, a card, an alert) that must
be themeable: a light theme and a dark theme, plus a "brand color" that flows through all
components, all driven by CSS custom properties — and switchable by adding ONE class to a
wrapper. No JavaScript. Give me the visual requirements only, no code.

My job: build the theming system using custom properties, inheritance, and scoping. The
theme switch must work by toggling a single class on an ancestor.

Review me like a senior. If I hardcode a color that should be a token, or set a variable at
the wrong scope so it does not flow correctly, do not fix it — ask me where that value comes
from and how the variable reaches (or fails to reach) the element. Make me explain why custom
properties inherit and how `var()` resolves, including its fallback.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Tokens (`--color-bg`, `--color-text`, `--color-brand`, spacing scale) defined on `:root`, then overridden on a `.theme-dark` class — so flipping the class re-scopes the variables for the whole subtree.
- Components consuming `var(--token)` rather than hardcoded values, so they respond to the theme automatically through inheritance.
- Correct use of `var(--x, fallback)` for resilience.
- Common mistakes to catch: setting theme variables on `:root` only (so the dark class on a wrapper has nothing to override); hardcoding a color in a component that should reference a token; expecting a custom property to inherit when it was set on a sibling, not an ancestor; not understanding that custom properties cascade like normal properties.
- Bonus: a typed `@property --brand-hue` to enable smooth animation between themes.

The principle: **custom properties are real cascading, inheriting CSS values — not compile-time find-and-replace.** Set a token on an ancestor and every descendant sees it; override it on a scoped class and you re-theme an entire subtree by changing one declaration.

</details>

---

## Module 2.4 — Normal Flow & Margin Collapse

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Normal flow** — https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Normal_Flow — *Focus on: block vs inline formatting, and how elements lay themselves out before you add any layout system.*
- **MDN — Mastering margin collapsing** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing — *Read the whole page. Focus on: the three cases where margins collapse, and what STOPS collapsing (padding, border, a BFC).*
- **MDN — Block formatting context** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_display/Block_formatting_context — *Focus on: what creates a BFC and how a BFC contains margins and floats.*

**Search prompts:**

- `"margin collapse parent child explained"`
- `"block formatting context what creates it"`
- `"why is there a gap above my element margin"`
- Ask an AI: *"Explain margin collapsing: the three scenarios where vertical margins collapse, and exactly what prevents it. Then explain what a Block Formatting Context is and how creating one changes margin and float behavior."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.4 — normal flow and margin collapse.
Completed so far: Track 1, Modules 2.1–2.3.

Build me an HTML/CSS file where vertical spacing behaves in confusing ways: a child's top
margin mysteriously pushes its PARENT down (leaving a gap above the parent), adjacent
elements' margins do not add up the way I would expect, and a container fails to wrap around
its floated child. No flexbox or grid — pure normal flow, so the flow rules are exposed.

Tell me only how to run it. The symptoms are spacing that "shouldn't be there" or "is the
wrong size." I will diagnose which margins are collapsing and why, and fix the layout so
spacing is predictable — understanding the fix, not just trying random properties. Make me
explain the margin-collapse rules and what a Block Formatting Context does.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Parent/child margin collapse: a child's `margin-top` collapses through the parent (which has no padding/border/BFC), so the gap appears ABOVE the parent instead of between parent and child.
- Adjacent sibling collapse: two stacked elements where the larger of the two margins wins rather than the margins summing.
- A container that does not contain its floated child (collapses to zero height) because it does not establish a BFC.
- Empty-element collapse (an element with only margins collapsing its own top and bottom together).
- A fix attempt using a tiny `padding` or `overflow: hidden`/`auto` that "works" but the developer cannot explain why (it creates a BFC or interposes a border edge).

The principle: **in normal flow, adjacent vertical margins collapse into a single margin (the larger one), and a child's margin can escape through a parent that has no border, padding, or BFC between them.** Creating a Block Formatting Context (`overflow`, `display: flow-root`, etc.) contains both margins and floats.

</details>

---

## Module 2.5 — Positioning: static, relative, absolute, fixed, sticky

**Difficulty:** ●●●○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Positioning** — https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning — *Focus on: the five `position` values and what each does to flow and the containing block.*
- **MDN — Containing block** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_display/Containing_block — *Focus on: how `position: absolute` finds its containing block (the nearest positioned ancestor), and how `transform`/`filter` create one.*
- **MDN — `position: sticky`** — https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky — *Focus on: the conditions sticky needs to work (a scroll container, a threshold, room to move).*

**Search prompts:**

- `"position absolute containing block nearest positioned ancestor"`
- `"position sticky not working overflow parent"`
- Ask an AI: *"Explain how `position: absolute` decides what to position relative to. What is a containing block, what establishes one, and why does adding `position: relative` to a parent change where an absolute child lands? Then: list every reason `position: sticky` silently fails."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.5 — positioning. Completed so far:
Track 1, Modules 2.1–2.4.

Build me a UI with several positioned elements: a badge pinned to the corner of a card, a
dropdown menu, a "back to top" button that should be fixed, and a section header that should
stick while its section scrolls. Construct them so each one is BROKEN in a realistic way: the
badge lands in the wrong place, the dropdown is positioned relative to the wrong ancestor,
the fixed button is not actually fixed, and the sticky header does not stick.

Tell me only how to run it. I will diagnose each positioning failure — figuring out which
containing block each element is actually resolving against, and why sticky is dead — and fix
them. Make me explain what establishes a containing block and the exact conditions sticky
requires.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- An `position: absolute` badge whose parent is NOT positioned, so it resolves against a distant ancestor (or the viewport) and lands wrong.
- A dropdown positioned absolutely inside a parent that unexpectedly IS a containing block (because of a `transform` or `filter` on an ancestor), shifting it.
- A `position: fixed` element that is not fixed to the viewport because an ancestor has a `transform`/`filter`/`will-change` (which makes it the containing block for fixed descendants).
- A `position: sticky` header that fails because: its parent has `overflow: hidden`, or there is no threshold (`top`/`bottom`) set, or the sticky element's parent is not tall enough to scroll, or a flex/grid alignment is interfering.
- Confusion between `relative` (offsets from its own normal position, keeps its space) and `absolute` (removed from flow).

The principle: **positioning is about the containing block — the rectangle an element's offsets resolve against.** `absolute`/`fixed` hunt up the tree for a positioned (or transform/filter) ancestor; `sticky` needs a scrollable container, a threshold, and room to travel. Most "it's in the wrong place" bugs are the element resolving against a containing block you did not expect.

</details>

---

## Module 2.6 — Flexbox Fundamentals

**Difficulty:** ●●○○○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Basic concepts of flexbox** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox — *Focus on: main axis vs cross axis, and that flex direction changes which is which.*
- **MDN — Aligning items in a flex container** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Aligning_items_in_a_flex_container — *Focus on: `justify-content` (main axis) vs `align-items` (cross axis) vs `align-content`.*
- **CSS Tricks — A Complete Guide to Flexbox** — https://css-tricks.com/snippets/css/a-guide-to-flexbox/ — *The reference diagram for every property.*

**Optional course resources:**

- Kevin Powell — his Flexbox crash content (excellent for axis intuition).
- Josh Comeau — *CSS for JS Developers* — the Flexbox module.

**Search prompts:**

- `"flexbox main axis cross axis justify align"`
- `"flex direction row column axis change"`
- Ask an AI: *"In flexbox, explain the main axis and cross axis, and how `flex-direction` swaps them. Then tell me which property aligns on which axis: `justify-content`, `align-items`, `align-self`, `align-content`. Quiz me with a few scenarios."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.6 — flexbox fundamentals. Completed so
far: Track 1, Modules 2.1–2.5.

Build me several small flex layouts — a navbar, a centered hero, a row of cards, a media
object (image + text), a footer with items pushed to opposite ends — where the alignment is
WRONG. Things that should be centered are not, items that should be spaced apart are bunched,
something that should be on the cross axis is being aligned on the main axis. The mistakes
should be the classic confusions between justify and align, and between the axes.

Tell me only how to run it. I will fix each layout by reasoning about which axis I am working
on and which property controls it. Make me state, for every fix, which is the main axis here,
which property I used, and why it aligns on that axis.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- Using `align-items` when the goal was main-axis distribution (`justify-content`) or vice versa.
- A column-direction flex container where the developer's mental model is still row, so justify/align feel "swapped."
- Trying to center both ways but only setting one axis.
- `align-content` confused with `align-items` (the former only matters with wrapped multi-line content).
- Pushing one item to the end with the wrong tool instead of `margin-left: auto` (a clean flex idiom) or `justify-content: space-between`.
- A media object where the image and text are not aligned to the top because `align-items` defaults to `stretch`.

The principle: **flexbox has a main axis (set by `flex-direction`) and a cross axis perpendicular to it.** `justify-content` distributes along the main axis; `align-items`/`align-self` align on the cross axis. Almost every "why won't it center" is using the axis-wrong property.

</details>

---

## Module 2.7 — Flexbox: The Six Production Bugs

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **W3C — Flexbox spec, §9.7 Resolving Flexible Lengths** — https://www.w3.org/TR/css-flexbox-1/#resolve-flexible-lengths — *The actual sizing algorithm. Read it slowly once.*
- **MDN — `min-width` (the `auto` value)** — https://developer.mozilla.org/en-US/docs/Web/CSS/min-width — *Focus on: why flex items refuse to shrink below their content size by default.*
- **MDN — Controlling ratios of flex items** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Controlling_ratios_of_flex_items_along_the_main_axis — *Focus on: `flex-grow`, `flex-shrink`, `flex-basis`, and the `flex` shorthand expansion.*
- **Stack Overflow — Why don't flex items shrink past content size** — https://stackoverflow.com/questions/36247140/why-dont-flex-items-shrink-past-content-size — *The canonical `min-width: auto` answer.*

**Optional course resources:**

- Kevin Powell — videos specifically on flexbox gotchas / `min-width: 0`.

**Search prompts:**

- `"flex item text overflow min-width auto"`
- `"flex-shrink 0 fixed width sidebar"`
- `"flexbox gap overflow flex-basis"`
- Ask an AI: *"Explain the flexbox flexible-lengths algorithm: given `flex-grow`, `flex-shrink`, and `flex-basis`, how does the browser compute each item's final main-size? Then explain the `min-width: auto` rule and why it causes text overflow, and why three `flex: 1` items overflow when you add a `gap`."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.7 — the six classic flexbox production
bugs. Completed so far: Track 1, Modules 2.1–2.6. Raise the difficulty here.

Build me one HTML/CSS file with six labelled sections, each a flex layout that breaks the way
real production flex layouts break. The territory: a fixed-width element that collapses; a
flex item whose long text overflows instead of truncating; an image that stretches to full
container height; cards that refuse to wrap; a wrapped last row that stretches a lone card;
and `flex: 1` items that overflow once a `gap` is added. Construct realistic instances —
do not label which bug is which beyond "section 1..6."

Tell me only how to run it. For each section I will diagnose the spec-level cause and fix it.
Do not accept a fix until I explain, at the level of the flex sizing algorithm or the
`min-width: auto` rule, WHY the browser behaved that way — not just which property I changed.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

1. **Collapsing fixed-width element:** a `width: 200px` sidebar without `flex-shrink: 0`, so it shrinks under pressure. Fix: `flex-shrink: 0` (or `flex: 0 0 200px`).
2. **Text overflow:** a flex item with long text overflowing because `min-width: auto` prevents it shrinking below content size. Fix: `min-width: 0` (or `overflow: hidden`) on the item.
3. **Stretched image:** an image stretched to the container's height because `align-items` defaults to `stretch`. Fix: `align-items: flex-start` or `align-self: flex-start`.
4. **No wrapping:** items with a `min-width` larger than the container can fit, so `flex-wrap: wrap` cannot help (an item cannot be smaller than its min-width). Fix: reduce `min-width` / use `flex-basis` + wrap correctly.
5. **Stretched last row:** `flex: 1` on cards makes a lone card on the final wrapped row span full width. Fix: cap with `max-width`, switch to grid, or use `flex-basis` math.
6. **Gap overflow:** three `flex: 1` items (basis `0%`) plus a `gap` overflow because gap space is additive and not absorbed by the flex grow. Fix: account for gap in basis (`calc`) or use grid.

The principle: **flex item sizing follows a precise algorithm, and the default `min-width: auto` floor is the source of most "it won't shrink" bugs.** Once you can recite what grow/shrink/basis and the min-size floor do, these stop being mysteries.

</details>

---

## Module 2.8 — Grid Fundamentals

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Basic concepts of grid layout** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout — *Focus on: tracks, lines, cells, areas; `grid-template-columns`/`rows`; the `fr` unit; `gap`.*
- **MDN — Line-based placement** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_using_line-based_placement — *Focus on: placing items with `grid-column`/`grid-row` and negative line numbers.*
- **MDN — `minmax()`, `repeat()`, `auto-fill`/`auto-fit`** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Auto-placement_in_grid_layout — *Focus on: responsive track patterns without media queries.*
- **CSS Tricks — A Complete Guide to CSS Grid** — https://css-tricks.com/snippets/css/complete-guide-grid/ — *The reference.*

**Optional course resources:**

- Kevin Powell — his Grid courses/videos (best practical grid teacher).
- Josh Comeau — *CSS for JS Developers* — the Grid module.

**Search prompts:**

- `"css grid fr unit explained"`
- `"grid repeat auto-fit minmax responsive"`
- Ask an AI: *"Explain the CSS Grid `fr` unit precisely: how does the browser size `fr` tracks when some columns are fixed? Then explain `repeat(auto-fit, minmax(200px, 1fr))` token by token and why it creates a responsive grid with no media queries."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.8 — grid fundamentals, build challenge.
Completed so far: Track 1, Modules 2.1–2.7.

Give me a spec for three layouts I must build with CSS Grid: (1) a classic holy-grail page
(header, footer, full-width; sidebar + main in the middle), (2) a photo gallery that goes
from 1 to N columns responsively WITHOUT media queries, and (3) a dashboard where specific
widgets span multiple columns/rows. Give me the requirements and the breakp's behavior, no
code.

My job: build all three with grid — using `fr`, `minmax`, `repeat`, `auto-fit`, and explicit
line placement as appropriate.

Review me strictly. If the responsive gallery uses media queries (it should not need them),
or a widget is placed by trial-and-error rather than by named lines/spans, do not fix it —
ask me what track structure I actually defined and whether the item's placement references
lines that exist. Make me explain `fr` sizing and the `auto-fit`/`minmax` mechanism.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Holy-grail: a clean `grid-template` with named areas or explicit lines; full-width header/footer via `grid-column: 1 / -1`.
- Responsive gallery: `grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))` — no media queries needed; understand `auto-fit` vs `auto-fill` (collapsing vs preserving empty tracks).
- Dashboard: widgets placed with `grid-column: span 2` / explicit line numbers; understand the difference between the explicit and implicit grid and how `grid-auto-rows` sizes implicit tracks.
- Common mistakes: using `fr` without realizing fixed tracks are subtracted first; reaching for media queries where `auto-fit`/`minmax` suffices; placing items with margins instead of grid placement; forgetting `gap`.

The principle: **grid is a true two-dimensional system: you define tracks and lines, then place items into the resulting cells.** `fr` distributes leftover space after fixed tracks; `repeat(auto-fit, minmax(...))` lets the grid itself decide column count from available width — responsiveness without breakpoints.

</details>

---

## Module 2.9 — Grid: The Magazine Layout

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Grid template areas** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas — *Focus on: the ASCII-art area syntax and the rule that areas must form rectangles.*
- **MDN — Named grid lines** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Layout_using_named_grid_lines — *Focus on: `[line-name]` syntax and `*-start`/`*-end` automatic naming from areas.*
- **Grid by Example (Rachel Andrew)** — https://gridbyexample.com/examples/ — *Run examples 12–22, focused on content-bleed and complex placement.*

**Optional course resources:**

- Jen Simmons — "Designing Intrinsic Layouts" talk (YouTube) — magazine-style thinking.

**Search prompts:**

- `"css grid full bleed content layout"`
- `"grid named lines bleed into margin"`
- Ask an AI: *"I want a magazine layout where body text is a centered ~65ch column but pull-quotes and images bleed into the margins. How do I structure the grid columns so some elements can extend past the text column? Explain the column scheme I need."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.9 — the magazine layout, build challenge.
Completed so far: Track 1, Modules 2.1–2.8.

Give me a long-form article's CONTENT and a precise layout spec: full-bleed hero; a centered
~65ch text column; a pull-quote that bleeds into the LEFT margin; an inline photo that spans
the text column plus the RIGHT margin; a caption aligned to the photo's outer edge; a sidebar
beside the final paragraphs on desktop; everything stacking below 700px. Provide the HTML
content and the non-grid styles only. I write all the grid CSS.

Review me like a senior. Before I write CSS, make me state how many columns this needs and
the purpose/width of each. When the pull-quote won't bleed, do not tell me the fix — ask me
which grid line it needs to start at and whether that line exists in my track definition.
Make me defend the column scheme from memory.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A multi-column track scheme such as: `[full-start] minmax(1rem,1fr) [content-start] min(65ch, 100%) [content-end] minmax(1rem,1fr) [full-end]` — giving named lines for full-bleed, the content column, and the margins. Many variants work; the key is having explicit lines for "content edge" and "page edge."
- Body text placed in the content column; the hero and full-bleed elements spanning `full-start / full-end`; the pull-quote spanning from a margin line into content; the photo spanning content into the right margin.
- A `@media (max-width: 700px)` (or a container query) collapsing everything to one column.
- Common mistakes: trying to bleed with negative margins instead of grid lines; not defining enough columns to express "content vs margin"; the sidebar placed with floats instead of grid; caption alignment done by hand instead of sharing the photo's grid line.

The principle: **bleed effects come from a grid that explicitly models the page margins as their own tracks.** Once "page edge," "margin," and "content edge" are named lines, any element can be told exactly how far to extend.

</details>

---

## Module 2.10 — Stacking Contexts & z-index

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — The stacking context** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context — *Read the full list of what creates a stacking context. This is the crux.*
- **MDN — Stacking without z-index** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_without_z-index — *Focus on: the default paint order before any z-index.*
- **Philip Walton — What No One Told You About z-index** — https://philipwalton.com/articles/what-no-one-told-you-about-z-index/ — *The canonical explanation: z-index only compares within the same stacking context.*
- **Chrome DevTools — Layers panel** — https://developer.chrome.com/docs/devtools/layers — *How to see compositing layers.*

**Optional course resources:**

- Josh Comeau — *CSS for JS Developers* — the stacking context lesson.

**Search prompts:**

- `"css stacking context complete list properties"`
- `"z-index 9999 not working transform opacity"`
- Ask an AI: *"List every CSS property that creates a new stacking context, and for each explain why. Then: my modal has z-index 9999 but renders behind a card with z-index 1 — give me the three most likely causes and how to diagnose which one it is in DevTools."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.10 — stacking contexts and z-index.
Completed so far: Track 1, Modules 2.1–2.9.

Build me one HTML/CSS file with three scenes where z-index appears broken in ways that look
impossible: a high-z-index modal trapped behind a low-z-index card; a max-z-index tooltip
clipped by a scroll container; a sticky header that vanishes behind a section that has
"performance optimization" CSS on it. The z-index VALUES will look correct — the cause is
stacking contexts created by non-obvious properties.

Tell me only how to run it. Do not say "stacking context." I will open the DevTools Layers
panel, discover which property is creating an unexpected context, and fix each scene WITHOUT
just cranking z-index higher (and for the clipped tooltip, without `position: fixed`). Make
me name the property that created each context and explain why it does.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- **Scene 1:** the card has `transform: translateY(0)` (a "harmless" hint) which creates a stacking context, trapping the modal that lives inside or behind it. Fix: remove the transform, or move the modal out of that context (e.g. portal to `<body>`).
- **Scene 2:** the tooltip is clipped because an ancestor has `overflow: auto/hidden`. The structural fix is to render the tooltip outside the clipping ancestor (portal/restructure), NOT `position: fixed`.
- **Scene 3:** the content section has `will-change: transform` and/or `filter: brightness(1)`, each creating a stacking context that outranks the sticky header's. Fix: remove the unnecessary "optimizations."
- The z-index numbers themselves are correct throughout — that is the lesson.

The principle: **z-index only orders elements within the same stacking context; many properties (`transform`, `opacity < 1`, `filter`, `will-change`, `isolation`, positioned + z-index, etc.) silently create new contexts.** A trapped element is in a lower-ranked context, so no z-index value inside it can escape. DevTools Layers shows you the truth.

</details>

---

## Module 2.11 — Responsive Design & Media Queries

**Difficulty:** ●●○○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Beginner's guide to media queries** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries — *Focus on: syntax, `min-width` vs `max-width`, and combining conditions.*
- **MDN — Responsive design** — https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design — *Focus on: mobile-first methodology and choosing breakpoints by content, not devices.*
- **web.dev — Responsive web design basics** — https://web.dev/articles/responsive-web-design-basics — *Focus on: the viewport meta tag's role and fluid layouts.*
- **MDN — `prefers-color-scheme`, `prefers-reduced-motion`** — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme — *Focus on: media queries that respond to user preferences, not just size.*

**Optional course resources:**

- Kevin Powell — his responsive design content (mobile-first, content-based breakpoints).

**Search prompts:**

- `"mobile first vs desktop first media queries"`
- `"choosing breakpoints by content not device"`
- Ask an AI: *"Make the case for mobile-first CSS with `min-width` queries over desktop-first `max-width`. Then explain how to choose breakpoints based on where the content breaks rather than on specific device widths."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.11 — responsive design, build challenge.
Completed so far: Track 1, Modules 2.1–2.10.

Give me a spec for a landing page that must work from 320px to ultrawide: a nav that becomes
a stacked menu on small screens, a hero whose layout reflows, a card grid that changes column
count, and typography that adapts. Require it to be mobile-first and to respect
`prefers-color-scheme` and `prefers-reduced-motion`. Give me the requirements, no code.

My job: build it mobile-first, adding complexity at `min-width` breakpoints chosen where the
CONTENT breaks. Review me strictly. If I write desktop-first overrides, or pick arbitrary
device-pixel breakpoints, or ignore the preference queries, do not fix it — ask me what
happens at 320px first, and where the layout actually starts to look bad as it widens. Make
me justify every breakpoint by pointing at the content, not a phone model.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Base styles target the smallest screen; `@media (min-width: ...)` layers on enhancements as space allows (mobile-first).
- Breakpoints chosen where the design breaks (e.g. when cards get too narrow), not at 768/1024 by rote.
- A nav that is a vertical list by default and becomes horizontal at a breakpoint (often achievable with flex/grid + one query, sometimes zero queries with `flex-wrap`).
- `@media (prefers-color-scheme: dark)` adjusting theme tokens; `@media (prefers-reduced-motion: reduce)` disabling non-essential animation.
- Common mistakes: desktop-first `max-width` cascades that fight themselves; fixed-px widths that overflow at 320px; using `vw`-only typography that becomes unreadable at extremes (should combine with `clamp`); forgetting the viewport meta.

The principle: **mobile-first means the simplest layout is the default and complexity is added as the viewport grows, which matches how `min-width` queries cascade.** Breakpoints belong where your content stops looking good, and good responsive design also responds to user *preferences*, not just screen size.

</details>

---

## Module 2.12 — Container Queries

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Container queries** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries — *Focus on: `container-type`, `container-name`, and the `@container` rule. The shift from "how big is the viewport" to "how big is my container."*
- **MDN — CSS containment** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Using_CSS_containment — *Focus on: what `contain` does and why container queries require a containment context.*
- **web.dev — Container queries** — https://web.dev/articles/cq-stable — *Focus on: the component-reusability problem they solve that media queries cannot.*

**Search prompts:**

- `"container queries vs media queries difference"`
- `"container-type inline-size @container"`
- Ask an AI: *"Why do container queries exist if we already have media queries? Give me a concrete component (a card that should be horizontal when wide and stacked when narrow) that is impossible to do correctly with media queries alone, and show how container queries solve it. Explain `container-type: inline-size`."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.12 — container queries, build challenge.
Completed so far: Track 1, Modules 2.1–2.11.

Give me a spec for a SINGLE reusable card component that must adapt to the width of whatever
container it is dropped into — horizontal layout when its container is wide, stacked when
narrow — and then a page that places the SAME card in three different-width slots (a wide
main area, a narrow sidebar, a medium grid cell) at the same time. Give me requirements, no
code.

The trap: this is impossible to do correctly with viewport media queries, because the same
card at the same viewport must render differently depending on its slot. My job: solve it
with container queries.

Review me strictly. If I reach for media queries, do not tell me why they fail — drop the
card into the narrow sidebar and the wide main at the same viewport and ask me why one media
query cannot satisfy both. Make me explain `container-type` and why a containment context is
required.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The card's wrapper (or the slots) declare `container-type: inline-size` (and optionally `container-name`), establishing a query container.
- The card's internal styles use `@container (min-width: ...)` to switch between stacked and horizontal — based on the CONTAINER's width, so the same card adapts per-slot.
- Common mistakes: putting `container-type` on the card itself and then trying to query its own size (you query an ANCESTOR container, not self); using media queries (which only see the viewport and therefore render all three slots identically at a given viewport); forgetting that `container-type: inline-size` also establishes layout containment which can affect height.

The principle: **media queries ask "how big is the viewport"; container queries ask "how big is my container."** A truly reusable component cannot know the viewport is relevant — it only knows the space it was given. Container queries let a component respond to its own context, which is what makes it portable.

</details>

---

## Module 2.13 — Modern CSS: clamp, min, max, and Fluid Type

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — `clamp()`** — https://developer.mozilla.org/en-US/docs/Web/CSS/clamp — *Focus on: the `clamp(MIN, PREFERRED, MAX)` model and how the preferred value (often using `vw`) flexes between bounds.*
- **MDN — `min()` and `max()`** — https://developer.mozilla.org/en-US/docs/Web/CSS/min — *Focus on: using them for responsive sizing without queries (e.g. `width: min(90%, 60ch)`).*
- **web.dev — min(), max(), clamp()** — https://web.dev/articles/min-max-clamp — *Focus on: real responsive patterns built without breakpoints.*
- **Utopia.fyi — Fluid type** — https://utopia.fyi/type/calculator/ — *A calculator; understand the math behind a fluid type scale.*

**Search prompts:**

- `"css clamp fluid typography vw calc"`
- `"width min 90% 60ch responsive without media query"`
- Ask an AI: *"Explain `clamp(MIN, PREFERRED, MAX)` and how to build fluid typography that scales smoothly with the viewport between a floor and a ceiling. Walk me through deriving the `vw`-based preferred value so the font hits exactly my min size at one viewport and my max at another."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.13 — fluid sizing with clamp/min/max,
build challenge. Completed so far: Track 1, Modules 2.1–2.12.

Give me a spec for a page whose typography and spacing scale FLUIDLY between a small-screen
floor and a large-screen ceiling — ideally with as few media queries as possible. Headings
grow smoothly with the viewport but never below X or above Y; the content column is
`min(some%, some ch)`; spacing scales fluidly. Give me the target min/max values and
viewports, no code.

My job: implement it with `clamp()`, `min()`, `max()`. I should be able to derive a fluid
type value that hits exactly the floor at the small viewport and the ceiling at the large
one. Review me strictly. If a heading jumps instead of scaling, or overshoots its bounds, do
not fix it — ask me what my preferred (middle) value evaluates to at the two key viewports.
Make me show the math.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Fluid headings via `font-size: clamp(MIN, calc(... + ...vw), MAX)` where the middle term is derived so the value equals MIN at the small viewport and MAX at the large one (the Utopia/linear-interpolation math).
- Content width as `width: min(60ch, 100% - 2rem)` — capping line length while staying fluid, no media query.
- Fluid spacing using `clamp()` for section padding/gaps.
- Common mistakes: a `vw`-only font-size with no clamp (unbounded, unreadable at extremes); a `clamp` whose preferred term is a constant (so it never actually scales); getting the interpolation math wrong so it overshoots; using `clamp` where a simple `min()` would be clearer.
- Verify: resize the viewport and confirm values move smoothly and pin at the bounds.

The principle: **`clamp(MIN, PREFERRED, MAX)` is a self-contained responsive value — the preferred term (usually viewport-relative) interpolates, while MIN/MAX cap it.** This replaces whole stacks of media-query font-size overrides with one continuous function.

</details>

---

## Module 2.14 — Transitions & Keyframe Animations

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Using CSS transitions** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions/Using_CSS_transitions — *Focus on: what is animatable, `transition-property/duration/timing-function/delay`, and that transitions need a state change to trigger.*
- **MDN — Using CSS animations** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations — *Focus on: `@keyframes`, `animation-*` properties, `animation-fill-mode`, iteration.*
- **MDN — `<easing-function>`** — https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function — *Focus on: `cubic-bezier`, `steps()`, and why easing makes motion feel natural.*
- **web.dev — Animations guide** — https://web.dev/articles/animations-guide — *Focus on: which properties are cheap to animate (compositor) vs expensive (layout/paint).*

**Optional course resources:**

- Josh Comeau — *Whimsical Animations* / the Animations content — easing, orchestration, and taste.

**Search prompts:**

- `"css transition vs keyframe animation when to use"`
- `"cubic-bezier easing function explained"`
- `"animation-fill-mode forwards backwards"`
- Ask an AI: *"What is the difference between a CSS transition and a `@keyframes` animation, and when do I use each? Explain `animation-fill-mode` (none/forwards/backwards/both) with a concrete example, and why easing functions matter for perceived quality."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.14 — transitions and keyframe animations,
build challenge. Completed so far: Track 1, Modules 2.1–2.13.

Give me a spec for a few motion pieces: a button with a tasteful hover/active transition; a
card that animates in on load; a loading spinner; and a notification toast that slides in,
pauses, and slides out. Specify the feel (timing, easing, whether state should persist after
the animation). Give me requirements, no code.

My job: build each with the right tool — transition vs `@keyframes` — and correct easing and
fill-mode. Also respect `prefers-reduced-motion`. Review me strictly: if my toast snaps back
to its start position after animating (fill-mode), or I use a keyframe animation where a
transition was right (or vice versa), do not fix it — ask me what state the element is in
before, during, and after, and which tool models that. Make me justify each easing choice.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Button hover: a `transition` (state change driven by `:hover`/`:active`) — not a keyframe animation.
- Card-in and spinner and toast: `@keyframes` + `animation`, because they run without a user-driven state change or need multiple steps/looping.
- Correct `animation-fill-mode: forwards` so the toast/card stays at its end state instead of snapping back; understanding `backwards`/`both`.
- Easing chosen intentionally (ease-out for entrances, custom `cubic-bezier` for character) rather than the default `ease`.
- A `@media (prefers-reduced-motion: reduce)` block disabling/cutting the motion.
- Common mistakes: animating `width`/`top`/`left` (layout-triggering) instead of `transform`/`opacity`; transition with nothing to transition to; missing fill-mode; ignoring reduced-motion.

The principle: **transitions interpolate between two states triggered by a change; keyframe animations define their own multi-step timeline that can run unprompted and loop.** Fill-mode controls whether the element holds its animated state. Easing is what separates motion that feels mechanical from motion that feels designed.

</details>

---

## Module 2.15 — Transforms & The Compositor

**Difficulty:** ●●●●○ · **Format:** Broken project · **Stack:** Vanilla HTML/CSS/JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Using CSS transforms** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transforms/Using_CSS_transforms — *Focus on: `translate`, `scale`, `rotate`, the transform origin, and that transforms do not affect layout (they paint).*
- **web.dev — Stick to compositor-only properties** — https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count — *Focus on: why `transform`/`opacity` animate cheaply and `top`/`left`/`width` do not.*
- **web.dev — Animations and performance** — https://web.dev/articles/animations-and-performance — *Focus on: the layout → paint → composite pipeline and which properties trigger which stage.*
- **CSS Triggers** — https://csstriggers.com — *Look up properties to see whether they trigger layout, paint, or composite.*

**Optional course resources:**

- Josh Comeau — Animations content on performant motion (transform/opacity, FLIP).

**Search prompts:**

- `"css transform vs top left animation performance jank"`
- `"compositor only properties transform opacity"`
- `"FLIP animation technique explained"`
- Ask an AI: *"Why can the browser animate `transform` and `opacity` at 60fps without triggering layout or paint, but animating `top`/`left`/`width` causes jank? Explain the compositor thread and what 'compositor-only' means. Then explain the FLIP technique."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.15 — transforms and the compositor.
Completed so far: Track 1, Modules 2.1–2.14. Raise the difficulty.

Build me a page with animations that LOOK like they should be smooth but are janky — a list
that reorders by animating positions, a panel that expands, an element that moves across the
screen — all implemented using layout-triggering properties (animating `top`/`left`/`width`/
`height`). Provide a way to throttle CPU in DevTools and to record a Performance trace.

Tell me only how to run it. I will record traces, see the jank and what stage of the pipeline
is firing each frame, and rework the animations to be compositor-friendly (transform/opacity,
and the FLIP technique where positions change). Make me explain why the original properties
forced layout every frame and why transform does not.
```

### 🔒 SPOILER — The Planted Bugs

<details>
<summary>Click only after you finish or give up</summary>

- A moving element animated via `left`/`top` (or `margin`) — triggers layout every frame. Fix: animate `transform: translate()`.
- A panel expanding by animating `height`/`width` — layout thrash. Fix: animate `transform: scaleY()` (with care for content), or use a grid/`max-height` technique, or measure-and-FLIP.
- A list reorder where items jump or animate by position properties — should use the FLIP technique: measure First and Last positions, Invert with a transform, Play by transitioning the transform to zero.
- Possibly a `box-shadow` or `width` animation causing paint/layout where a cheaper approach exists.
- The Performance trace shows long "Layout"/"Recalculate Style"/"Paint" work on the main thread each frame; the fixed version shows compositor frames with minimal main-thread work.

The principle: **the browser can move and fade layers on the compositor thread without re-running layout or paint — but only for `transform` and `opacity`.** Animating geometric properties forces the main thread to recompute layout 60 times a second. FLIP lets you achieve position changes using only transforms.

</details>

---

## Module 2.16 — Logical Properties & Writing Modes

**Difficulty:** ●●●○○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — CSS logical properties** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values — *Focus on: `margin-inline`, `padding-block`, `inset-inline-start`, etc., and how they map to physical directions based on writing mode.*
- **MDN — Writing modes** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_writing_modes — *Focus on: `writing-mode`, `direction`, and the inline vs block axes.*
- **web.dev — Logical properties** — https://web.dev/articles/learn/css/logical-properties — *Focus on: why logical properties make layouts internationalization-ready by default.*

**Search prompts:**

- `"css logical properties inline block start end"`
- `"writing-mode rtl logical properties internationalization"`
- Ask an AI: *"Explain CSS logical properties: what are the inline and block axes, and how do `margin-inline-start`, `padding-block`, and `inset-inline-end` map to physical sides depending on writing mode and direction? Why does using them make a layout work in RTL languages automatically?"*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.16 — logical properties and writing modes,
build challenge. Completed so far: Track 1, Modules 2.1–2.15.

Give me a spec for a component (say a card with an icon, text, and a trailing action, plus
some spacing and a left/right border accent) that must work CORRECTLY in both left-to-right
AND right-to-left languages — when I flip the document to RTL, the layout should mirror
properly with no per-direction overrides. Give me requirements, no code.

The trap: if I use physical properties (`margin-left`, `padding-right`, `left`), the RTL
version breaks and I will be tempted to write a pile of `[dir="rtl"]` overrides. My job:
build it with logical properties so a single `dir="rtl"` on the root mirrors everything.

Review me strictly. If I use physical properties and then patch RTL with overrides, do not
tell me the logical equivalent — flip the dir, show me what breaks, and ask me which axis
("inline") that spacing actually lives on. Make me explain the inline/block model.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- Spacing and borders expressed logically: `margin-inline-start/end`, `padding-block`, `border-inline-start`, `inset-inline-start`, `text-align: start/end` — so they follow the inline direction.
- Flipping `dir="rtl"` (or `writing-mode`) mirrors the whole component with zero overrides.
- Common mistakes: using `margin-left`/`padding-right`/`left`/`text-align: left` (physical), then needing `[dir="rtl"]` overrides; mixing logical and physical so only half mirrors; not realizing `width`/`height` have logical equivalents (`inline-size`/`block-size`) too.

The principle: **logical properties describe space along the inline and block axes rather than physical left/right/top/bottom, so the browser maps them correctly for any writing mode or direction.** Build with them and internationalization (RTL, vertical scripts) mostly comes for free instead of as a second layout you maintain by hand.

</details>

---

## Module 2.17 — Subgrid & Advanced Grid Alignment

**Difficulty:** ●●●●○ · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — Subgrid** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid — *Focus on: how a nested grid can adopt its parent's tracks so items in separate children align to a shared grid.*
- **MDN — Box alignment in grid** — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_alignment/Box_alignment_in_grid_layout — *Focus on: `justify-items`/`align-items`/`justify-self`/`align-self` and `place-*` shorthands.*
- **web.dev — Subgrid** — https://web.dev/articles/css-subgrid — *Focus on: the "cards with aligned rows" problem subgrid solves.*

**Search prompts:**

- `"css subgrid align cards across columns"`
- `"subgrid vs nested grid difference"`
- Ask an AI: *"What problem does CSS subgrid solve that a normal nested grid cannot? Give me the canonical example — a row of cards where each card's title, body, and footer should align across all cards even though content lengths differ — and explain how subgrid makes the inner elements share the outer grid's row tracks."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.17 — subgrid, build challenge. Completed
so far: Track 1, Modules 2.1–2.16.

Give me a spec for a row of cards (varying content lengths) where, ACROSS all cards, the
titles must align to the same baseline row, the bodies share a row, and the footers align —
so the cards read like a clean table even though each card is an independent component.
Without subgrid this requires fixed heights or JS measuring. Give me requirements, no code.

My job: solve it with subgrid so each card participates in the parent grid's row tracks.
Review me strictly. If I fall back to fixed heights or hacks, do not hand me subgrid — show
me two cards with very different content lengths, point at the misaligned footers, and ask me
how the inner rows could be made to share the OUTER grid's tracks. Make me explain what
`grid-template-rows: subgrid` actually does.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- The card container is a grid with explicit row tracks (e.g. `grid-template-rows: auto 1fr auto` for title/body/footer); each card spans those rows and declares `grid-template-rows: subgrid` so its internal title/body/footer line up to the PARENT's rows.
- The cards laid out in columns by the outer grid; each card a subgrid on the row axis.
- Result: all titles align, all footers align, no fixed heights, no JS.
- Common mistakes: trying to solve it with `align-items`/`flex` (cannot align across separate flex containers); fixed heights (brittle); not declaring `subgrid` on the right axis; forgetting the card must span the parent rows it wants to subscribe to.

The principle: **subgrid lets a nested grid borrow its parent's track lines, so elements in separate children can align to one shared grid.** It solves the long-standing "align things across independent components" problem that flex and nested grid cannot, without fixed sizing or measurement.

</details>

---

## Module 2.18 — Build Your Own Layout Engine (Constraint Solver)

**Difficulty:** ●●●●● · **Format:** Build challenge · **Stack:** Vanilla HTML/CSS/JS

### Prep — Read Before You Start

**Primary sources:**

- **MDN — ResizeObserver** — https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver — *Focus on: `contentRect`, when the callback fires, and using it to re-run layout on resize.*
- **MDN — CSS Layout API (Houdini)** — https://developer.mozilla.org/en-US/docs/Web/API/CSS_Layout_API/Using_the_CSS_Layout_API — *Focus on: what a layout worklet receives and returns; the browser's own layout contract.*
- **Overconstrained — Cassowary intro** — https://overconstrained.io — *Focus on: constraints vs rules, and priorities when constraints conflict.*
- **Paul Hudson — SwiftUI layout system** — https://www.hackingwithswift.com/books/ios-swiftui/understanding-the-swiftui-layout-system — *Focus on: the propose-size/return-size two-phase model.*

**Search prompts:**

- `"constraint solver layout algorithm cassowary"`
- `"ResizeObserver re-layout on container resize"`
- Ask an AI: *"Explain how a constraint-based layout system (like Cassowary, used in Apple's Auto Layout) works at a high level. What is a constraint, what does it mean for constraints to conflict, and how do priorities resolve conflicts? Give me a tiny example with two boxes."*

### The Prompt

```text
Follow the teaching contract in CLAUDE.md. Module 2.18 — build your own layout engine,
build challenge. Completed so far: ALL of Track 2 so far. This is the capstone — make it
genuinely hard.

Give me a starter (one HTML file): a canvas div with three absolutely-positioned boxes A, B,
C; buttons to toggle constraints ("B right of A", "C below A", "A width = B width", "keep all
in bounds"); a `renderBoxes()` that applies positions from a state object (already written);
and a `solveConstraints()` that is an EMPTY stub. Wire a ResizeObserver on the canvas that
calls solve+render. Drag-to-move should be wired. Do NOT implement solveConstraints.

My job: implement solveConstraints so all active constraints hold simultaneously, in real
time as I drag and as the canvas resizes. Review me like a senior. Before I write code, make
me write each constraint as a plain-English equation. When constraints conflict, ask me which
should win and how priorities express that. When the solver fails on resize, ask me what
`contentRect` gave me and when the observer fired. After it works, make me compare my
sequential solver to what a real constraint solver (Cassowary) and the CSS layout engine do.
```

### 🔒 SPOILER — What A Rigorous Reviewer Pushes On

<details>
<summary>Click only after you finish or give up</summary>

- A `solveConstraints()` that reads the active-constraint set and mutates box positions to satisfy each: `B.x = A.x + A.w + gap`, `C.y = A.y + A.h + gap`, `A.w = B.w`, and clamping all boxes within `[0, canvasWidth/Height]`.
- Handling INTERACTING constraints (B-right-of-A plus A-width=B-width) by iterating to a fixed point or defining an evaluation order, since solving one changes the inputs to another.
- The ResizeObserver re-running the solve using `entry.contentRect` so "keep in bounds" re-fires on resize.
- A discussion of where a naive sequential solver fails (genuinely circular constraints, conflicting equal-priority constraints) and how Cassowary's incremental simplex with priorities handles what yours cannot.
- Common mistakes: solving each constraint once without iterating (dependent constraints settle wrong); reading stale dimensions instead of `contentRect`; no conflict/priority strategy; forgetting to re-solve on drag AND resize.

The principle: **layout is constraint satisfaction.** CSS, Auto Layout, and SwiftUI are all systems that take a set of constraints and compute positions that satisfy them. Building even a toy solver makes the browser's layout engine legible: you feel why order, conflicts, and priorities are the hard parts, and why the platform's solver is worth respecting.

</details>

---

*End of Track 2 — CSS & Layout. 18 modules.*

*Next: Track 3 — JavaScript & the Language, in its own file.*