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

## Sécurité

### Assainissement des données externes

Tags, couleurs, polices et images peuvent venir d'un fichier importé ou d'un
JSON collé. Trois filtres valident leur **forme**, ils ne se contentent pas
d'échapper : `safeColor()`, `safeFont()`, `safeImg()`.

Toute valeur issue des données qui finit dans un attribut HTML ou SVG doit
passer par l'un d'eux. `esc()` sert au texte libre — il échappe aussi les
guillemets, il est donc utilisable en contexte d'attribut.

`safeImg()` refuse les data-URL SVG : un SVG est un document, donc un porteur
de script. Ne pas les réautoriser « pour la qualité vectorielle ».

Ne jamais décoder d'entités HTML par affectation `innerHTML`, même sur un
élément détaché. Utiliser `DOMParser`, qui construit un document inerte.

### Politique de sécurité de contenu

La CSP est déclarée en `<meta>` dans le `<head>`, **avant tout chargement de
ressource**. GitHub Pages ne permettant pas de définir d'en-têtes HTTP, c'est
la seule voie possible — avec deux limites : `frame-ancestors` et `report-uri`
sont inopérants en `<meta>`.

Contrat en vigueur :

- `connect-src 'self'` — aucun appel réseau sortant. C'est la directive la
  plus importante : elle ferme le canal d'exfiltration. Toute fonction
  nécessitant un appel distant doit l'élargir explicitement et sciemment.
- `script-src` — seul `cdnjs.cloudflare.com` est autorisé, pour jsPDF et
  JSZip. Ne pas ajouter de CDN sans nécessité.
- `object-src 'none'`, `base-uri 'none'`, `form-action 'none'` — sans coût,
  ferment le détournement d'URL relatives et l'exfiltration par formulaire.

Ajouter une ressource externe impose de mettre la CSP à jour, sinon elle est
bloquée silencieusement. Vérifier la console au moindre doute.

**Pourquoi `'unsafe-inline'` subsiste :** l'interface compte 114 gestionnaires
d'événements en attribut (`onclick="…"`). Ni les empreintes SHA-256 ni les
nonces ne les couvrent — seule leur suppression le permettrait, en les
convertissant tous en `addEventListener`. Les nonces sont de toute façon
inapplicables ici : ils exigent une génération côté serveur, que l'hébergement
statique ne fournit pas. Les empreintes fonctionneraient pour les deux blocs
`<script>` et le bloc `<style>`, mais devraient être recalculées à **chaque**
modification du fichier — friction réelle sur un fichier édité en continu.

Ne pas entreprendre ce chantier sans demande explicite. Tant qu'il n'est pas
fait, `'unsafe-inline'` n'annule pas la CSP : un script injecté s'exécuterait,
mais sans pouvoir sortir la moindre donnée.

### Avant d'ajouter l'authentification

Le stockage du jeton de session est le point suivant : `localStorage` expose
le compte entier à une XSS résiduelle, un cookie `httpOnly` ne l'expose pas.

## Vérifications après modification

Le fichier n'a pas de tests automatisés. Après une passe, contrôler :

1. La console est vide au chargement. **Toute erreur mentionnant
   « Content Security Policy » signale une ressource bloquée** : corriger la
   CSP ou renoncer à la ressource, jamais élargir par réflexe.
2. Impression : aperçu en A3 et en A1, fond blanc puis fond du thème. Le
   document d'impression est écrit dans une iframe qui hérite de la CSP —
   c'est le point le plus susceptible de casser après un durcissement.
3. Export : télécharger un SVG et un PNG. Les téléchargements passent par des
   URL `blob:` et `data:`, sensibles à la CSP.
4. Bascule jour / nuit : parcourir la barre d'outils, la barre latérale, le
   menu contextuel, les modales d'import et d'impression, le gestionnaire de
   tags. Chercher le texte devenu illisible.
5. Sélection multiple de trois nœuds : les trois sont cerclés, l'ancre est
   plus marquée, et **toutes** leurs liaisons ressortent.
6. Ouvrir le SVG produit dans un navigateur, comparer à l'écran. Vérifier que
   le texte est bien du texte, pas des tracés.
7. Rechargement de la page : les cartes sont retrouvées.

## Langue

Interface, commentaires et messages en français. Les identifiants de code
sont en anglais, comme dans l'existant.
