---
name: designing-in-figma
description: Use when designing a site or an app in Figma through the Figma MCP (create_new_file, use_figma, get_screenshot) — a new or existing file, a moodboard, a UI kit, components, a first mockup, declining pages — or when someone asks for « the Figma design of… », « the mockups of… », wants icons drawn, extra colors, a shadow or effect, all pages at once, or says they will review at the end.
---

# Concevoir un site ou une app dans Figma

## Vue d'ensemble

**Le moodboard décide, les variables portent, les composants répètent, une page se valide avant les autres.** Rien ne se déduit en silence : une couleur, une police, un effet, une disposition sont des questions posées à l'utilisateur, jamais des choix faits à sa place. Le délai réduit le périmètre — moins de pages, moins d'états — jamais une phase ni une porte.

Les skills officiels de Figma décrivent l'API, ce skill décrit la méthode. À charger avant tout appel : `figma-use` avant chaque `use_figma`, `figma-create-new-file` avant `create_new_file`, `figma-generate-library` dès qu'un composant se crée. Charger les schémas des outils Figma en un seul `ToolSearch` avec `select:`.

Ce skill se copie tel quel dans `~/.claude/skills/` ou dans le `.claude/skills/` d'un projet. Si le projet a son propre document de standards Figma, celui-ci l'emporte là où il contredit ce skill.

## Le plan gratuit, posé en tête

- **Trois pages par fichier, et pas une de plus : `Design`, `Composant`, `Moodboard`.** Tout le reste est une Section. Ce qui est remplacé va dans une Section `Archive` en bout de `Design`, jamais à la corbeille.
- **Un seul mode par collection de variables.** Pas de Light/Dark ni de Mobile/Desktop par modes : les écarts entre formats se règlent variante par variante.
- **Pas de bibliothèque d'équipe.** Tout vit dans le fichier.
- L'API ne crée pas de dossier Figma : un dossier se crée à la main, et l'utilisateur en donne le lien.

## Les trois portes

| Porte | Après                          | Ce qu'on montre                                                                        |
| ----- | ------------------------------ | -------------------------------------------------------------------------------------- |
| 1     | le moodboard analysé           | la lecture du moodboard et la `Synthèse` : chaque choix validé question par question   |
| 2     | l'UI kit et les composants     | capture de la page `Composant`, audits vides                                           |
| 3     | la première maquette retouchée | la page validée par l'utilisateur dans Figma, ses retouches remontées dans les masters |

Une porte est un arrêt : on montre, on pose la question, on termine son tour, on attend. « Je regarderai après » dit **quand** l'utilisateur valide, pas **si** : on montre la première page et on attend quand même.

## Phase 0 — Cadrage, avant tout appel

Poser ces questions par lots de quatre au plus, et n'ouvrir aucun outil avant les réponses :

1. **Un fichier Figma existe-t-il ?** Oui → son lien. Non → où le créer ? Recommander un **dossier Figma** créé par l'utilisateur, dont il donne le lien ; sinon, à la racine (brouillons).
2. **Site ou app**, et la liste des pages ou écrans.
3. **Formats** : recommander **mobile first, puis desktop** ; l'utilisateur tranche.
4. **Librairie d'icônes** : recommander Material Design Icons. Aucune icône ne sera dessinée.
5. **Langue des noms** (celle de l'utilisateur) et **langue du contenu** (celle du produit).
6. **Stack et assets** : une stack à polices recommandées (Shopify…) ? Un logo, des photos ?

## Phase 1 — Fichier, moodboard, synthèse

1. Créer ou renommer les trois pages (`references/setup.md` §1). Sur `Moodboard`, une Section `Moodboard 1` avec quatre sous-sections **vides** : `Design`, `Couleur`, `Typo`, `À éviter`. Arrêt : l'utilisateur remplit. S'il demande des inspirations, les chercher **uniquement sur Dribbble** (`references/setup.md` §4).
2. **Analyser** : capturer chaque sous-section et écrire ce qu'on y voit — couleurs candidates avec leur valeur et l'image d'origine, familles et graisses, grille, densité, rayons, traitement des images, et **tout effet particulier** : ombre, dégradé, flou, bordure, texture, illustration, indice de mouvement.
3. **Questionner** : un choix par question, avec les options vues dans le moodboard ; chaque effet repéré fait l'objet d'une question explicite — le veut-on, où. Par lots de quatre au plus, jusqu'à épuisement.
4. Écrire les décisions dans une sous-section `Synthèse` de `Moodboard 1`. **Porte 1.**

## Phase 2 — UI kit

Sur `Composant`, Section `UI Kit` (`references/ui-kit.md`). Quatre collections en une couche : `color` minimaliste — une primary, jusqu'à trois secondaires, les neutres de blanc à noir, succès et erreur — `space` sur l'échelle 4 … 128, `radius`, `type`. Sept styles de texte (H1 à H4, Body, Small, Caption), en deux groupes si deux formats. Polices **Google Fonts** par défaut, sauf stack à polices recommandées. Un jeu `Icon` importé de la librairie choisie. Les atomes avec tous leurs états. Chaque jeu de variantes en **auto-layout, hug sur les deux axes**, sinon les variantes sont coupées.

## Phase 3 — Composants

Sur `Composant`, une Section par famille (`references/components.md`) : navigation, footer, formulaires, cartes, puis **toutes les sections du produit**. Les états — vide, chargement, erreur, succès — sont des variantes ; `Breakpoint` n'existe que si deux formats ont été retenus. Audits, capture. **Porte 2.**

## Phase 4 — Première maquette

Une seule page, choisie avec l'utilisateur, dans le premier format, assemblée **uniquement d'instances**, dans une Section de `Design` (`references/setup.md` §3). Capture, audits. **Porte 3** : l'utilisateur retouche directement dans Figma, puis valide. Relire alors la page validée et **remonter ses retouches dans les masters** (`references/components.md` §6) — sinon les autres pages ne les auront pas.

## Phase 5 — Déclinaison

Les autres pages, dans l'ordre : instance telle quelle, instance dérogée, nouvelle variante, et seulement ensuite nouveau composant. Puis le second format, dérivé du premier par clonage. Avant de rendre la main : audits sur chaque page, Sections rangées, captures regardées.

## Règles non négociables

1. **Tout en variables** : couleur, espacement, padding, rayon, taille et interligne, famille et graisse. Zéro valeur nue.
2. **L'échelle** : `4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128` — on choisit dedans, on n'en sort pas.
3. **Couleurs minimalistes** : une primary, une à trois secondaires au plus, les neutres, succès et erreur. Rien d'autre sans ligne dans la `Synthèse`.
4. **Icônes importées** d'une seule librairie, jamais dessinées.
5. **Jeux de variantes en auto-layout et hug**, variantes en largeur fixe.
6. **Tout bloc présent deux fois est un composant**, master sur `Composant`, page = instances.
7. **Une page validée avant les autres.**
8. **Consistance** : un seul style par rôle — un bouton, un titre de section, une carte ont une seule recette.
9. **Jamais supprimer** : Section `Archive`, renommée avec la date.
10. **Rien de déduit en silence** : police, couleur, effet, disposition — une question, une réponse, une ligne dans la `Synthèse`.

## Rationalisations à reconnaître

| Ce qu'on se dit                                                         | Ce qui est vrai                                                                                                                                   |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| « Elle a dit qu'elle regarderait à la fin, j'enchaîne sans validation » | « Après » dit quand elle valide, pas si. On montre la première page et on attend.                                                                 |
| « Pas de fausse pause qui contredirait sa demande »                     | Les portes ne sont pas des pauses : c'est là que le travail devient le sien. Sans elles, cinq pages à refaire au lieu d'une.                      |
| « Des icônes maison, ça fera plus perso, c'est la bonne demande »       | Un glyphe dessiné est hors grille, introuvable dans le code, refait à chaque projet. La personnalité vient du moodboard. On propose la librairie. |
| « Pas de contrainte de marque, je choisis la typo seul »                | Aucune police sans moodboard. Sans moodboard, on le crée et on attend qu'il soit rempli.                                                          |
| « Une vraie interface a besoin de nuances, deux ou trois par teinte »   | Les états se font avec `primary-hover` et les neutres. Une teinte de plus est une décision écrite dans la `Synthèse`, pas une nuance.             |
| « Une ombre par défaut soignée, réglable ensuite »                      | Un effet non validé est une décision prise à la place de l'utilisateur. On demande : le veut-on, où, comment.                                     |
| « Une page Fondations, une page Composants, cinq pages de site »        | Trois pages. Le reste, ce sont des Sections.                                                                                                      |
| « Je pars sur des placeholders pour ne pas bloquer »                    | Rien ne se construit avant la porte 1. Le temps gagné se perd à refaire.                                                                          |
| « `get_variable_defs` suffit pour vérifier »                            | Il liste les variables, pas les nœuds déliés. Les audits sont des scripts, dans un appel séparé.                                                  |

## Signaux d'alerte — s'arrêter

- « fais toutes les pages d'un coup », « je regarderai après », « dessine-moi des icônes », « ajoute un vert et un ocre », « une ombre comme sur ce site »
- une police, un hex ou un effet sans ligne dans la `Synthèse`
- un frame de page créé avant la porte 2
- une quatrième page Figma
- `create_new_file` sans avoir demandé si un fichier existe et où le créer
- `.remove()` sur quoi que ce soit

## Avant chaque porte — obligatoire

- [ ] Skills officiels chargés avant le premier appel d'écriture
- [ ] Inventaire lu avant de construire (`references/audits.md` §0) ; rien construit en double
- [ ] Audits relancés dans un appel séparé, toutes les listes vides : §1 valeurs hors variable, §3 jeux de variantes (porte 2), §2 débordement et §6 titres (portes 3 et suivantes), §7 Sections
- [ ] Captures regardées, dans chaque format retenu
- [ ] Rien supprimé : ce qui est remplacé est dans `Archive`, daté
- [ ] Message : ce qui a été fait (noms et identifiants), les questions ouvertes, ce que l'utilisateur doit valider — puis fin du tour

## Références

| Fichier                    | Quand l'ouvrir                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| `references/setup.md`      | phase 1 : pages, moodboard, inspirations Dribbble, disposition de `Design`                  |
| `references/ui-kit.md`     | phase 2 : inventaire exact des variables, styles, icônes, atomes                            |
| `references/components.md` | phases 3 à 5 : familles obligatoires, états, nommage, réutilisation, remontée des retouches |
| `references/audits.md`     | avant chaque porte, après toute suppression de variable, pour mesurer un contraste          |
