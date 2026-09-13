# designing-in-figma

A skill for Claude Code that encodes a method for designing websites and apps in Figma through the Figma MCP server: scoping questions, a moodboard analyzed and validated choice by choice, a minimalist variable-based UI kit, components with all their states, and a first mockup validated before the other pages are derived. It is written for Figma's **free plan** (three pages per file, one mode per collection, no team library).

The skill asks the user which language to use for page, variable and component names, and applies it consistently across the file.

## Requirements

- [Claude Code](https://claude.com/claude-code) with the Figma connector enabled (the claude.ai Figma MCP server).
- A Figma account, free or paid plan.

## Installation

For every project on the machine:

```bash
git clone https://github.com/Yagami38/designing-in-figma.git ~/.claude/skills/designing-in-figma
```

For a single project, from its root:

```bash
git clone https://github.com/Yagami38/designing-in-figma.git .claude/skills/designing-in-figma
```

Update with `git pull` inside the folder. Claude Code loads the skill as soon as a conversation is about designing in Figma; it can also be invoked with `/designing-in-figma`.

## What the repository contains

| File                       | Content                                                                                                                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SKILL.md`                 | the method: free-plan constraints, the three validation gates, scoping, the five phases, the non-negotiable rules, the rationalizations to recognize, the mandatory checklist before every gate |
| `references/setup.md`      | existing or new file (Figma folder recommended), the three pages, the `Moodboard 1` Section and its `Summary`, the `Design` page layout, Dribbble inspiration                                   |
| `references/ui-kit.md`     | the exact inventory of variables (minimal colors, 4 → 128 spacing scale, radii, typography), the seven text styles, icon import, the atoms and their states                                     |
| `references/components.md` | the mandatory component families for a website and for an app, states as variants, naming, the reuse order, pushing the user's retouches back into the masters                                  |
| `references/audits.md`     | ten read-only check scripts, to run before every gate                                                                                                                                           |

## The method in brief

1. **Scoping** — before any call: existing file or folder to create, website or app, formats (mobile first recommended), icon library (Material Design recommended, never a drawn icon), languages, stack and assets.
2. **Moodboard** — three pages `Design`, `Components`, `Moodboard`; an empty `Moodboard 1` Section (Design, Color, Typography, Avoid) that the user fills in; Claude analyzes it and validates every choice with a question, effects included; the decisions are written into a `Summary`. **Gate 1.**
3. **UI kit** — single-layer variables: one primary, up to three secondaries, the neutrals, success and error; spacing on the scale `4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128`; text styles bound to variables; imported icons; atoms with Hover, Focus, Disabled, Error, Success.
4. **Components** — navigation, footer, forms, cards, then every section of the product, with Empty, Loading and Error as variants. **Gate 2.**
5. **First mockup** — a single page, assembled from instances; the user retouches it directly in Figma and validates it; the retouches are pushed back into the masters. **Gate 3.**
6. **Deriving** — the other pages with as many existing components as possible, then the second format.

Every value — color, spacing, padding, radius, size and line height, family and weight — is a variable. Nothing is inferred silently: a typeface, a color, an effect are questions asked to the user.

## Known limits

- The Figma API cannot create a folder: the user creates it and provides the link.
- Dribbble refuses automated reading of its pages; the skill relies on domain-restricted web search and creates inspiration cards (title, author, link, image slot) that the user completes.

## Contributing

A pitfall met in real use is added to the reference concerned as: observed symptom, cause, remedy, date. Issues and pull requests are welcome.

## License

MIT — see `LICENSE`.
