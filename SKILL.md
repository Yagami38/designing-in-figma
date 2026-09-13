---
name: designing-in-figma
description: Use when designing a website or an app in Figma through the Figma MCP (create_new_file, use_figma, get_screenshot) — a new or existing file, a moodboard, a UI kit, components, a first mockup, deriving the remaining pages — or when someone asks for "the Figma design of…", "the mockups for…", wants icons drawn, extra colors, a shadow or an effect, all pages at once, or says they will review at the end.
---

# Designing a website or an app in Figma

## Overview

**The moodboard decides, variables carry, components repeat, one page is validated before the others.** Nothing is inferred silently: a color, a typeface, an effect, a layout are questions asked to the user, never choices made on their behalf. A deadline shrinks the scope — fewer pages, fewer states — never a phase or a gate.

Figma's official skills describe the API; this skill describes the method. Load before any call: `figma-use` before every `use_figma`, `figma-create-new-file` before `create_new_file`, `figma-generate-library` as soon as a component is created. Load the Figma tool schemas in a single `ToolSearch` with `select:`.

This skill can be copied as is into `~/.claude/skills/` or into a project's `.claude/skills/`. If the project has its own Figma standards document, that document wins wherever it contradicts this skill.

**Language.** Page, Section, variable, component and layer names follow the **user's language**, consistently across the whole file: the English names used below (`Components`, `Color`, `Summary`, `Button`) are translated when the user works in another language (`Composant`, `Couleur`, `Synthèse`, `Bouton`). Mockup content is written in the product's language.

## The free plan, stated up front

- **Three pages per file, not one more: `Design`, `Components`, `Moodboard`.** Everything else is a Section. Whatever gets replaced goes into an `Archive` Section at the end of `Design`, never to the trash.
- **One mode per variable collection.** No Light/Dark or Mobile/Desktop through modes: differences between formats are handled variant by variant.
- **No team library.** Everything lives in the file.
- The API cannot create a Figma folder: the user creates it by hand and provides the link.

## The three gates

| Gate | After                         | What is shown                                                                               |
| ---- | ----------------------------- | ------------------------------------------------------------------------------------------- |
| 1    | the moodboard analysis        | the reading of the moodboard and the `Summary`: every choice validated question by question |
| 2    | the UI kit and the components | a screenshot of the `Components` page, empty audits                                         |
| 3    | the first mockup, retouched   | the page validated by the user in Figma, their retouches pushed back into the masters       |

A gate is a stop: show, ask, end the turn, wait. "I'll review at the end" says **when** the user validates, not **whether**: the first page is shown and the wait happens anyway.

## Phase 0 — Scoping, before any call

Ask these questions in batches of four at most, and open no tool before the answers:

1. **Does a Figma file already exist?** Yes → its link. No → where should it be created? Recommend a **Figma folder** (project) created by the user, who provides its link; otherwise the root (drafts).
2. **Website or app**, and the list of pages or screens.
3. **Formats**: recommend **mobile first, then desktop**; the user decides.
4. **Icon library**: recommend Material Design Icons. No icon will ever be drawn.
5. **Language of names** (the user's) and **language of content** (the product's).
6. **Stack and assets**: a stack with recommended fonts (Shopify…)? A logo, photos?

## Phase 1 — File, moodboard, summary

1. Create or rename the three pages (`references/setup.md` §1). On `Moodboard`, a `Moodboard 1` Section with four **empty** sub-sections: `Design`, `Color`, `Typography`, `Avoid`. Stop: the user fills it in. If they ask for inspiration, search **Dribbble only** (`references/setup.md` §4).
2. **Analyze**: screenshot each sub-section and write down what is there — candidate colors with their value and source image, families and weights, grid, density, radii, image treatment, and **every specific effect**: shadow, gradient, blur, border, texture, illustration, hint of motion.
3. **Ask**: one choice per question, with the options seen in the moodboard; every effect spotted gets an explicit question — is it wanted, and where. In batches of four at most, until nothing is left.
4. Write the decisions into a `Summary` sub-section of `Moodboard 1`. **Gate 1.**

## Phase 2 — UI kit

On `Components`, a `UI Kit` Section (`references/ui-kit.md`). Four single-layer collections: `color` kept minimal — one primary, up to three secondaries, neutrals from white to black, success and error — `space` on the 4 … 128 scale, `radius`, `type`. Seven text styles (H1 to H4, Body, Small, Caption), in two groups if two formats were chosen. **Google Fonts** by default, unless the stack recommends its own fonts. An `Icon` set imported from the chosen library. The atoms with all their states. Every variant set in **auto-layout, hugging both axes**, otherwise variants get clipped.

## Phase 3 — Components

On `Components`, one Section per family (`references/components.md`): navigation, footer, forms, cards, then **every section of the product**. States — empty, loading, error, success — are variants; `Breakpoint` exists only if two formats were chosen. Audits, screenshot. **Gate 2.**

## Phase 4 — First mockup

A single page, chosen with the user, in the first format, assembled **from instances only**, inside a Section of `Design` (`references/setup.md` §3). Screenshot, audits. **Gate 3**: the user retouches directly in Figma, then validates. Then re-read the validated page and **push their retouches back into the masters** (`references/components.md` §6) — otherwise the other pages will not have them.

## Phase 5 — Deriving the other pages

The remaining pages, in this order: instance as is, instance with overrides, new variant, and only then a new component. Then the second format, derived from the first by cloning. Before handing back: audits on every page, Sections tidied, screenshots looked at.

## Non-negotiable rules

1. **Everything in variables**: color, spacing, padding, radius, size and line height, family and weight. Zero bare values.
2. **The scale**: `4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128` — pick from it, never leave it.
3. **Minimal colors**: one primary, one to three secondaries at most, the neutrals, success and error. Nothing else without a line in the `Summary`.
4. **Icons imported** from a single library, never drawn.
5. **Variant sets in auto-layout and hug**, variants with a fixed width.
6. **Any block present twice is a component**, master on `Components`, page = instances.
7. **One page validated before the others.**
8. **Consistency**: one recipe per role — a button, a section title, a card each have a single recipe.
9. **Never delete**: `Archive` Section, renamed with the date.
10. **Nothing inferred silently**: typeface, color, effect, layout — one question, one answer, one line in the `Summary`.

## Rationalizations to recognize

| What you tell yourself                                            | What is true                                                                                                                       |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| "She said she'd review at the end, I'll go on without validation" | "At the end" says when she validates, not whether. Show the first page and wait.                                                   |
| "No fake pause that would contradict her request"                 | Gates are not pauses: they are where the work becomes hers. Without them, five pages to redo instead of one.                       |
| "Hand-drawn icons feel more personal, it's the right request"     | A drawn glyph is off-grid, unfindable in code, redone on every project. Personality comes from the moodboard. Propose the library. |
| "No brand constraint, I'll pick the typography myself"            | No typeface without a moodboard. Without a moodboard, create it and wait until it is filled.                                       |
| "A real interface needs shades, two or three per hue"             | States are done with `primary-hover` and the neutrals. One more hue is a decision written in the `Summary`, not a shade.           |
| "A tasteful default shadow, adjustable later"                     | An unvalidated effect is a decision made in the user's place. Ask: is it wanted, where, how.                                       |
| "A Foundations page, a Components page, five site pages"          | Three pages. The rest are Sections.                                                                                                |
| "I'll start with placeholders so I don't block"                   | Nothing gets built before gate 1. Time saved is lost redoing.                                                                      |
| "`get_variable_defs` is enough to verify"                         | It lists variables, not unbound nodes. Audits are scripts, in a separate call.                                                     |

## Red flags — stop

- "do all the pages at once", "I'll review at the end", "draw me some icons", "add a green and an ochre", "a shadow like on that site"
- a typeface, a hex or an effect with no line in the `Summary`
- a page frame created before gate 2
- a fourth Figma page
- `create_new_file` without having asked whether a file exists and where to create it
- `.remove()` on anything

## Before every gate — mandatory

- [ ] Official skills loaded before the first write call
- [ ] Inventory read before building (`references/audits.md` §0); nothing built twice
- [ ] Audits re-run in a separate call, every list empty: §1 unbound values, §3 variant sets (gate 2), §2 overflow and §6 headings (gate 3 and later), §7 Sections
- [ ] Screenshots looked at, in every chosen format
- [ ] Nothing deleted: whatever was replaced is in `Archive`, dated
- [ ] Message: what was done (names and ids), open questions, what the user must validate — then end the turn

## References

| File                       | When to open it                                                                  |
| -------------------------- | -------------------------------------------------------------------------------- |
| `references/setup.md`      | phase 1: pages, moodboard, Dribbble inspiration, `Design` page layout            |
| `references/ui-kit.md`     | phase 2: the exact inventory of variables, styles, icons, atoms                  |
| `references/components.md` | phases 3 to 5: mandatory families, states, naming, reuse, pushing retouches back |
| `references/audits.md`     | before every gate, after any variable deletion, to measure a contrast            |
