# designing-in-figma

Un skill pour Claude Code qui encode une méthode de conception de sites et d'apps dans Figma, par le serveur MCP Figma : cadrage, moodboard analysé et validé choix par choix, UI kit minimaliste en variables, composants avec tous leurs états, première maquette validée avant de décliner les autres pages. Il est écrit pour le **plan gratuit** de Figma (trois pages par fichier, un mode par collection, pas de bibliothèque d'équipe).

Le skill est en français. Les identifiants Figma d'exemple sont en anglais ; il demande à l'utilisateur dans quelle langue nommer ses variables et composants.

## Prérequis

- [Claude Code](https://claude.com/claude-code) avec le connecteur Figma activé (serveur MCP Figma de claude.ai).
- Un compte Figma, plan gratuit ou payant.

## Installation

Pour tous les projets :

```bash
git clone https://github.com/Yagami38/designing-in-figma.git ~/.claude/skills/designing-in-figma
```

Pour un seul projet, depuis sa racine :

```bash
git clone https://github.com/Yagami38/designing-in-figma.git .claude/skills/designing-in-figma
```

Mise à jour : `git pull` dans le dossier. Claude Code charge le skill dès qu'une conversation touche à la conception dans Figma ; on peut aussi l'invoquer avec `/designing-in-figma`.

## Ce que contient le dépôt

| Fichier                    | Contenu                                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `SKILL.md`                 | la méthode : contraintes du plan gratuit, les trois portes de validation, le cadrage, les cinq phases, les règles non négociables, les rationalisations à reconnaître, la liste obligatoire avant chaque porte |
| `references/setup.md`     | fichier existant ou nouveau (dossier Figma recommandé), les trois pages, la Section `Moodboard 1` et sa `Synthèse`, la disposition de `Design`, les inspirations Dribbble |
| `references/ui-kit.md`    | l'inventaire exact des variables (couleurs minimalistes, échelle d'espacement 4 → 128, rayons, typographie), les sept styles de texte, l'import d'icônes, les atomes et leurs états |
| `references/components.md` | les familles de composants obligatoires pour un site et pour une app, les états en variantes, le nommage, l'ordre de réutilisation, la remontée des retouches dans les masters |
| `references/audits.md`    | dix scripts de contrôle en lecture seule, à passer avant chaque porte                                        |

## La méthode en bref

1. **Cadrage** — avant tout appel : fichier existant ou dossier à créer, site ou app, formats (mobile first recommandé), librairie d'icônes (Material Design recommandée, jamais d'icône dessinée), langues, stack et assets.
2. **Moodboard** — trois pages `Design`, `Composant`, `Moodboard` ; une Section `Moodboard 1` vide (Design, Couleur, Typo, À éviter) que l'utilisateur remplit ; Claude l'analyse et valide chaque choix par une question, effets compris ; les décisions sont écrites dans une `Synthèse`. **Porte 1.**
3. **UI kit** — variables en une couche : une primary, jusqu'à trois secondaires, les neutres, succès et erreur ; espacements sur l'échelle `4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 48, 56, 64, 72, 80, 96, 112, 128` ; styles de texte liés aux variables ; icônes importées ; atomes avec Hover, Focus, Disabled, Erreur, Succès.
4. **Composants** — navigation, footer, formulaires, cartes, puis toutes les sections du produit, avec Vide, Chargement et Erreur en variantes. **Porte 2.**
5. **Première maquette** — une seule page, assemblée d'instances ; l'utilisateur la retouche directement dans Figma et la valide ; ses retouches remontent dans les masters. **Porte 3.**
6. **Déclinaison** — les autres pages avec le maximum de composants existants, puis le second format.

Toutes les valeurs — couleur, espacement, padding, rayon, taille et interligne, famille et graisse — sont des variables. Rien n'est déduit en silence : une police, une couleur, un effet sont des questions posées à l'utilisateur.

## Limites connues

- L'API Figma ne crée pas de dossier : l'utilisateur le crée et donne le lien.
- Dribbble refuse la lecture automatisée de ses pages ; le skill s'appuie sur la recherche web restreinte au domaine et crée des cartes d'inspiration (titre, auteur, lien, emplacement d'image) que l'utilisateur complète.

## Contribuer

Un piège rencontré en situation réelle s'ajoute dans la référence concernée sous la forme : symptôme observé, cause, parade, date. Les issues et pull requests sont les bienvenues.

## Licence

MIT — voir `LICENSE`.

---

## English summary

A Claude Code skill that encodes a design method for websites and apps in Figma through the Figma MCP server: scoping questions, a moodboard analyzed and validated choice by choice, a minimalist variable-based UI kit, components with all their states, and a first mockup validated before the other pages are derived. Written for Figma's free plan (three pages per file, one mode per collection, no team library). The skill itself is in French; install it with `git clone https://github.com/Yagami38/designing-in-figma.git ~/.claude/skills/designing-in-figma`.
