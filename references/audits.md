# Audits — the checks to run before every gate

Every check is a **read-only** `use_figma` script that returns a list. **The list must be empty.** Whatever it returns gets fixed, then the check runs again.

Two execution rules:

- **A call separate from the fix.** A check run in the same call as the change confirms the plugin's in-memory state, not the saved state of the file.
- **One page per call.** Every script starts with `await figma.setCurrentPageAsync(page)`, once. For several pages, several calls in the same message.

Replace `<id>` with the identifier found in the inventory (§0). Scripts are Plugin API JavaScript, as passed to `use_figma`. Page names are English defaults: adjust them to the user's language.

---

## 0. Inventory of a page

To run before building anything: what already exists, under which name, and what is lying outside any Section.

```js
const page = figma.root.children.find((p) => p.name === "Design");
await figma.setCurrentPageAsync(page);
const describe = (s) => ({
  name: s.name,
  id: s.id,
  children: s.children.map((k) =>
    k.type === "SECTION" ? describe(k) : `${k.type} ${k.name} (${k.id})`,
  ),
});
const sections = page.children.filter((c) => c.type === "SECTION").map(describe);
const looseFrames = page.children
  .filter((c) => c.type !== "SECTION")
  .map((c) => ({ type: c.type, name: c.name, id: c.id }));
return { sections, looseFrames }; // looseFrames must be empty
```

On `Components`, same script with `"Components"` instead; add the list of variant sets:

```js
return page
  .findAllWithCriteria({ types: ["COMPONENT_SET", "COMPONENT"] })
  .filter((n) => n.type === "COMPONENT_SET" || n.parent.type !== "COMPONENT_SET")
  .map((n) => ({
    name: n.name,
    id: n.id,
    variants: n.type === "COMPONENT_SET" ? n.children.length : 1,
  }));
```

---

## 1. Values outside variables, texts without a style

Every solid paint, every spacing, every radius, every text size is bound to a variable; every text carries a style. This check also runs **after any variable deletion**: Figma then freezes the resolved value and removes the binding, with no warning.

```js
const R = await figma.getNodeByIdAsync("<id of the Section or frame>");
const offenders = [];
const unbound = (paints) =>
  Array.isArray(paints) &&
  paints.some(
    (p) =>
      p.type === "SOLID" && p.visible !== false && !(p.boundVariables && p.boundVariables.color),
  );
const notBound = (n, prop) =>
  typeof n[prop] === "number" && n[prop] !== 0 && !(n.boundVariables && n.boundVariables[prop]);
const visit = (n) => {
  if (n.visible === false) return;
  if ("fills" in n && unbound(n.fills)) offenders.push({ id: n.id, name: n.name, what: "fill" });
  if ("strokes" in n && unbound(n.strokes))
    offenders.push({ id: n.id, name: n.name, what: "stroke" });
  if (n.type === "TEXT" && n.getStyledTextSegments(["textStyleId"]).some((s) => !s.textStyleId)) {
    offenders.push({ id: n.id, name: n.name, what: "text without style" });
  }
  if ("layoutMode" in n && n.layoutMode !== "NONE") {
    for (const p of ["itemSpacing", "paddingTop", "paddingRight", "paddingBottom", "paddingLeft"]) {
      if (notBound(n, p)) offenders.push({ id: n.id, name: n.name, what: p });
    }
  }
  if ("cornerRadius" in n) {
    for (const p of ["topLeftRadius", "topRightRadius", "bottomLeftRadius", "bottomRightRadius"]) {
      if (notBound(n, p)) offenders.push({ id: n.id, name: n.name, what: p });
    }
  }
  if ("children" in n) n.children.forEach(visit);
};
visit(R);
return offenders; // must be empty
```

Allowed exceptions, to list by name in the script: a logo or a third-party brand artwork placed as an image, and nothing else.

---

## 2. Horizontal overflow, by absolute bounds

The only check that counts. Comparing every node to its parent lets through the children of a row that is too narrow and the absolutely positioned nodes.

```js
const F = await figma.getNodeByIdAsync("<id of the page frame>");
const B = F.absoluteBoundingBox;
const insideScroller = (n) => {
  for (let p = n.parent; p && p !== F.parent; p = p.parent) {
    if (p.overflowDirection === "HORIZONTAL" || p.overflowDirection === "BOTH") return true;
  }
  return false;
};
const outside = [];
const visit = (n) => {
  const bb = n.absoluteBoundingBox;
  if (
    bb &&
    n !== F &&
    n.visible !== false &&
    !insideScroller(n) &&
    (bb.x < B.x - 0.5 || bb.x + bb.width > B.x + B.width + 0.5)
  ) {
    outside.push({ name: n.name, id: n.id, x: Math.round(bb.x - B.x), w: Math.round(bb.width) });
  }
  if ("children" in n) n.children.forEach(visit);
};
visit(F);
return outside; // must be empty
```

A carousel is a wanted overflow: it is declared on its container (`overflowDirection = "HORIZONTAL"`, `clipsContent = false`, explicit name) and the script excludes it.

---

## 3. Variant sets clipped or stretched

A variant set must be in auto-layout, hugging both axes, and no variant may leave its bounds or have been stretched to the width of the widest one.

```js
const page = figma.root.children.find((p) => p.name === "Components");
await figma.setCurrentPageAsync(page);
const issues = [];
for (const S of page.findAllWithCriteria({ types: ["COMPONENT_SET"] })) {
  if (S.layoutMode === "NONE") issues.push({ set: S.name, what: "no auto-layout" });
  if (S.primaryAxisSizingMode !== "AUTO" || S.counterAxisSizingMode !== "AUTO")
    issues.push({ set: S.name, what: "not hugging" });
  if (S.layoutMode !== "NONE" && S.counterAxisAlignItems === "STRETCH")
    issues.push({ set: S.name, what: "variants stretched (STRETCH)" });
  for (const v of S.children) {
    if (v.x < 0 || v.y < 0 || v.x + v.width > S.width + 0.5 || v.y + v.height > S.height + 0.5) {
      issues.push({ set: S.name, variant: v.name, what: "outside the set" });
    }
    if (v.layoutSizingHorizontal === "FILL")
      issues.push({ set: S.name, variant: v.name, what: "width in FILL" });
  }
}
return issues; // must be empty
```

Remedy, after creating a set:

```js
variantSet.layoutMode = "HORIZONTAL";
variantSet.layoutWrap = "WRAP";
variantSet.primaryAxisSizingMode = "AUTO";
variantSet.counterAxisSizingMode = "AUTO";
variantSet.counterAxisAlignItems = "MIN";
variantSet.itemSpacing = 16;
variantSet.counterAxisSpacing = 16; // then bind to space/16
variantSet.paddingTop =
  variantSet.paddingRight =
  variantSet.paddingBottom =
  variantSet.paddingLeft =
    24; // then bind to space/24
for (const v of variantSet.children) {
  v.layoutSizingHorizontal = "FIXED";
  v.layoutSizingVertical = "HUG";
}
```

---

## 4. Crushed nodes

Symptom of a `FILL` inherited after an auto-layout axis change: titles at 1 px, cards shrunk to a third.

```js
const F = await figma.getNodeByIdAsync("<id>");
const crushed = [];
const visit = (n) => {
  if (n.visible === false) return;
  if (
    (n.type === "TEXT" && n.height < 8) ||
    (n.type === "FRAME" && n.layoutMode !== "NONE" && n.height < 4)
  ) {
    crushed.push({ id: n.id, name: n.name, h: n.height });
  }
  if ("children" in n) n.children.forEach(visit);
};
visit(F);
return crushed; // must be empty
```

Fix: set back to `HUG` every child in vertical `FILL` (`textAutoResize = "HEIGHT"` on texts), from the leaves up to the root.

---

## 5. A single style per text

A deleted word leaves a style range behind. Read `getStyledTextSegments`, not the first character.

```js
const F = await figma.getNodeByIdAsync("<id>");
const multi = [];
for (const t of F.findAllWithCriteria({ types: ["TEXT"] })) {
  const styles = new Set(t.getStyledTextSegments(["textStyleId"]).map((s) => s.textStyleId));
  if (styles.size > 1) multi.push({ id: t.id, name: t.name, styles: [...styles] });
}
return multi; // must be empty
```

---

## 6. A single H1 per page, no skipped level

An `Hn` style will be an `<hn>`. The accessibility audit reads headings in document order, page by page: the first one is the single `H1`, and none drops by more than one level. Order used: absolutely positioned children first (the header), then layer order, depth first. Overlays (modal, drawer) have their own hierarchy: exclude them by name.

```js
const F = await figma.getNodeByIdAsync("<id of the page frame>");
const EXCLUDED = /^(Overlay|Modal|Drawer)/;
const levels = new Map();
for (const s of await figma.getLocalTextStylesAsync()) {
  const m = s.name.match(/H([1-6])$/);
  if (m) levels.set(s.id, Number(m[1]));
}
const headings = [];
const visit = (n) => {
  if (n.visible === false || EXCLUDED.test(n.name)) return;
  if (n.type === "TEXT") {
    const level = levels.get(n.textStyleId);
    if (level) headings.push({ level, text: n.characters.slice(0, 40), id: n.id });
    return;
  }
  if (!("children" in n)) return;
  const absolute = n.children.filter((c) => c.layoutPositioning === "ABSOLUTE");
  const flow = n.children.filter((c) => c.layoutPositioning !== "ABSOLUTE");
  [...absolute, ...flow].forEach(visit);
};
visit(F);
const issues = [];
if (!headings.length || headings[0].level !== 1) issues.push("the first heading is not an H1");
if (headings.filter((h) => h.level === 1).length !== 1) issues.push("there is not exactly one H1");
for (let i = 1; i < headings.length; i++) {
  if (headings[i].level > headings[i - 1].level + 1) {
    issues.push(
      `skip ${headings[i - 1].level} → ${headings[i].level} before "${headings[i].text}"`,
    );
  }
}
return { issues, headings }; // issues must be empty
```

---

## 7. Sections: overlaps and heights

A Section does not follow its frames when they grow, and nothing flags it. After any change, its height is `PAD + height of the tallest frame + PAD` (`setup.md` §3).

```js
const parent = await figma.getNodeByIdAsync("<id of the page or of the parent Section>");
const PAD = 80;
const blocks = parent.children;
const overlaps = [];
for (let i = 0; i < blocks.length; i++) {
  for (let j = i + 1; j < blocks.length; j++) {
    const a = blocks[i],
      b = blocks[j];
    if (
      a.x < b.x + b.width &&
      b.x < a.x + a.width &&
      a.y < b.y + b.height &&
      b.y < a.y + a.height
    ) {
      overlaps.push([a.name, b.name]);
    }
  }
}
const heights = [];
for (const S of blocks.filter((c) => c.type === "SECTION")) {
  const bottom = Math.max(...S.children.map((k) => k.y + k.height)); // coordinates relative to the Section
  if (Math.abs(S.height - (bottom + PAD)) > 0.5)
    heights.push({ section: S.name, actual: S.height, expected: bottom + PAD });
  const outsideChildren = S.children
    .filter((k) => k.x < 0 || k.y < 0 || k.x + k.width > S.width || k.y + k.height > S.height)
    .map((k) => k.name);
  if (outsideChildren.length) heights.push({ section: S.name, outsideChildren });
}
return { overlaps, heights }; // both must be empty
```

---

## 8. A pass touched exactly what it targeted

Before transforming, **return what the selector matches** and read it; afterwards, compare the transformed count to the expected count. A name pattern that was too broad has already turned checkboxes into buttons.

```js
const targets = root.findAll((n) => /^Button/.test(n.name));
return targets.map((n) => ({ id: n.id, name: n.name, parent: n.parent.name })); // read before acting
```

For a pass over several variants of the same component: find every required layer **in every variant**, and stop without changing anything if one is missing.

---

## 9. Contrast between two color variables

Every new color is measured before being adopted. The script follows aliases, composites the foreground alpha over the background, and applies the WCAG formula. Thresholds: text 4.5:1; large text and controls 3:1.

```js
const names = ["color/neutral-1000", "color/neutral-0"]; // [foreground, background]
const all = await figma.variables.getLocalVariablesAsync("COLOR");
const [fg, bg] = names.map((n) => all.find((v) => v.name === n));
if (!fg || !bg) return { error: "variable not found", names };
const lin = (c) => (c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4));
const lum = ({ r, g, b }) => 0.2126 * lin(r) + 0.7152 * lin(g) + 0.0722 * lin(b);
const over = (top, base) => {
  const a = top.a === undefined ? 1 : top.a;
  return {
    r: top.r * a + base.r * (1 - a),
    g: top.g * a + base.g * (1 - a),
    b: top.b * a + base.b * (1 - a),
  };
};
const resolve = async (v) => {
  let val = Object.values(v.valuesByMode)[0]; // a single mode
  while (val && val.type === "VARIABLE_ALIAS") {
    const target = await figma.variables.getVariableByIdAsync(val.id);
    val = Object.values(target.valuesByMode)[0];
  }
  return val;
};
const f = await resolve(fg),
  b = await resolve(bg);
const L1 = lum(over(f, b)),
  L2 = lum(b);
const ratio = (Math.max(L1, L2) + 0.05) / (Math.min(L1, L2) + 0.05);
return { ratio: Math.round(ratio * 100) / 100, text: ratio >= 4.5, largeText: ratio >= 3 };
```

---

## 10. The eye

No script replaces the screenshot. After every visible change: `get_screenshot` or `await node.screenshot()`, and **look** — a duplicated text, a cropped badge, an empty variant are neither an overflow, nor a bare value, nor a missing style. Audits catch what has already been met; the screenshot catches the rest.
