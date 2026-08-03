# MindFlow

Éditeur de cartes mentales pour usage professionnel. Application web autonome,
sans build ni dépendance d'installation : un seul fichier HTML qui embarque
sa structure, ses styles et sa logique.

**En ligne : https://ninjathune-human.github.io/mindflow/**

## Lancer

Trois façons, au choix :

- **Depuis le web** — ouvrir l'adresse ci-dessus. Rien à installer, y compris
  sur tablette.
- **En local** — ouvrir `mindflow.html` dans un navigateur. Fonctionne
  directement en `file://`, sans serveur.
- **Hors ligne** — copier le seul fichier `mindflow.html` sur une clé ou un
  partage réseau. Il est autonome.

Deux bibliothèques sont chargées depuis un CDN : jsPDF pour l'export PDF,
JSZip pour la lecture des archives `.gmind`. Elles se dégradent proprement —
sans réseau, ces deux fonctions se désactivent et le reste de l'application
continue de tourner.

## Ce que fait l'application

**Édition** — nœuds à formes multiples (arrondi, rectangle, pilule, cercle,
hexagone), redimensionnement par poignées, disposition radiale ou
arborescente, solveur de chevauchement, guides d'accrochage, sélection
multiple avec alignement et répartition, historique à 120 pas.

**Organisation** — projets multiples, tags colorés avec filtrage, liens
transversaux libres entre nœuds, recherche avec navigation dans les
résultats, images et PDF attachés.

**Thème** — jour / nuit, bascule par le bouton ☾ / ☀ ou `Ctrl+Maj+L`. Le
thème nuit est en noir absolu : sur écran OLED le pixel est éteint, ce qui
donne un contraste réel et supprime le voile gris. Au premier lancement, le
thème suit le réglage du système ; ensuite il suit votre choix.

**Sortie** — SVG et PDF vectoriels avec texte sélectionnable, PNG haute
résolution, impression du A4 au A0 avec cartouche, et quatre formats
d'échange : FreeMind `.mm`, OPML, Markdown, JSON natif.

**Entrée** — `.gmind` (GitMind), `.json` (GitMind ou MindFlow), `.mm`
(FreeMind), avec détection d'encodage UTF-8 puis repli Windows-1252 pour les
fichiers produits sous Windows en français.

## Raccourcis

| Touche | Action |
| --- | --- |
| `Tab` | Nouveau nœud enfant |
| `Entrée` | Nouveau nœud frère |
| `F2` | Éditer le nœud |
| `Suppr` | Supprimer la sélection |
| `Espace` | Recentrer sur la carte |
| `Échap` | Désélectionner, fermer les panneaux |
| `Ctrl+Z` / `Ctrl+Y` | Annuler / Rétablir |
| `Ctrl+D` | Dupliquer |
| `Ctrl+A` | Tout sélectionner |
| `Ctrl+E` | Sélectionner la branche |
| `Ctrl+F` | Rechercher |
| `Ctrl+P` | Imprimer |
| `Ctrl+Maj+L` | Basculer jour / nuit |
| `Maj` + clic | Ajouter à la sélection |
| `Maj` + glisser | Encadrer une zone |
| `Alt` + flèches | Déplacer de 10 px (`Alt+Maj` : 1 px) |

## Persistance

Les cartes sont stockées sur l'appareil, en cascade sur quatre couches :
`window.storage`, IndexedDB, `localStorage`, puis mémoire. L'écriture va sur
toutes les couches en parallèle, la lecture prend la première qui répond.

Rien ne part sur un serveur. En conséquence, **les cartes ne suivent pas d'un
appareil à l'autre**, ni même d'un navigateur à l'autre : chaque navigateur a
sa propre base. Pour transporter une carte, passer par l'export JSON.

Corollaire à connaître : vider les données de navigation du site efface les
cartes. Exporter avant toute opération de nettoyage.

## Hébergement

Le site est publié par GitHub Pages depuis la branche `main`, dossier racine.
Le fichier `index.html` ne fait que rediriger vers `mindflow.html`.

L'adresse Pages reprend le nom du dépôt **en respectant la casse**, et GitHub
ne redirige pas les anciennes adresses Pages après un renommage. Renommer le
dépôt casse donc les liens existants.

## Modifier le fichier

### Depuis l'interface web de GitHub

Ouvrir le fichier dans le dépôt, cliquer sur l'icône crayon, éditer, valider
le commit. Suffisant pour une retouche ponctuelle.

Pour créer un fichier dont le nom commence par un point — `.gitignore` par
exemple — passer par *Add file → Create new file* et taper le nom dans le
champ. Le glisser-déposer ne convient pas : le système d'exploitation masque
ces fichiers et les rend indisponibles à la sélection.

### Depuis un poste, avec Git

```bash
git clone https://github.com/Ninjathune-Human/mindflow.git
cd mindflow
# … modifications …
git add -A
git commit -m "Description de la passe"
git push
```

Sur les autres postes, `git pull` avant de commencer. Sur un fichier de cette
taille, `git diff` est le meilleur outil de relecture : il isole la
modification réelle au milieu des 4 800 lignes.

### Conventions

`CLAUDE.md` décrit les règles internes à respecter — système de jetons de
couleur, moteur vectoriel partagé entre l'écran et les exports, distinction
entre la sélection et le nœud ancre, et la liste des points à vérifier après
chaque modification. Le fichier est lu automatiquement par Claude Code au
début de chaque session.

## Structure du dépôt

| Fichier | Rôle |
| --- | --- |
| `mindflow.html` | L'application entière |
| `index.html` | Redirection pour GitHub Pages |
| `CLAUDE.md` | Conventions de développement |
| `README.md` | Ce document |
| `.gitignore` | Exclut les exports produits par l'application |
