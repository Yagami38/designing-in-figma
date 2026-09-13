# Setup — file, pages, moodboard, layout

Everything that happens before gate 1, plus the layout of the `Design` page used from phase 4 onwards.

Names below are English defaults; translate them into the user's language, consistently across the whole file.

---

## 1. Existing or new file — a question, not an assumption

**Before any call**, ask: does a Figma file already exist for this product?

**Yes.** The user provides the link. Inventory (`audits.md` §0) on every page: page names, Sections, loose frames, variant sets, variable collections. If the file has more than three pages, it is not on the free plan or comes from a template: **ask** before merging or renaming anything. If pages carry content under other names, ask too.

**No.** Ask where to create it, recommending a **Figma folder**: the user creates the folder ("New project" in their team) and provides its link, of the form `https://www.figma.com/files/project/<projectId>`. The API cannot create a folder. Without a folder, the file goes to the root (drafts). Then, with the official `figma-create-new-file` skill loaded: `whoami` for the `planKey`, and `create_new_file` with `editorType: "design"`, `fileName: "<Product> — Design"` (in the user's language) and `projectId` if there is one.

Rename the pages — read first, rename second:

```js
const pages = figma.root.children.map((p) => ({
  name: p.name,
  id: p.id,
  children: p.children.length,
}));
return pages; // read, and ask if a page has children under another name
```

```js
const names = ["Design", "Components", "Moodboard"];
const pages = figma.root.children;
if (pages.length > 3) return { error: "more than three pages", pages: pages.map((p) => p.name) };
const ids = [];
for (let i = 0; i < 3; i++) {
  const p = pages[i] || figma.createPage();
  p.name = names[i];
  ids.push(p.id);
}
return { mutatedNodeIds: ids };
```

A new file has a single page: the other two are created. An existing file keeps its three pages, renamed.

---

## 2. `Moodboard 1` — four empty sub-sections

On the `Moodboard` page, a `Moodboard 1` Section containing four empty Sections side by side: `Design`, `Color`, `Typography`, `Avoid`. A second direction, later, would be `Moodboard 2`, never a mix inside the first.

```js
const page = figma.root.children.find((p) => p.name === "Moodboard");
await figma.setCurrentPageAsync(page);
const PAD = 80,
  GAP = 135,
  H = 1600;
const M = figma.createSection();
M.name = "Moodboard 1";
page.appendChild(M);
const subs = [
  ["Design", 2400],
  ["Color", 1200],
  ["Typography", 1200],
  ["Avoid", 1200],
];
let x = PAD;
for (const [name, w] of subs) {
  const s = figma.createSection();
  s.name = name;
  M.appendChild(s);
  s.x = x;
  s.y = PAD;
  s.resizeWithoutConstraints(w, H);
  x += w + GAP;
}
M.resizeWithoutConstraints(x - GAP + PAD, H + PAD * 2);
return { createdNodeIds: [M.id, ...M.children.map((c) => c.id)] };
```

Then **stop**: the message says what each sub-section expects — `Design`: screenshots of websites or apps whose layout the user likes; `Color`: images, palettes, photos whose hues they like; `Typography`: examples of headings and body text; `Avoid`: anything they dislike — and the turn ends. The user fills it in Figma and says when they are done.

### The `Summary`

After the analysis and the questions of phase 1, a fifth sub-section `Summary` receives a vertical auto-layout text frame with the validated decisions, in this order:

```
Colors     primary #…  (from: …) · secondary-1 #… · neutrals: warm | cool · success / error: default | #…
Type       headings: <family> <weight> · body: <family> <weight> · source: Google Fonts | <stack>
Layout     grid … · density … · radii: surfaces … / controls … · images: …
Effects    shadow: none | yes, on … · gradient: … · blur: … · borders: …
Avoid      …
Scoping    formats: mobile first then desktop | … · icons: <library> <variant> · language of names: … · language of content: …
```

Every line corresponds to a question asked and an answer received. A line without an answer does not exist: never write "default" in the user's place, except for success and error, which have a documented default in `ui-kit.md`.

---

## 3. Layout of the `Design` page

One product page = one Section. Inside each Section, the page frames in the order of the chosen formats — **mobile on the left when mobile first**, desktop on the right. Sections are aligned on `y = 0`, side by side in the order of the user's journey. An `Archive` Section closes the row, at the far right: whatever gets replaced goes there, renamed `<name> (replaced on YYYY-MM-DD)`.

| Constant      | Value | Role                                           |
| ------------- | ----: | ---------------------------------------------- |
| `PAD`         |    80 | margin between a Section's edge and its frames |
| `FRAME_GAP`   |   240 | between two frames of the same Section         |
| `SECTION_GAP` |   135 | between two neighboring Sections               |
| Mobile        |   390 | frame width                                    |
| Desktop       |  1440 | frame width                                    |

Frame naming: `<Page> — Mobile`, `<Page> — Desktop`.

Tidy the canvas — idempotent, to re-run after any page creation or change:

```js
const page = figma.root.children.find((p) => p.name === "Design");
await figma.setCurrentPageAsync(page);
const PAD = 80,
  FRAME_GAP = 240,
  SECTION_GAP = 135;
const order = ["Home", "Projects", "Project", "About", "Contact", "Archive"]; // the journey, Archive last
const sections = page.children
  .filter((c) => c.type === "SECTION")
  .sort((a, b) => {
    const ia = order.indexOf(a.name),
      ib = order.indexOf(b.name);
    return (ia < 0 ? 98 : ia) - (ib < 0 ? 98 : ib);
  });
let cursorX = 0;
const arranged = [];
for (const S of sections) {
  const kids = [...S.children].sort((a, b) => a.width - b.width); // mobile then desktop
  let x = PAD,
    maxH = 0;
  for (const k of kids) {
    k.x = x;
    k.y = PAD;
    x += k.width + FRAME_GAP;
    maxH = Math.max(maxH, k.height);
  }
  S.x = cursorX;
  S.y = 0;
  S.resizeWithoutConstraints(Math.max(x - FRAME_GAP + PAD, PAD * 2), maxH + PAD * 2);
  cursorX += S.width + SECTION_GAP;
  arranged.push(S.id);
}
return { mutatedNodeIds: arranged };
```

Then, in a separate call, `audits.md` §7. A Section does not follow its frames when they grow, and nothing flags it.

---

## 4. Inspiration — Dribbble, and nothing else

Only if the user **asks** for inspiration. A single source: Dribbble. Not Behance, not Pinterest, not Awwwards, not a gallery from memory.

**What Dribbble allows.** Its pages — search and shots alike — refuse automated reading: they come back empty. What works is the **web search restricted to the domain**: `WebSearch` with `allowed_domains: ["dribbble.com"]`, which returns each shot's title, author and link. The image is never seen; never claim to have seen it.

Method:

1. Three to five queries in English, `<product type> <style or industry>`: `architecture studio landing page`, `minimal portfolio website`, `fintech mobile app onboarding`. The user can also start from `https://dribbble.com/search/<keywords>` to find more.
2. Keep eight to twelve shots, by different authors, based on the titles and descriptions returned.
3. In the `Design` sub-section of `Moodboard 1`, **one card per shot**: title, author, clickable link, a "Look at: …" line drawn from the title or summary — never from an image that was not seen — and an empty 480 × 360 `Image` rectangle where the user pastes the visual.
4. The message lists the links and asks the user to paste the visuals they keep and delete the cards they discard. That sorting is part of the moodboard, hence of gate 1.

```js
const page = figma.root.children.find((p) => p.name === "Moodboard");
await figma.setCurrentPageAsync(page);
const design = page.findOne((n) => n.type === "SECTION" && n.name === "Design");
await figma.loadFontAsync({ family: "Inter", style: "Regular" });
await figma.loadFontAsync({ family: "Inter", style: "Bold" });
const shots = [{ title: "…", author: "…", url: "https://dribbble.com/shots/…", lookAt: "…" }];
const ids = [];
let x = 80,
  y = 80;
for (const s of shots) {
  const card = figma.createAutoLayout("VERTICAL", {
    name: `Inspiration · ${s.title}`,
    itemSpacing: 12,
  });
  card.paddingTop = card.paddingBottom = card.paddingLeft = card.paddingRight = 16;
  card.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
  design.appendChild(card);
  card.x = x;
  card.y = y;
  const img = figma.createRectangle();
  img.name = "Image";
  img.resize(480, 360);
  img.fills = [{ type: "SOLID", color: { r: 0.93, g: 0.93, b: 0.93 } }];
  card.appendChild(img);
  const title = figma.createText();
  title.fontName = { family: "Inter", style: "Bold" };
  title.characters = s.title;
  const author = figma.createText();
  author.characters = s.author;
  const link = figma.createText();
  link.characters = s.url;
  link.hyperlink = { type: "URL", value: s.url };
  const note = figma.createText();
  note.characters = `Look at: ${s.lookAt}`;
  for (const t of [title, author, link, note]) {
    card.appendChild(t);
    t.layoutSizingHorizontal = "FILL";
    t.textAutoResize = "HEIGHT";
  }
  ids.push(card.id);
  x += 480 + 32 + 32;
  if (x > 2400 - 560) {
    x = 80;
    y += 360 + 140;
  }
}
return { createdNodeIds: ids };
```

The cards are moodboard scaffolding, not a component: they live on `Moodboard`, never on `Components`, and follow no variable.

---

## 5. What the API cannot do

To list as checkboxes in the message, never to work around:

- [ ] create a Figma folder (project) — the user creates it and provides the link
- [ ] change plan, add a mode, publish a library
- [ ] read a Dribbble page — the user pastes the visuals
- [ ] validate: every gate waits for a written answer from the user
