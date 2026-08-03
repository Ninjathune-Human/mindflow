# Conventions — MindFlow

## Nature du projet

Application mono-fichier. `mindflow.html` contient tout : structure, styles,
logique. Pas de build, pas de bundler, pas de `node_modules`. JavaScript
natif, aucun framework.

Ne pas proposer de découper le fichier en modules, de migrer vers React ou
d'ajouter une étape de compilation sans que ce soit demandé explicitement.
L'autonomie du fichier est une contrainte de conception : il doit rester
ouvrable en `file://` et transportable par simple copie.

## Couleurs — jetons obligatoires

Toutes les couleurs de l'interface passent par des variables CSS définies
dans les deux blocs `:root` en tête de `<style>` : le thème nuit, puis
`:root[data-theme="light"]`.

**Ne jamais écrire une couleur en dur dans une règle de l'interface.** Si un
jeton manque, en créer un dans les deux blocs plutôt que de coder la valeur.

Deux exceptions légitimes : les nuanciers proposés à l'utilisateur pour
colorer ses nœuds, et les couleurs propres aux nœuds (`nd.s.bg`,
`nd.s.color`) qui sont des données, pas de l'interface.

Piège récurrent : un élément posé **au-dessus d'un nœud** ne peut pas
s'appuyer sur `--acc`, puisque `--acc` est aussi la couleur de fond par
défaut des nœuds. C'est ce qui rendait invisible l'anneau de sélection
multiple. Pour tout ce qui se superpose à un nœud, prévoir deux couches
contrastées.

Vérifier les contrastes dans **les deux thèmes**. Une teinte pâle sur fond
sombre devient illisible sur fond blanc, et réciproquement.

## Moteur vectoriel

`buildSVG()`, `exportPDF()` et `doPrint()` partagent une seule source de
géométrie : `mapBounds()`, `ndSize()`, `edgeGeom()`, `edgeList()`,
`crossList()`, `wrapText()`, `ndBoxes()`.

Toute modification du rendu à l'écran qui touche la géométrie doit être
répercutée là. À défaut, l'export cesse de correspondre à l'affichage — c'est
la principale source de régression silencieuse sur ce fichier.

`capCanvas()` (rendu bitmap pour le PNG) utilise les mêmes helpers et lit le
fond dans `--paper`.

## Sélection

`SELS` est la sélection, un `Set`. `SEL` n'est que le nœud ancre, celui qui
pilote la barre de format. Toute logique portant sur « ce qui est
sélectionné » doit interroger `SELS`, jamais `SEL` seul.

Cette confusion a produit deux bugs successifs : les nœuds non-ancres non
mis en surbrillance, puis leurs connecteurs estompés au lieu d'être mis en
avant.

## Persistance

`DB.g()` / `DB.s()` écrivent sur les quatre couches en parallèle et lisent la
première disponible. Ne pas appeler `localStorage` ou IndexedDB directement.

## Vérifications après modification

Le fichier n'a pas de tests automatisés. Après une passe, contrôler :

1. La console est vide au chargement.
2. Bascule jour / nuit : parcourir la barre d'outils, la barre latérale, le
   menu contextuel, les modales d'import et d'impression, le gestionnaire de
   tags. Chercher le texte devenu illisible.
3. Sélection multiple de trois nœuds : les trois sont cerclés, l'ancre est
   plus marquée, et **toutes** leurs liaisons ressortent.
4. Export SVG : ouvrir le fichier produit dans un navigateur, comparer à
   l'écran. Vérifier que le texte est bien du texte, pas des tracés.
5. Impression : aperçu en A3 et en A1, fond blanc puis fond du thème.
6. Rechargement de la page : les cartes sont retrouvées.

## Langue

Interface, commentaires et messages en français. Les identifiants de code
sont en anglais, comme dans l'existant.
