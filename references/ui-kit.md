# UI kit — variables, styles, icônes, atomes

L'UI kit vit sur la page `Composant`, dans une Section `UI Kit` composée de cinq sous-sections : `Couleurs`, `Typographie`, `Espacements et rayons`, `Icônes`, `Atomes`. Il se construit **après** la porte 1 (moodboard validé, synthèse écrite) et **avant** les composants. Aucune valeur n'est décidée ici : elles viennent toutes de la `Synthèse` du moodboard.

Les exemples sont nommés en anglais. Si l'utilisateur travaille en français, le schéma est le même, traduit (`couleur/primaire`, `Bouton`, `Champ`) — une seule langue dans tout le fichier.

---

## 1. Variables — une couche, par rôle et par échelle

Le plan gratuit n'offre **qu'un mode par collection** : pas de Light/Dark ni de Mobile/Desktop par modes. Chaque variable a une valeur, point. Les écarts entre formats se règlent variante par variante, en liant une autre valeur de la même échelle.

Quatre collections. Tout ce qui suit est **la liste complète** : on retire, on n'ajoute pas sans décision écrite dans la synthèse du moodboard.

### `color` — minimaliste

| Variable                                  | Rôle                                                      |
| ----------------------------------------- | --------------------------------------------------------- |
| `color/primary`                           | l'accent unique : boutons primaires, liens, focus         |
| `color/primary-hover`                     | le même, assombri ou éclairci de 8 à 12 %                 |
| `color/secondary-1` … `color/secondary-3` | **au plus trois**, seulement si le moodboard les impose   |
| `color/neutral-0` … `color/neutral-1000`  | dégradé de blanc à noir : 0, 100, 200 … 900, 1000         |
| `color/success`, `color/success-bg`       | confirmation ; le `-bg` est la même teinte à 12 % d'alpha |
| `color/error`, `color/error-bg`           | erreur, rupture, suppression ; idem                       |

Règles :

- **Le texte et les fonds viennent des neutres**, jamais des secondaires. Une secondaire est un accent de section ou d'illustration.
- **L'alpha vit dans la variable** (`{ r, g, b, a }`), jamais dans `paint.opacity` ni `node.opacity` : une opacité posée sur la peinture après liaison est ignorée au rendu.
- **Contraste mesuré avant d'adopter** : texte ≥ 4,5:1, grand texte et contrôles ≥ 3:1 (`audits.md` §9). Un candidat à 4,4:1 s'écarte.
- Un dégradé, une ombre, un flou n'existent que si la synthèse du moodboard les a validés ; ils deviennent alors un **style d'effet** nommé, jamais une valeur posée à la main.

### `space` — l'échelle, et rien d'autre

```
4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128
```

Multiples de 4 jusqu'à 40, de 8 jusqu'à 80, de 16 au-delà. Un besoin de 44 ou 90 px n'a pas de solution dédiée : trancher entre deux voisins. Tout `itemSpacing`, tout `padding`, tout écart entre blocs est lié à l'une de ces variables.

### `radius`

```
radius/0, radius/4, radius/8, radius/12, radius/16, radius/full (999)
```

Deux rayons par produit suffisent presque toujours : un pour les surfaces (cartes, images, panneaux), un pour les contrôles (boutons, champs). `full` est réservé aux pastilles et aux interrupteurs.

### `type` — tailles, interlignes, familles

```
type/size-12  type/size-16  type/size-20  type/size-24
type/size-32  type/size-40  type/size-48  type/size-64

type/line-16  type/line-24  type/line-28  type/line-32
type/line-40  type/line-48  type/line-56  type/line-72

font/heading  font/body            (STRING : la famille, issue du moodboard)
font/weight-heading  font/weight-body   (STRING : le style, ex. « Bold », « Regular »)
```

**Polices : Google Fonts par défaut**, vérifiées disponibles dans Figma (`listAvailableFontsAsync`) avant toute liaison. Exception : une stack qui recommande ses propres polices — Shopify et sa bibliothèque de polices, une charte d'entreprise — l'emporte.

---

## 2. Styles de texte

Sept rôles, liés aux variables (taille, interligne, famille, graisse), jamais des valeurs brutes. **Deux groupes quand deux formats sont retenus** — le plan gratuit n'a pas de modes, donc `Mobile/H1` et `Desktop/H1` sont deux styles portant le même rôle.

| Rôle      | Mobile (taille / interligne)        | Desktop | Emploi                               |
| --------- | ----------------------------------- | ------- | ------------------------------------ |
| `H1`      | 40 / 48                             | 64 / 72 | titre de page — **un seul par page** |
| `H2`      | 32 / 40                             | 48 / 56 | titre de section                     |
| `H3`      | 24 / 32                             | 32 / 40 | titre de carte, de sous-bloc         |
| `H4`      | 20 / 28                             | 24 / 32 | sous-titre, intitulé de formulaire   |
| `Body`    | 16 / 24                             | 16 / 24 | texte courant                        |
| `Small`   | 12 / 16                             | 12 / 16 | mentions, aide, métadonnées          |
| `Caption` | 12 / 16, capitales, +4 % d'approche | idem    | étiquettes, surtitres                |

Un style `Hn` **est** un `<hn>` : ce qui ressemble à un titre sans en être un — un prix, un chiffre clé, un logo — ne porte pas un style de titre. Si le produit en a besoin, un huitième style `Emphasis`, même rendu que `H4` et aucun rôle de titre, s'ajoute avec une ligne dans la `Synthèse`. Le corps de texte ne descend jamais sous 16 ; le 12 est réservé aux mentions.

---

## 3. Icônes — importées, jamais dessinées

**Question de cadrage :** quelle librairie ? Recommander **Material Design Icons** (Google). Alternatives acceptables si l'utilisateur les préfère : Lucide, Phosphor, Heroicons. Une seule librairie par produit, variante unique (`filled` ou `outlined`, pas les deux).

Un glyphe dessiné à la main est une dette : hors grille optique, introuvable dans le code, refait à chaque projet. Même pour « faire plus perso ».

Recette d'import, glyphe par glyphe :

```bash
# Material Design Icons, variante filled
curl -sfL -o <name>.svg "https://cdn.jsdelivr.net/npm/@material-design-icons/svg/filled/<name>.svg"
# Lucide
curl -sfL -o <name>.svg "https://cdn.jsdelivr.net/npm/lucide-static/icons/<name>.svg"
```

Puis `upload_assets` (`count` = nombre de fichiers, `Content-Type: image/svg+xml`) : chaque SVG arrive en arbre de vecteurs sur la page courante. Un script `use_figma` transforme chaque arbre en `COMPONENT` de 24 × 24, contraintes des vecteurs en `SCALE`, remplissage lié à `color/neutral-1000`, puis `figma.combineAsVariants` en un jeu `Icon` avec la propriété `Name = <nom exact de la librairie>`. Trois tailles à l'usage : 16 dans un bouton, 20 dans un contrôle, 24 en navigation.

**L'icône prend la couleur du texte qu'elle accompagne**, en réutilisant sa variable par dérogation d'instance.

---

## 4. Atomes — les composants que tout produit possède

Chaque atome est un jeu de variantes. **Le jeu est en auto-layout, retour à la ligne, hug sur les deux axes, avec un padding `space/24` et un écart `space/16`**, pour qu'aucune variante ne soit coupée ni étirée ; chaque variante garde une largeur `FIXED` (`audits.md` §3). Les états obligatoires sont ceux du tableau ; on n'en retire pas.

| Atome      | Propriétés                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| `Button`   | `Style = Primary \| Secondary \| Ghost` · `Size = M \| S` · `State = Default \| Hover \| Focus \| Disabled` |
| `Link`     | `State = Default \| Hover \| Focus`                                                                         |
| `Input`    | `Type = Text \| Textarea \| Select` · `State = Default \| Focus \| Filled \| Error \| Success \| Disabled`  |
| `Checkbox` | `Checked = Off \| On` · `State = Default \| Focus \| Disabled`                                              |
| `Radio`    | idem                                                                                                        |
| `Toggle`   | idem                                                                                                        |
| `Tag`      | `Tone = Neutral \| Primary \| Success \| Error`                                                             |
| `Message`  | `Tone = Success \| Error` — le texte d'aide sous un champ, la bannière d'un formulaire                      |

Recettes qui évitent de tout redessiner :

- `Button` : padding `space/12` × `space/24` en M, `space/8` × `space/16` en S ; écart interne `space/8` ; rayon des contrôles ; libellé en `Body` (M) ou `Small` (S). Un emplacement `Icon` **visible dans le master**, masqué par dérogation sur les instances qui n'en veulent pas — un enfant masqué dans le master n'existe pas pour ses instances.
- `Hover` : fond `color/primary-hover` pour Primary ; `color/neutral-100` pour Secondary et Ghost. `Focus` : anneau de 2 px en `color/primary` à l'extérieur, jamais une suppression du contour. `Disabled` : `color/neutral-300` sur `color/neutral-100`, curseur sans effet.
- `Input` : fond `color/neutral-0`, bordure 1 px `color/neutral-300`, `Focus` en `color/primary`, `Error` en `color/error` avec `Message`, `Success` en `color/success`. La valeur remplit la largeur : une instance s'étire dans son parent sans réglage. `Textarea` = même coquille, texte ancré en haut, hauteur `space/128`.

---

## 5. Contrôle de sortie de l'UI kit

Avant de passer aux composants — dans un appel séparé de toute écriture :

- [ ] `audits.md` §1 sur la Section `UI Kit` : aucune valeur hors variable, aucun texte sans style
- [ ] `audits.md` §3 : aucun jeu de variantes coupé ou étiré
- [ ] `audits.md` §9 : contraste de `color/neutral-1000` sur `neutral-0`, de `neutral-0` sur `primary`, de `success` et `error` sur `neutral-0`
- [ ] Capture de la Section `UI Kit` regardée : toutes les variantes visibles, lisibles, alignées
