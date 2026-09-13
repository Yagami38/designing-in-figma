# Setup — fichier, pages, moodboard, disposition

Tout ce qui se passe avant la porte 1, plus la disposition de la page `Design` qui sert à partir de la phase 4.

---

## 1. Fichier existant ou nouveau — une question, pas une hypothèse

**Avant tout appel**, demander : un fichier Figma existe-t-il pour ce produit ?

**Oui.** L'utilisateur donne le lien. Inventaire (`audits.md` §0) sur chaque page : noms des pages, Sections, frames lâches, jeux de variantes, collections de variables. Si le fichier a plus de trois pages, il n'est pas sur le plan gratuit ou vient d'un gabarit : **demander** avant de fusionner ou renommer quoi que ce soit. Si des pages portent du contenu sous d'autres noms, demander aussi.

**Non.** Demander où le créer, en recommandant un **dossier Figma** : l'utilisateur crée le dossier (« Nouveau projet » dans son équipe) et donne son lien, de la forme `https://www.figma.com/files/project/<projectId>`. L'API ne crée pas de dossier. Sans dossier, le fichier va à la racine (brouillons). Puis, avec le skill officiel `figma-create-new-file` chargé : `whoami` pour le `planKey`, et `create_new_file` avec `editorType: "design"`, `fileName: "<Produit> — Design"` (dans la langue de l'utilisateur) et `projectId` s'il y en a un.

Renommer les pages — lecture d'abord, renommage ensuite :

```js
const pages = figma.root.children.map((p) => ({
  name: p.name,
  id: p.id,
  enfants: p.children.length,
}));
return pages; // lire, et demander si une page a des enfants sous un autre nom
```

```js
const noms = ["Design", "Composant", "Moodboard"];
const pages = figma.root.children;
if (pages.length > 3) return { erreur: "plus de trois pages", pages: pages.map((p) => p.name) };
const ids = [];
for (let i = 0; i < 3; i++) {
  const p = pages[i] || figma.createPage();
  p.name = noms[i];
  ids.push(p.id);
}
return { mutatedNodeIds: ids };
```

Un fichier neuf n'a qu'une page : les deux autres sont créées. Un fichier existant garde ses trois pages, renommées.

---

## 2. `Moodboard 1` — quatre sous-sections vides

Sur la page `Moodboard`, une Section `Moodboard 1` qui contient quatre Sections vides, côte à côte : `Design`, `Couleur`, `Typo`, `À éviter`. Une deuxième direction, plus tard, serait `Moodboard 2`, jamais un mélange dans la première.

```js
const page = figma.root.children.find((p) => p.name === "Moodboard");
await figma.setCurrentPageAsync(page);
const PAD = 80,
  ECART = 135,
  H = 1600;
const M = figma.createSection();
M.name = "Moodboard 1";
page.appendChild(M);
const sous = [
  ["Design", 2400],
  ["Couleur", 1200],
  ["Typo", 1200],
  ["À éviter", 1200],
];
let x = PAD;
for (const [nom, w] of sous) {
  const s = figma.createSection();
  s.name = nom;
  M.appendChild(s);
  s.x = x;
  s.y = PAD;
  s.resizeWithoutConstraints(w, H);
  x += w + ECART;
}
M.resizeWithoutConstraints(x - ECART + PAD, H + PAD * 2);
return { createdNodeIds: [M.id, ...M.children.map((c) => c.id)] };
```

Puis **s'arrêter** : le message dit ce que chaque sous-section attend — `Design` : captures de sites ou d'apps dont la mise en page plaît ; `Couleur` : images, palettes, photos dont les teintes plaisent ; `Typo` : exemples de titres et de textes ; `À éviter` : tout ce qui déplaît — et le tour se termine. L'utilisateur remplit dans Figma et prévient quand il a fini.

### La `Synthèse`

Après l'analyse et les questions de la phase 1, une cinquième sous-section `Synthèse` reçoit un cadre texte en auto-layout vertical avec les décisions validées, dans cet ordre :

```
Couleurs   primary #…  (vient de : …) · secondary-1 #… · neutres : chauds | froids · succès / erreur : par défaut | #…
Typo       titres : <famille> <graisse> · texte : <famille> <graisse> · source : Google Fonts | <stack>
Mise en page   grille … · densité … · rayons : surfaces … / contrôles … · images : …
Effets     ombre : non | oui, sur … · dégradé : … · flou : … · bordures : …
À éviter   …
Cadrage    formats : mobile first puis desktop | … · icônes : <librairie> <variante> · langue des noms : … · langue du contenu : …
```

Chaque ligne correspond à une question posée et à une réponse reçue. Une ligne sans réponse n'existe pas : on n'écrit pas « par défaut » à la place de l'utilisateur, sauf pour succès et erreur, qui ont un défaut documenté dans `ui-kit.md`.

---

## 3. Disposition de la page `Design`

Une page du produit = une Section. Dans chaque Section, les frames de la page dans l'ordre des formats retenus — **mobile à gauche quand on est mobile first**, desktop à droite. Les Sections sont alignées sur `y = 0`, côte à côte dans l'ordre du parcours de l'utilisateur. Une Section `Archive` ferme la rangée, tout à droite : ce qui est remplacé y va, renommé `<nom> (remplacée le AAAA-MM-JJ)`.

| Constante        | Valeur | Rôle                                            |
| ---------------- | -----: | ----------------------------------------------- |
| `PAD`            |     80 | marge entre le bord d'une Section et ses frames |
| `ECART_FRAMES`   |    240 | entre deux frames d'une même Section            |
| `ECART_SECTIONS` |    135 | entre deux Sections voisines                    |
| Mobile           |    390 | largeur du frame                                |
| Desktop          |   1440 | largeur du frame                                |

Nommage des frames : `<Page> — Mobile`, `<Page> — Desktop`.

Ranger le canevas — idempotent, à repasser après toute création ou modification de page :

```js
const page = figma.root.children.find((p) => p.name === "Design");
await figma.setCurrentPageAsync(page);
const PAD = 80,
  ECART_FRAMES = 240,
  ECART_SECTIONS = 135;
const ordre = ["Home", "Projects", "Project", "About", "Contact", "Archive"]; // parcours, Archive en dernier
const sections = page.children
  .filter((c) => c.type === "SECTION")
  .sort((a, b) => {
    const ia = ordre.indexOf(a.name),
      ib = ordre.indexOf(b.name);
    return (ia < 0 ? 98 : ia) - (ib < 0 ? 98 : ib);
  });
let curseurX = 0;
const ranges = [];
for (const S of sections) {
  const enfants = [...S.children].sort((a, b) => a.width - b.width); // mobile puis desktop
  let x = PAD,
    hMax = 0;
  for (const k of enfants) {
    k.x = x;
    k.y = PAD;
    x += k.width + ECART_FRAMES;
    hMax = Math.max(hMax, k.height);
  }
  S.x = curseurX;
  S.y = 0;
  S.resizeWithoutConstraints(Math.max(x - ECART_FRAMES + PAD, PAD * 2), hMax + PAD * 2);
  curseurX += S.width + ECART_SECTIONS;
  ranges.push(S.id);
}
return { mutatedNodeIds: ranges };
```

Puis, dans un appel séparé, `audits.md` §7. Une Section ne suit pas ses frames quand ils grandissent, et rien ne le signale.

---

## 4. Inspirations — Dribbble, et rien d'autre

Seulement si l'utilisateur **demande** des inspirations. Une seule source : Dribbble. Ni Behance, ni Pinterest, ni Awwwards, ni une galerie de mémoire.

**Ce que Dribbble laisse faire.** Ses pages — recherche comme fiches — refusent la lecture automatisée : elles reviennent vides. Ce qui fonctionne est la **recherche web restreinte au domaine** : `WebSearch` avec `allowed_domains: ["dribbble.com"]`, qui renvoie pour chaque shot son titre, son auteur et son lien. On ne voit pas l'image ; on ne prétend jamais l'avoir vue.

Méthode :

1. Trois à cinq requêtes en anglais, `<type de produit> <style ou secteur>` : `architecture studio landing page`, `minimal portfolio website`, `fintech mobile app onboarding`. Pour en trouver d'autres, l'utilisateur peut aussi partir de `https://dribbble.com/search/<mots-clés>`.
2. Retenir huit à douze shots, d'auteurs différents, en s'appuyant sur les titres et les descriptions renvoyées.
3. Dans la sous-section `Design` de `Moodboard 1`, une **carte par shot** : titre, auteur, lien cliquable, une ligne « À regarder : … » tirée du titre ou du résumé — jamais d'une image qu'on n'a pas vue — et un rectangle `Image` vide de 480 × 360 où l'utilisateur colle le visuel.
4. Le message reprend la liste des liens et demande à l'utilisateur de coller les visuels qu'il retient et de supprimer les cartes qu'il écarte. Ce tri fait partie du moodboard, donc de la porte 1.

```js
const page = figma.root.children.find((p) => p.name === "Moodboard");
await figma.setCurrentPageAsync(page);
const design = page.findOne((n) => n.type === "SECTION" && n.name === "Design");
await figma.loadFontAsync({ family: "Inter", style: "Regular" });
await figma.loadFontAsync({ family: "Inter", style: "Bold" });
const shots = [{ titre: "…", auteur: "…", url: "https://dribbble.com/shots/…", regarder: "…" }];
const ids = [];
let x = 80,
  y = 80;
for (const s of shots) {
  const carte = figma.createAutoLayout("VERTICAL", {
    name: `Inspiration · ${s.titre}`,
    itemSpacing: 12,
  });
  carte.paddingTop = carte.paddingBottom = carte.paddingLeft = carte.paddingRight = 16;
  carte.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
  design.appendChild(carte);
  carte.x = x;
  carte.y = y;
  const img = figma.createRectangle();
  img.name = "Image";
  img.resize(480, 360);
  img.fills = [{ type: "SOLID", color: { r: 0.93, g: 0.93, b: 0.93 } }];
  carte.appendChild(img);
  const titre = figma.createText();
  titre.fontName = { family: "Inter", style: "Bold" };
  titre.characters = s.titre;
  const auteur = figma.createText();
  auteur.characters = s.auteur;
  const lien = figma.createText();
  lien.characters = s.url;
  lien.hyperlink = { type: "URL", value: s.url };
  const note = figma.createText();
  note.characters = `À regarder : ${s.regarder}`;
  for (const t of [titre, auteur, lien, note]) {
    carte.appendChild(t);
    t.layoutSizingHorizontal = "FILL";
    t.textAutoResize = "HEIGHT";
  }
  ids.push(carte.id);
  x += 480 + 32 + 32;
  if (x > 2400 - 560) {
    x = 80;
    y += 360 + 140;
  }
}
return { createdNodeIds: ids };
```

Les cartes sont un échafaudage du moodboard, pas un composant : elles vivent sur `Moodboard`, jamais sur `Composant`, et ne suivent aucune variable.

---

## 5. Ce que l'API ne fait pas

À lister en cases à cocher dans le message, jamais à contourner :

- [ ] créer un dossier (projet) Figma — l'utilisateur le crée et donne le lien
- [ ] changer de plan, ajouter un mode, publier une bibliothèque
- [ ] lire une page Dribbble — l'utilisateur colle les visuels
- [ ] valider : chaque porte attend une réponse écrite de l'utilisateur
