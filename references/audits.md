# Audits — les contrôles à passer avant chaque porte

Chaque contrôle est un script `use_figma` **en lecture seule** qui renvoie une liste. **La liste doit être vide.** Ce qu'il renvoie se corrige, puis le contrôle se relance.

Deux règles d'exécution :

- **Un appel séparé de la correction.** Un contrôle lancé dans le même appel que la modification confirme l'état en mémoire du plugin, pas l'état enregistré du fichier.
- **Une page par appel.** Chaque script commence par `await figma.setCurrentPageAsync(page)`, une seule fois. Pour plusieurs pages, plusieurs appels dans un même message.

Remplacer `<id>` par l'identifiant relevé dans l'inventaire (§0). Les scripts sont en JavaScript de l'API Plugin, tels qu'on les passe à `use_figma`.

---

## 0. Inventaire d'une page

À passer avant de construire quoi que ce soit : ce qui existe déjà, sous quel nom, et ce qui traîne hors de toute Section.

```js
const page = figma.root.children.find((p) => p.name === "Design");
await figma.setCurrentPageAsync(page);
const lister = (s) => ({
  name: s.name,
  id: s.id,
  enfants: s.children.map((k) =>
    k.type === "SECTION" ? lister(k) : `${k.type} ${k.name} (${k.id})`,
  ),
});
const sections = page.children.filter((c) => c.type === "SECTION").map(lister);
const framesLaches = page.children
  .filter((c) => c.type !== "SECTION")
  .map((c) => ({ type: c.type, name: c.name, id: c.id }));
return { sections, framesLaches }; // framesLaches doit être vide
```

Sur `Composant`, même script en remplaçant `"Design"` ; y ajouter la liste des jeux de variantes :

```js
return page
  .findAllWithCriteria({ types: ["COMPONENT_SET", "COMPONENT"] })
  .filter((n) => n.type === "COMPONENT_SET" || n.parent.type !== "COMPONENT_SET")
  .map((n) => ({
    name: n.name,
    id: n.id,
    variantes: n.type === "COMPONENT_SET" ? n.children.length : 1,
  }));
```

---

## 1. Valeurs hors variable, textes sans style

Toute peinture unie, tout espacement, tout rayon, toute taille de texte est lié à une variable ; tout texte porte un style. Ce contrôle se repasse aussi **après toute suppression de variable** : Figma fige alors la valeur résolue et retire la liaison, sans alerte.

```js
const R = await figma.getNodeByIdAsync("<id de la Section ou du frame>");
const fautifs = [];
const libre = (paints) =>
  Array.isArray(paints) &&
  paints.some(
    (p) =>
      p.type === "SOLID" && p.visible !== false && !(p.boundVariables && p.boundVariables.color),
  );
const nonLie = (n, prop) =>
  typeof n[prop] === "number" && n[prop] !== 0 && !(n.boundVariables && n.boundVariables[prop]);
const v = (n) => {
  if (n.visible === false) return;
  if ("fills" in n && libre(n.fills)) fautifs.push({ id: n.id, name: n.name, quoi: "fill" });
  if ("strokes" in n && libre(n.strokes)) fautifs.push({ id: n.id, name: n.name, quoi: "stroke" });
  if (n.type === "TEXT" && n.getStyledTextSegments(["textStyleId"]).some((s) => !s.textStyleId)) {
    fautifs.push({ id: n.id, name: n.name, quoi: "texte sans style" });
  }
  if ("layoutMode" in n && n.layoutMode !== "NONE") {
    for (const p of ["itemSpacing", "paddingTop", "paddingRight", "paddingBottom", "paddingLeft"]) {
      if (nonLie(n, p)) fautifs.push({ id: n.id, name: n.name, quoi: p });
    }
  }
  if ("cornerRadius" in n) {
    for (const p of ["topLeftRadius", "topRightRadius", "bottomLeftRadius", "bottomRightRadius"]) {
      if (nonLie(n, p)) fautifs.push({ id: n.id, name: n.name, quoi: p });
    }
  }
  if ("children" in n) n.children.forEach(v);
};
v(R);
return fautifs; // doit être vide
```

Exceptions admises, à lister par nom dans le script : un logo ou un artwork de marque tierce posé en image, et rien d'autre.

---

## 2. Débordement horizontal, par bornes absolues

Le seul contrôle qui fasse foi. Comparer chaque nœud à son parent laisse passer les enfants d'une rangée trop étroite et les nœuds en position absolue.

```js
const F = await figma.getNodeByIdAsync("<id du frame de page>");
const B = F.absoluteBoundingBox;
const dansUnDefilement = (n) => {
  for (let p = n.parent; p && p !== F.parent; p = p.parent) {
    if (p.overflowDirection === "HORIZONTAL" || p.overflowDirection === "BOTH") return true;
  }
  return false;
};
const hors = [];
const v = (n) => {
  const bb = n.absoluteBoundingBox;
  if (
    bb &&
    n !== F &&
    n.visible !== false &&
    !dansUnDefilement(n) &&
    (bb.x < B.x - 0.5 || bb.x + bb.width > B.x + B.width + 0.5)
  ) {
    hors.push({ name: n.name, id: n.id, x: Math.round(bb.x - B.x), w: Math.round(bb.width) });
  }
  if ("children" in n) n.children.forEach(v);
};
v(F);
return hors; // doit être vide
```

Un carrousel est un débordement voulu : il se déclare sur son conteneur (`overflowDirection = "HORIZONTAL"`, `clipsContent = false`, nom explicite) et le script l'exclut.

---

## 3. Jeux de variantes coupés ou étirés

Un jeu de variantes doit être en auto-layout, hug sur les deux axes, et aucune variante ne doit sortir de ses bornes ni avoir été étirée à la largeur de la plus large.

```js
const page = figma.root.children.find((p) => p.name === "Composant");
await figma.setCurrentPageAsync(page);
const defauts = [];
for (const S of page.findAllWithCriteria({ types: ["COMPONENT_SET"] })) {
  if (S.layoutMode === "NONE") defauts.push({ set: S.name, quoi: "pas d'auto-layout" });
  if (S.primaryAxisSizingMode !== "AUTO" || S.counterAxisSizingMode !== "AUTO")
    defauts.push({ set: S.name, quoi: "pas en hug" });
  const largeurs = new Set(S.children.map((v) => Math.round(v.width)));
  if (S.layoutMode !== "NONE" && S.counterAxisAlignItems === "STRETCH")
    defauts.push({ set: S.name, quoi: "variantes étirées (STRETCH)" });
  for (const v of S.children) {
    if (v.x < 0 || v.y < 0 || v.x + v.width > S.width + 0.5 || v.y + v.height > S.height + 0.5) {
      defauts.push({ set: S.name, variante: v.name, quoi: "hors du jeu" });
    }
    if (v.layoutSizingHorizontal === "FILL")
      defauts.push({ set: S.name, variante: v.name, quoi: "largeur en FILL" });
  }
}
return defauts; // doit être vide
```

Remède, après création d'un jeu :

```js
jeu.layoutMode = "HORIZONTAL";
jeu.layoutWrap = "WRAP";
jeu.primaryAxisSizingMode = "AUTO";
jeu.counterAxisSizingMode = "AUTO";
jeu.counterAxisAlignItems = "MIN";
jeu.itemSpacing = 16;
jeu.counterAxisSpacing = 16; // puis lier à space/16
jeu.paddingTop = jeu.paddingRight = jeu.paddingBottom = jeu.paddingLeft = 24; // puis lier à space/24
for (const v of jeu.children) {
  v.layoutSizingHorizontal = "FIXED";
  v.layoutSizingVertical = "HUG";
}
```

---

## 4. Nœuds écrasés

Symptôme d'un `FILL` hérité après un changement d'axe d'auto-layout : titres à 1 px, cartes réduites au tiers.

```js
const F = await figma.getNodeByIdAsync("<id>");
const ecrases = [];
const v = (n) => {
  if (n.visible === false) return;
  if (
    (n.type === "TEXT" && n.height < 8) ||
    (n.type === "FRAME" && n.layoutMode !== "NONE" && n.height < 4)
  ) {
    ecrases.push({ id: n.id, name: n.name, h: n.height });
  }
  if ("children" in n) n.children.forEach(v);
};
v(F);
return ecrases; // doit être vide
```

Réparation : repasser en `HUG` tout enfant en `FILL` vertical (`textAutoResize = "HEIGHT"` sur les textes), des feuilles vers la racine.

---

## 5. Un seul style par texte

Un mot supprimé laisse une plage de style derrière lui. Lire `getStyledTextSegments`, pas le premier caractère.

```js
const F = await figma.getNodeByIdAsync("<id>");
const multi = [];
for (const t of F.findAllWithCriteria({ types: ["TEXT"] })) {
  const styles = new Set(t.getStyledTextSegments(["textStyleId"]).map((s) => s.textStyleId));
  if (styles.size > 1) multi.push({ id: t.id, name: t.name, styles: [...styles] });
}
return multi; // doit être vide
```

---

## 6. Un seul H1 par page, aucun niveau sauté

Un style `Hn` sera un `<hn>`. L'audit d'accessibilité lit les titres dans l'ordre du document, page par page : le premier est l'unique `H1`, et aucun ne descend de plus d'un niveau. Ordre retenu : les enfants en position absolue d'abord (l'en-tête), puis l'ordre des calques, en profondeur. Les surcouches (modale, tiroir) ont leur propre hiérarchie : les exclure par nom.

```js
const F = await figma.getNodeByIdAsync("<id du frame de page>");
const EXCLUS = /^(Overlay|Modal|Drawer|Surcouche|Modale|Tiroir)/;
const niveaux = new Map();
for (const s of await figma.getLocalTextStylesAsync()) {
  const m = s.name.match(/H([1-6])$/);
  if (m) niveaux.set(s.id, Number(m[1]));
}
const titres = [];
const v = (n) => {
  if (n.visible === false || EXCLUS.test(n.name)) return;
  if (n.type === "TEXT") {
    const niv = niveaux.get(n.textStyleId);
    if (niv) titres.push({ niveau: niv, texte: n.characters.slice(0, 40), id: n.id });
    return;
  }
  if (!("children" in n)) return;
  const absolus = n.children.filter((c) => c.layoutPositioning === "ABSOLUTE");
  const flux = n.children.filter((c) => c.layoutPositioning !== "ABSOLUTE");
  [...absolus, ...flux].forEach(v);
};
v(F);
const defauts = [];
if (!titres.length || titres[0].niveau !== 1) defauts.push("le premier titre n'est pas un H1");
if (titres.filter((t) => t.niveau === 1).length !== 1)
  defauts.push("il n'y a pas exactement un H1");
for (let i = 1; i < titres.length; i++) {
  if (titres[i].niveau > titres[i - 1].niveau + 1) {
    defauts.push(`saut ${titres[i - 1].niveau} → ${titres[i].niveau} avant « ${titres[i].texte} »`);
  }
}
return { defauts, titres }; // defauts doit être vide
```

---

## 7. Sections : recouvrements et hauteurs

Une Section ne suit pas ses frames quand ils grandissent, et rien ne le signale. Après toute modification, sa hauteur vaut `PAD + hauteur du plus haut frame + PAD` (`setup.md` §3).

```js
const parent = await figma.getNodeByIdAsync("<id de la page ou de la Section parente>");
const PAD = 80;
const blocs = parent.children;
const recouvrements = [];
for (let i = 0; i < blocs.length; i++) {
  for (let j = i + 1; j < blocs.length; j++) {
    const a = blocs[i],
      b = blocs[j];
    if (
      a.x < b.x + b.width &&
      b.x < a.x + a.width &&
      a.y < b.y + b.height &&
      b.y < a.y + a.height
    ) {
      recouvrements.push([a.name, b.name]);
    }
  }
}
const hauteurs = [];
for (const S of blocs.filter((c) => c.type === "SECTION")) {
  const bas = Math.max(...S.children.map((k) => k.y + k.height)); // coordonnées relatives à la Section
  if (Math.abs(S.height - (bas + PAD)) > 0.5)
    hauteurs.push({ section: S.name, actuelle: S.height, attendue: bas + PAD });
  const sortants = S.children
    .filter((k) => k.x < 0 || k.y < 0 || k.x + k.width > S.width || k.y + k.height > S.height)
    .map((k) => k.name);
  if (sortants.length) hauteurs.push({ section: S.name, sortants });
}
return { recouvrements, hauteurs }; // les deux doivent être vides
```

---

## 8. Une passe a touché exactement ce qu'elle visait

Avant de transformer, **retourner ce que le sélecteur ramène** et le lire ; après, comparer le compte transformé au compte attendu. Un motif de nom trop large a déjà converti des cases à cocher en boutons.

```js
const cibles = racine.findAll((n) => /^Button/.test(n.name));
return cibles.map((n) => ({ id: n.id, name: n.name, parent: n.parent.name })); // lire avant d'agir
```

Pour une passe sur plusieurs variantes d'un même composant : trouver chaque calque nécessaire **dans chaque variante**, et s'arrêter sans rien modifier s'il en manque un.

---

## 9. Contraste entre deux variables de couleur

Toute couleur nouvelle se mesure avant d'être adoptée. Le script suit les alias, compose l'alpha du premier plan sur le fond, et applique la formule WCAG. Seuil texte : 4,5:1 ; grand texte et contrôles : 3:1.

```js
const noms = ["color/neutral-1000", "color/neutral-0"]; // [premier plan, fond]
const toutes = await figma.variables.getLocalVariablesAsync("COLOR");
const [fg, bg] = noms.map((n) => toutes.find((v) => v.name === n));
if (!fg || !bg) return { erreur: "variable introuvable", noms };
const lin = (c) => (c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4));
const lum = ({ r, g, b }) => 0.2126 * lin(r) + 0.7152 * lin(g) + 0.0722 * lin(b);
const sur = (haut, bas) => {
  const a = haut.a === undefined ? 1 : haut.a;
  return {
    r: haut.r * a + bas.r * (1 - a),
    g: haut.g * a + bas.g * (1 - a),
    b: haut.b * a + bas.b * (1 - a),
  };
};
const resoudre = async (v) => {
  let val = Object.values(v.valuesByMode)[0]; // un seul mode
  while (val && val.type === "VARIABLE_ALIAS") {
    const cible = await figma.variables.getVariableByIdAsync(val.id);
    val = Object.values(cible.valuesByMode)[0];
  }
  return val;
};
const f = await resoudre(fg),
  b = await resoudre(bg);
const L1 = lum(sur(f, b)),
  L2 = lum(b);
const ratio = (Math.max(L1, L2) + 0.05) / (Math.min(L1, L2) + 0.05);
return { ratio: Math.round(ratio * 100) / 100, texte: ratio >= 4.5, grandTexte: ratio >= 3 };
```

---

## 10. Le regard

Aucun script ne remplace la capture. Après chaque changement visible : `get_screenshot` ou `await node.screenshot()`, et **regarder** — un texte en double, un badge rogné, une variante vide ne sont ni un débordement, ni une valeur en dur, ni un style manquant. Les audits attrapent ce qu'on a déjà rencontré ; la capture attrape le reste.
