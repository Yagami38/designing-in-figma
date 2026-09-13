# Composants — familles, états, nommage, réutilisation

Les composants vivent sur la page `Composant`, **une Section par famille**, après la Section `UI Kit`. Ils se construisent une fois l'UI kit terminé, et la porte 2 les valide **avec** l'UI kit. Les masters ne sont jamais sur `Design` : une page n'est qu'un assemblage d'instances.

Les noms d'exemple sont en anglais ; en français, même schéma traduit, une seule langue dans le fichier.

---

## 1. Ce qu'un produit possède toujours

Pour un **site** :

| Famille      | Composants obligatoires                                                                                                                            |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Navigation` | en-tête (`State = Default \| Menu open` en mobile), fil d'Ariane si le site a des niveaux                                                          |
| `Footer`     | pied de page complet : liens, mentions, newsletter si prévue                                                                                       |
| `Form`       | formulaire de contact (`State = Default \| Error \| Success`), newsletter (idem)                                                                   |
| `Card`       | une carte par type de contenu : projet, article, produit, membre…                                                                                  |
| `Section`    | **toutes les sections du produit**, une par bloc de page : hero, liste de cartes, bandeau d'appel à l'action, témoignages, FAQ, contenu riche, 404 |

Pour une **app** :

| Famille      | Composants obligatoires                                                                  |
| ------------ | ---------------------------------------------------------------------------------------- |
| `Navigation` | barre du haut, barre d'onglets ou tiroir, retour                                         |
| `List`       | ligne de liste (`State = Default \| Selected`), en-tête de groupe                        |
| `Card`       | une par type de contenu                                                                  |
| `Overlay`    | modale, feuille du bas, toast (`Tone = Neutral \| Success \| Error`)                     |
| `Screen`     | **tous les écrans du produit** ; les états vide, chargement et erreur sont des variantes |

Les atomes (`Button`, `Input`…) viennent de l'UI kit et ne se redessinent jamais dans un composant : on y pose des instances.

---

## 2. Les états sont des variantes

Une maquette ne montre par défaut que le cas nominal. Les autres cas se dessinent comme **propriétés de variante** sur le composant concerné, jamais comme frames dupliqués :

```
Section / Project grid    Breakpoint = Mobile | Desktop
                          State      = Default | Empty | Loading | Error
Form / Contact            Breakpoint = Mobile | Desktop
                          State      = Default | Error | Success
Navigation / Header       Breakpoint = Mobile | Desktop
                          State      = Default | Menu open
```

Obligatoires :

- `Empty`, `Loading`, `Error` sur toute section ou écran qui affiche des données (liste, grille, résultats, tableau de bord).
- `Error` et `Success` sur tout formulaire.
- `Hover`, `Focus`, `Disabled` sur tout contrôle — ils viennent de l'UI kit.

**Un état retire autant qu'il ajoute.** Une grille vide n'affiche pas ses filtres de tri ; un formulaire envoyé n'affiche plus son bouton. Un état qui empile un message sur l'état nominal laisse des commandes qui ne mènent nulle part.

`Breakpoint` n'existe que si deux formats ont été retenus au cadrage. Les deux variantes diffèrent par la **structure** (colonnes empilées, menu replié), pas seulement par la largeur : c'est ce qui justifie deux variantes plutôt qu'un composant redimensionnable qui masquerait des calques.

---

## 3. Nommage

```
<Famille> / <Nom>            Section / Hero · Card / Project · Form / Contact
```

- Le nom dit le **rôle**, pas l'apparence ni la page : `Card / Project`, pas `Carte bleue` ni `Carte accueil`.
- Les propriétés de variante ont des valeurs en mots entiers : `Default`, pas `def`.
- Les calques internes sont nommés par rôle (`Title`, `Excerpt`, `Cover`, `Actions`) : c'est ce que les dérogations d'instance et les scripts retrouvent.

---

## 4. Construire un composant

1. **Chercher avant de créer.** Lister les `COMPONENT_SET` existants ; si une variante ou une dérogation couvre le besoin, il n'y a pas de nouveau composant.
2. Construire la variante principale (le premier format retenu) en instances d'atomes, tout en auto-layout, toutes les valeurs liées.
3. Dériver l'autre `Breakpoint` par **clonage** de cette variante, jamais par reconstruction : cloner, restructurer à la largeur d'origine, réduire, puis repasser en `HUG` tout ce qu'un changement d'axe a laissé en `FILL` (un titre écrasé à 1 px en est le symptôme).
4. Ajouter les états en clonant la variante `Default` et en retirant ou remplaçant ce qui n'a plus d'objet.
5. Ranger le jeu : auto-layout, retour à la ligne, hug, padding `space/24`, écart `space/16`, variantes en largeur `FIXED` (`audits.md` §3).
6. Capture, et audits §1 à §5 sur la Section de la famille, dans un appel séparé.

**Corriger le master, jamais l'instance.** Une correction dans une instance crée une dérogation qui survit aux mises à jour et diverge des autres pages. Ce qui varie d'une page à l'autre (un titre, un nombre) est une dérogation voulue ; ce qui varie structurellement est un autre composant.

---

## 5. Réutiliser pendant la déclinaison

En phase 5, chaque page se compose **d'abord** avec ce qui existe, dans cet ordre :

1. une instance telle quelle ;
2. une instance avec dérogations de contenu (textes, images, icône) ;
3. une nouvelle **variante** d'un composant existant (nouvel état, nouvelle disposition mineure) ;
4. un nouveau composant — seulement si la structure diffère réellement, et il rejoint sa famille sur `Composant` avant d'être posé.

Deux composants qui ne diffèrent que par leur contenu se fusionnent : permuter les instances vers la variante équivalente (`swapComponent`), reporter les dérogations, lire ce qui a réellement été dérogé dans `instance.overrides`, et ne supprimer l'ancien qu'à **zéro instance** restante.

---

## 6. Ce qui remonte dans les composants après la porte 3

Quand l'utilisateur a retouché la première maquette directement dans Figma, ses retouches sont dans des **instances** (dérogations) ou dans des frames détachés. Avant de décliner :

1. Lister, sur la page validée, les instances portant des `overrides` et les frames qui ne sont plus des instances.
2. Pour chaque écart, décider avec l'utilisateur s'il est **local** (une dérogation de contenu) ou **systémique** (une correction du composant, une nouvelle valeur d'échelle). Rien n'est déduit en silence.
3. Reporter le systémique dans le master, puis `resetOverrides()` sur les instances concernées ; réintégrer les frames détachés comme instances.

Sans cette passe, les autres pages n'auraient pas les retouches, et la première page divergerait de son propre système.
