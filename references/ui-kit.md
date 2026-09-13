# UI kit — variables, styles, icons, atoms

The UI kit lives on the `Components` page, in a `UI Kit` Section made of five sub-sections: `Colors`, `Typography`, `Spacing and radii`, `Icons`, `Atoms`. It is built **after** gate 1 (moodboard validated, summary written) and **before** the components. No value is decided here: they all come from the moodboard's `Summary`.

Names are English defaults. If the user works in another language, the scheme is the same, translated (`couleur/primaire`, `Bouton`, `Champ`) — one language across the whole file.

---

## 1. Variables — a single layer, by role and by scale

The free plan offers **only one mode per collection**: no Light/Dark or Mobile/Desktop through modes. Every variable has one value, period. Differences between formats are handled variant by variant, by binding another value of the same scale.

Four collections. What follows is **the complete list**: remove from it, never add without a written decision in the moodboard's summary.

### `color` — minimal

| Variable                                  | Role                                                   |
| ----------------------------------------- | ------------------------------------------------------ |
| `color/primary`                           | the single accent: primary buttons, links, focus       |
| `color/primary-hover`                     | the same, darkened or lightened by 8 to 12 %           |
| `color/secondary-1` … `color/secondary-3` | **three at most**, only if the moodboard requires them |
| `color/neutral-0` … `color/neutral-1000`  | gradient from white to black: 0, 100, 200 … 900, 1000  |
| `color/success`, `color/success-bg`       | confirmation; the `-bg` is the same hue at 12 % alpha  |
| `color/error`, `color/error-bg`           | error, out of stock, deletion; same                    |

Rules:

- **Text and backgrounds come from the neutrals**, never from the secondaries. A secondary is an accent for a section or an illustration.
- **Alpha lives in the variable** (`{ r, g, b, a }`), never in `paint.opacity` or `node.opacity`: an opacity set on the paint after binding is ignored at render time.
- **Contrast measured before adopting**: text ≥ 4.5:1, large text and controls ≥ 3:1 (`audits.md` §9). A candidate at 4.4:1 is rejected.
- A gradient, a shadow, a blur exist only if the moodboard's summary validated them; they then become a named **effect style**, never a value set by hand.

### `space` — the scale, and nothing else

```
4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128
```

Multiples of 4 up to 40, of 8 up to 80, of 16 beyond. A need for 44 or 90 px has no dedicated answer: pick one of the two neighbors. Every `itemSpacing`, every `padding`, every gap between blocks is bound to one of these variables.

### `radius`

```
radius/0, radius/4, radius/8, radius/12, radius/16, radius/full (999)
```

Two radii per product are almost always enough: one for surfaces (cards, images, panels), one for controls (buttons, fields). `full` is reserved for pills and toggles.

### `type` — sizes, line heights, families

```
type/size-12  type/size-16  type/size-20  type/size-24
type/size-32  type/size-40  type/size-48  type/size-64

type/line-16  type/line-24  type/line-28  type/line-32
type/line-40  type/line-48  type/line-56  type/line-72

font/heading  font/body            (STRING: the family, from the moodboard)
font/weight-heading  font/weight-body   (STRING: the style, e.g. "Bold", "Regular")
```

**Fonts: Google Fonts by default**, checked as available in Figma (`listAvailableFontsAsync`) before any binding. Exception: a stack that recommends its own fonts — Shopify and its font library, a corporate brand guide — wins.

---

## 2. Text styles

Seven roles, bound to variables (size, line height, family, weight), never raw values. **Two groups when two formats were chosen** — the free plan has no modes, so `Mobile/H1` and `Desktop/H1` are two styles carrying the same role.

| Role      | Mobile (size / line height)       | Desktop | Use                           |
| --------- | --------------------------------- | ------- | ----------------------------- |
| `H1`      | 40 / 48                           | 64 / 72 | page title — **one per page** |
| `H2`      | 32 / 40                           | 48 / 56 | section title                 |
| `H3`      | 24 / 32                           | 32 / 40 | card title, sub-block title   |
| `H4`      | 20 / 28                           | 24 / 32 | subtitle, form heading        |
| `Body`    | 16 / 24                           | 16 / 24 | running text                  |
| `Small`   | 12 / 16                           | 12 / 16 | notes, help text, metadata    |
| `Caption` | 12 / 16, uppercase, +4 % tracking | same    | labels, overlines             |

An `Hn` style **is** an `<hn>`: whatever looks like a heading without being one — a price, a key figure, a logo — does not carry a heading style. If the product needs it, an eighth `Emphasis` style, same rendering as `H4` and no heading role, is added with a line in the `Summary`. Body text never goes below 16; 12 is reserved for notes.

---

## 3. Icons — imported, never drawn

**Scoping question:** which library? Recommend **Material Design Icons** (Google). Acceptable alternatives if the user prefers them: Lucide, Phosphor, Heroicons. One library per product, one variant (`filled` or `outlined`, not both).

A hand-drawn glyph is debt: off the optical grid, unfindable in code, redone on every project. Even "to feel more personal".

Import recipe, glyph by glyph:

```bash
# Material Design Icons, filled variant
curl -sfL -o <name>.svg "https://cdn.jsdelivr.net/npm/@material-design-icons/svg/filled/<name>.svg"
# Lucide
curl -sfL -o <name>.svg "https://cdn.jsdelivr.net/npm/lucide-static/icons/<name>.svg"
```

Then `upload_assets` (`count` = number of files, `Content-Type: image/svg+xml`): each SVG lands as a vector tree on the current page. A `use_figma` script turns each tree into a 24 × 24 `COMPONENT`, vector constraints set to `SCALE`, fill bound to `color/neutral-1000`, then `figma.combineAsVariants` into an `Icon` set with the property `Name = <exact library name>`. Three sizes in use: 16 inside a button, 20 inside a control, 24 in navigation.

**The icon takes the color of the text it sits next to**, by reusing that text's variable through an instance override.

---

## 4. Atoms — the components every product has

Every atom is a variant set. **The set is in auto-layout, wrapping, hugging both axes, with a `space/24` padding and a `space/16` gap**, so that no variant is clipped or stretched; every variant keeps a `FIXED` width (`audits.md` §3). The mandatory states are those of the table; none is removed.

| Atom       | Properties                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| `Button`   | `Style = Primary \| Secondary \| Ghost` · `Size = M \| S` · `State = Default \| Hover \| Focus \| Disabled` |
| `Link`     | `State = Default \| Hover \| Focus`                                                                         |
| `Input`    | `Type = Text \| Textarea \| Select` · `State = Default \| Focus \| Filled \| Error \| Success \| Disabled`  |
| `Checkbox` | `Checked = Off \| On` · `State = Default \| Focus \| Disabled`                                              |
| `Radio`    | same                                                                                                        |
| `Toggle`   | same                                                                                                        |
| `Tag`      | `Tone = Neutral \| Primary \| Success \| Error`                                                             |
| `Message`  | `Tone = Success \| Error` — the help text under a field, the banner of a form                               |

Recipes that avoid redrawing everything:

- `Button`: padding `space/12` × `space/24` in M, `space/8` × `space/16` in S; inner gap `space/8`; the controls radius; label in `Body` (M) or `Small` (S). An `Icon` slot **visible in the master**, hidden by override on the instances that do not want it — a child hidden in the master does not exist for its instances.
- `Hover`: `color/primary-hover` background for Primary; `color/neutral-100` for Secondary and Ghost. `Focus`: a 2 px ring in `color/primary` on the outside, never a removed outline. `Disabled`: `color/neutral-300` on `color/neutral-100`, cursor with no effect.
- `Input`: `color/neutral-0` background, 1 px `color/neutral-300` border, `Focus` in `color/primary`, `Error` in `color/error` with a `Message`, `Success` in `color/success`. The value fills the width: an instance stretches inside its parent with no setting. `Textarea` = same shell, text anchored at the top, height `space/128`.

---

## 5. UI kit exit check

Before moving on to the components — in a call separate from any write:

- [ ] `audits.md` §1 on the `UI Kit` Section: no value outside a variable, no text without a style
- [ ] `audits.md` §3: no variant set clipped or stretched
- [ ] `audits.md` §9: contrast of `color/neutral-1000` on `neutral-0`, of `neutral-0` on `primary`, of `success` and `error` on `neutral-0`
- [ ] Screenshot of the `UI Kit` Section looked at: every variant visible, legible, aligned
