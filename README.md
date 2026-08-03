# MindFlow

Éditeur de cartes mentales pour usage professionnel. Application web autonome,
sans build ni dépendance d'installation : un seul fichier HTML qui embarque
sa structure, ses styles et sa logique.

## Lancer

Ouvrir `mindflow.html` dans un navigateur. C'est tout — y compris depuis
`file://`, sans serveur local.

Deux bibliothèques sont chargées depuis un CDN pour l'export PDF (jsPDF) et
la lecture des archives `.gmind` (JSZip). Elles se dégradent proprement : hors
ligne, ces deux fonctions se désactivent et le reste de l'application
fonctionne normalement.

## Ce que fait l'application

**Édition** — nœuds à formes multiples, redimensionnement par poignées,
disposition radiale ou arborescente, solveur de chevauchement, guides
d'accrochage, sélection multiple avec alignement et répartition, historique
à 120 pas.

**Organisation** — projets multiples, tags colorés avec filtrage, liens
transversaux libres entre nœuds, recherche avec navigation dans les
résultats, images et PDF attachés.

**Thème** — jour / nuit, bascule par `Ctrl+Maj+L`. Le thème nuit est en noir
absolu : sur écran OLED le pixel est éteint, ce qui donne un contraste réel
et supprime le voile gris.

**Sortie** — SVG et PDF vectoriels (texte sélectionnable), PNG haute
résolution, impression au format A4 à A0 avec cartouche, et quatre formats
d'échange : FreeMind `.mm`, OPML, Markdown, JSON natif.

**Entrée** — `.gmind` (GitMind), `.json` (GitMind ou MindFlow), `.mm`
(FreeMind), avec détection d'encodage UTF-8 puis Windows-1252.

## Persistance

Les cartes sont stockées sur l'appareil, en cascade sur quatre couches :
`window.storage`, IndexedDB, `localStorage`, puis mémoire. L'écriture va sur
toutes les couches en parallèle, la lecture prend la première qui répond.

Rien ne part sur un serveur. En conséquence, **les cartes ne suivent pas
d'un appareil à l'autre** : chaque navigateur a sa propre base. Pour
transporter une carte, passer par l'export JSON.

## Publier sur GitHub Pages

Une fois le dépôt sur GitHub : *Settings → Pages → Source : Deploy from a
branch*, brancher sur `main` et le dossier racine.

L'application devient accessible à une URL du type
`https://<compte>.github.io/mindflow/`. Le fichier `index.html` redirige vers
`mindflow.html`. C'est ce qui permet de s'en servir depuis une tablette ou un
poste sur lequel rien n'est installé.

Le dépôt doit être public pour Pages sur un compte gratuit.

## Travailler sur le fichier

Voir `CLAUDE.md` pour les conventions internes — système de jetons de couleur,
moteur vectoriel partagé, points à vérifier après modification.

Cycle habituel :

```bash
git pull                  # récupérer l'état du poste précédent
# … modifications …
git add -A
git commit -m "Description de la passe"
git push
```

Sur un fichier de cette taille, `git diff` est le meilleur outil de
relecture : il isole la modification réelle au milieu des 4 800 lignes.
