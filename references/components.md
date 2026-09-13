# Components — families, states, naming, reuse

Components live on the `Components` page, **one Section per family**, after the `UI Kit` Section. They are built once the UI kit is done, and gate 2 validates them **together with** the UI kit. Masters are never on `Design`: a page is nothing but an assembly of instances.

Example names are English defaults; in another language, same scheme translated, one language across the file.

---

## 1. What a product always has

For a **website**:

| Family       | Mandatory components                                                                                                             |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| `Navigation` | header (`State = Default \| Menu open` on mobile), breadcrumb if the site has levels                                             |
| `Footer`     | full footer: links, legal, newsletter if planned                                                                                 |
| `Form`       | contact form (`State = Default \| Error \| Success`), newsletter (same)                                                          |
| `Card`       | one card per content type: project, article, product, team member…                                                               |
| `Section`    | **every section of the product**, one per page block: hero, card list, call-to-action band, testimonials, FAQ, rich content, 404 |

For an **app**:

| Family       | Mandatory components                                                          |
| ------------ | ----------------------------------------------------------------------------- |
| `Navigation` | top bar, tab bar or drawer, back                                              |
| `List`       | list row (`State = Default \| Selected`), group header                        |
| `Card`       | one per content type                                                          |
| `Overlay`    | modal, bottom sheet, toast (`Tone = Neutral \| Success \| Error`)             |
| `Screen`     | **every screen of the product**; empty, loading and error states are variants |

The atoms (`Button`, `Input`…) come from the UI kit and are never redrawn inside a component: instances are placed.

---

## 2. States are variants

A mockup shows the nominal case by default. The other cases are drawn as **variant properties** on the component concerned, never as duplicated frames:

```
Section / Project grid    Breakpoint = Mobile | Desktop
                          State      = Default | Empty | Loading | Error
Form / Contact            Breakpoint = Mobile | Desktop
                          State      = Default | Error | Success
Navigation / Header       Breakpoint = Mobile | Desktop
                          State      = Default | Menu open
```

Mandatory:

- `Empty`, `Loading`, `Error` on every section or screen that displays data (list, grid, results, dashboard).
- `Error` and `Success` on every form.
- `Hover`, `Focus`, `Disabled` on every control — they come from the UI kit.

**A state removes as much as it adds.** An empty grid does not show its sort filters; a submitted form no longer shows its button. A state that stacks a message on top of the nominal state leaves controls that lead nowhere.

`Breakpoint` exists only if two formats were chosen at scoping. The two variants differ in **structure** (stacked columns, collapsed menu), not only in width: that is what justifies two variants rather than one resizable component that would hide layers.

---

## 3. Naming

```
<Family> / <Name>            Section / Hero · Card / Project · Form / Contact
```

- The name states the **role**, not the appearance nor the page: `Card / Project`, not `Blue card` nor `Home card`.
- Variant property values are whole words: `Default`, not `def`.
- Inner layers are named by role (`Title`, `Excerpt`, `Cover`, `Actions`): that is what instance overrides and scripts find.

---

## 4. Building a component

1. **Search before creating.** List the existing `COMPONENT_SET`s; if a variant or an override covers the need, there is no new component.
2. Build the main variant (the first chosen format) from atom instances, everything in auto-layout, every value bound.
3. Derive the other `Breakpoint` by **cloning** that variant, never by rebuilding: clone, restructure at the original width, shrink, then set back to `HUG` everything an axis change left in `FILL` (a title crushed to 1 px is the symptom).
4. Add the states by cloning the `Default` variant and removing or replacing what no longer applies.
5. Tidy the set: auto-layout, wrap, hug, `space/24` padding, `space/16` gap, variants in `FIXED` width (`audits.md` §3).
6. Screenshot, and audits §1 to §5 on the family's Section, in a separate call.

**Fix the master, never the instance.** A fix made in an instance creates an override that survives component updates and drifts from the other pages. What varies from one page to another (a title, a number) is a wanted override; what varies structurally is another component.

---

## 5. Reusing while deriving the pages

In phase 5, every page is composed **first** with what exists, in this order:

1. an instance as is;
2. an instance with content overrides (texts, images, icon);
3. a new **variant** of an existing component (new state, minor layout change);
4. a new component — only if the structure really differs, and it joins its family on `Components` before being placed.

Two components that differ only by their content get merged: swap the instances to the equivalent variant (`swapComponent`), carry the overrides across, read what was actually overridden in `instance.overrides`, and delete the old one only at **zero remaining instances**.

---

## 6. What goes back into the components after gate 3

When the user has retouched the first mockup directly in Figma, their retouches sit in **instances** (overrides) or in detached frames. Before deriving the other pages:

1. List, on the validated page, the instances carrying `overrides` and the frames that are no longer instances.
2. For each difference, decide with the user whether it is **local** (a content override) or **systemic** (a fix to the component, a new scale value). Nothing is inferred silently.
3. Push the systemic ones into the master, then `resetOverrides()` on the instances concerned; bring the detached frames back as instances.

Without this pass, the other pages would not have the retouches, and the first page would drift from its own system.
