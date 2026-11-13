# Format `trustdiagram` v1 — schémas d'architecture as code

English version: [docs/format.md](format.md)

Un bloc ` ```trustdiagram ` décrit **une vue d'architecture** dans une note Obsidian ou une page Confluence.
Il est écrit en YAML et contient deux parties qui ne se mélangent jamais :

| Partie | Rôle | Qui l'écrit |
|---|---|---|
| `model` (avec `title`, `description`) | **Le sens** : éléments, imbrication, flux, protocoles, liens vers les notes. Suffit à un humain ou à un LLM pour comprendre l'architecture. | L'architecte, ou un LLM |
| `layout` | **Le dessin** : positions, tailles, formes, styles. N'apporte aucune information d'architecture. | Surtout l'éditeur graphique |

> **Règle d'or.** Déplacer, redimensionner ou restyler une boîte ne modifie **que** `layout`.
> `model`, les commentaires et la mise en forme du reste du bloc restent identiques octet pour octet.
> Seules la création ou la suppression explicite d'un élément dans l'éditeur touchent `model`.

Validation formelle : [`packages/core/schema/trustdiagram.schema.json`](../packages/core/schema/trustdiagram.schema.json) (JSON Schema 2020-12).

## Pour un LLM qui lit un bloc

- Ignorez `layout` pour comprendre l'architecture : tout le sens est dans `model`, `title` et `description`.
- L'imbrication (`children`) signifie **appartenance ou hébergement** : un élément est dans une zone (réseau, SI, compte cloud), un groupe (niveau, domaine) ou un système.
- Une relation `from → to` est orientée dans le **sens du flux principal** (qui appelle, qui envoie). `label` est le verbe, `protocol` le moyen technique, `mode` la nature de l'échange.
- `ref: "[[Note]]"` pointe vers une note du vault qui détaille l'élément. Un statut (existant, nouveau, cible…) s'écrit en propriété (`properties: { status: new }`) ; une règle d'affichage en tire la couleur (voir « Règles d'affichage »).

## Structure

```yaml
trustdiagram: 1                       # version du format (obligatoire)
id: plateforme-llm-conteneurs       # identifiant stable du schéma (obligatoire), sert à la synchro
title: Plateforme LLM — vue conteneurs
kind: flow                     # flow (v1) ; sequence arrive en phase 6
description: Phrase ou paragraphe qui dit ce que montre la vue.
model:
  elements: [...]              # obligatoire
  relations: [...]
layout: {...}                  # optionnel
```

### Identifiants

Tous les `id` (schéma, éléments, relations) respectent `^[a-z0-9][a-z0-9-_]*$`.
Les identifiants des éléments et des relations sont **uniques dans tout le bloc** (un élément et une relation ne peuvent pas partager un id).

### Élément (`model.elements[]`, récursif via `children`)

| Champ | Obligatoire | Valeurs | Sens |
|---|---|---|---|
| `id` | oui | identifiant | Référencé par les relations et par `layout` |
| `type` | oui | `person`, `system`, `container`, `component`, `datastore`, `queue`, `node`, `zone`, `group`, `external` | Nature de l'élément (voir ci-dessous) |
| `name` | oui | texte non vide | Libellé affiché |
| `tech` | non | texte | Technologie, produit, protocole d'exposition |
| `description` | non | texte | Responsabilité de l'élément |
| `ref` | non | `"[[Note]]"` | Note du vault qui le détaille |
| `children` | non | liste d'éléments | Éléments contenus |

| `type` | Usage |
|---|---|
| `person` | Utilisateur, rôle, acteur humain |
| `system` | Système ou application vu comme un tout |
| `container` | Unité déployable d'un système (application, service, conteneur) |
| `component` | Composant interne à un conteneur |
| `datastore` | Base, index, stockage, datalake |
| `queue` | File, topic, bus de messages |
| `node` | Élément d'infrastructure (VM, cluster, équipement, boîtier) |
| `zone` | Périmètre d'hébergement ou de sécurité (SI, compte cloud, zone réseau) — toujours conteneur |
| `group` | Regroupement logique (niveau, domaine) — toujours conteneur |
| `external` | Service tiers, hors périmètre |

Un élément est **conteneur** s'il a des `children` ou s'il est de type `zone` ou `group`.

### Relation (`model.relations[]`)

| Champ | Obligatoire | Valeurs | Sens |
|---|---|---|---|
| `id` | oui | identifiant | Référencé par `layout.edges` |
| `from`, `to` | oui | id d'élément | Sens du flux principal ; un conteneur est une extrémité valide |
| `label` | non | texte | Verbe ou nature du flux |
| `protocol` | non | texte | `HTTPS`, `MCP`, `SQL`, `4G`… |
| `mode` | non | `sync`, `async`, `event` | Appel synchrone, échange asynchrone (traces, envois différés), événement. **Absent = non précisé** : renseignez-le dès que vous le connaissez, un LLM ne le devine pas |
| `data` | non | liste de textes | Données échangées. Recommandé sur les flux principaux : c'est ce qui permet de répondre à « que transporte ce flux ? » |
| `description` | non | texte | Précisions (authentification, contraintes, sécurité) |

### Mise en page (`layout`)

```yaml
layout:
  theme: encre                                   # thème du schéma : ardoise (défaut, non écrit), encre ou brique
  colors: ["#C03A64", "#5A7A4A"]                # couleurs ajoutées au nuancier de ce schéma (écrites par l'éditeur)
  font: serif                                    # police de base du texte : sans (Arimo, défaut, non écrit), serif (Tinos), mono (Cousine)
  direction: right                               # sens du placement automatique : right (défaut), down, left, up
  jumps: arc                                     # sauts en arc aux croisements de flèches : arc (défaut) ou none
  layers:                                        # calques, du fond vers le dessus (« defaut » implicite, tout en bas)
    - { id: infra, name: Infrastructure }
    - { id: flux, name: Flux de données, visible: false, locked: true }
  nodes:                                         # clé = id d'élément
    gw: { x: 320, y: 140, w: 160, h: 60, shape: aws.api-gateway, style: { fill: "#FFE6CC" } }
  edges:                                         # clé = id de relation
    r1: { route: orthogonal, points: [[400, 260]], labelPos: [410, 250], style: { stroke: "#FF9900" } }
```

- **Calques** : `layers` liste les calques du fond vers le dessus (`id`, `name`, `visible`, `locked`) ; `nodes.<id>.layer` range un élément **de premier niveau** dans un calque (ses enfants suivent ; une flèche suit sa source). Un calque masqué n'est **pas dessiné** au rendu (ni les flèches qui touchent ses éléments) mais garde sa place dans le placement automatique ; un calque verrouillé n'est pas modifiable dans l'éditeur. Un calque inconnu donne un avertissement.
- **Ordre d'empilement** : `nodes.<id>.z` (entier, 0 par défaut) ; entre frères, ordre de dessin = calque, puis `z`, puis ordre du modèle.
- `nodes.<id>` : `x`, `y` obligatoires (coin haut-gauche, **coordonnées absolues** en pixels, entiers) ; `w`, `h` optionnels ; `shape` au format `famille.nom` (`k8s.pod`, `c4.person`, `network.firewall` — voir « Formes ») ; `style` = paires clé/valeur libres.
- `edges.<id>` : `route` (`orthogonal`, `straight`, `curved`), `points` (points de passage), `labelPos` (centre du libellé, coordonnées absolues ; par défaut, milieu de la longueur du tracé), `sourcePort` / `targetPort` (accroche figée sur le contour de la source / de la cible, position relative `[rx, ry]` avec des valeurs de 0 à 1 : `[1, 0.25]` = bord droit, au quart de la hauteur), `style`.
- **Style** (`style` d'un nœud ou d'une flèche), clés lisibles : `fill` (couleur ou `none`), `stroke` (couleur du contour ou du trait), `width` (épaisseur, 0,5 à 8), `dash` (`solid`, `dashed`, `dotted`), `color` (couleur du texte), pour les boîtes `align` (position du texte : `top-left`, `top`, `top-right`, `left`, `center`, `right`, `bottom-left`, `bottom`, `bottom-right` ; par défaut `center`, et `top-left` pour une zone ou un groupe ; sans effet sur les formes à icône, dont le texte reste sous l'icône) et, pour les flèches, `start` / `end` (`none`, `block`, `open`, `diamond`, `oval` ; dans l'éditeur, des boutons dessinés en mode avancé). **Texte** (boîtes et flèches) : `size` (taille du nom en px, 8 à 48 ; 13 par défaut pour une boîte, 11 pour une flèche ; la `[technologie]` et les interlignes suivent la taille), `font` (`sans`, `serif`, `mono` ; par défaut la police de base `layout.font`), `bold` (`true` / `false` ; le nom d'une boîte est en gras par défaut, une flèche non), `italic` et `underline` (`true`). Exemple : `style: { size: 16, italic: true, bold: false }`. Une valeur hors limites donne un avertissement ; le rendu la borne ou l'ignore. Les clés de style maxGraph brutes restent acceptées.
- Chaque entrée tient **sur une ligne** (style flow YAML) pour garder la section courte et les diffs lisibles.
- Une entrée dont l'id n'existe plus dans `model` est **orpheline** : elle produit un avertissement, est ignorée au rendu et conservée par la fusion.

## Propriétés, classes et règles d'affichage

Le schéma peut porter des **données** (des propriétés nom / valeur sur les objets) et des **règles** qui en tirent l'affichage, comme la mise en forme conditionnelle de Lucidchart. Les propriétés et les classes sont du **sens** (`model`), les règles de l'**affichage** (`layout`).

### Propriétés (`properties`)

Sur un élément (à toute profondeur) ou une relation : des scalaires (texte, nombre, booléen ; `null` = non renseignée). Une valeur s'affiche **telle qu'elle est écrite** (`version: 2.10` s'affiche « 2.10 ») ; une valeur non scalaire (liste, objet) est une erreur.

```yaml
    - id: gw
      type: container
      name: Passerelle {{version}}
      classes: [composant-applicatif]
      properties: { version: 2.10, exposé_internet: true }
```

### Classes (`model.classes`, `classes`)

Une classe est un modèle de propriétés avec des valeurs par défaut ; un objet porte une liste `classes`. **Valeur effective** : les classes dans l'ordre de la liste (la dernière l'emporte), puis les `properties` de l'objet. Les valeurs héritées ne sont jamais recopiées dans l'objet. Une classe inconnue donne un avertissement ; un identifiant de classe en double, une erreur.

```yaml
model:
  classes:
    - id: composant-applicatif
      name: Composant applicatif
      properties: { responsable: null, criticité: moyenne, exposé_internet: false }
```

### Références dans le texte (`{{…}}`)

Dans `name` et `tech` d'un élément et `label` d'une relation, `{{nom}}` affiche la valeur effective de la propriété. Pseudo-propriétés : `{{id}}`, `{{type}}`, `{{tech}}` (élément), `{{protocol}}`, `{{mode}}` (relation). Booléens affichés « oui » / « non » ; propriété inconnue ou vide : rien n'est affiché, avec un avertissement. Le texte brut reste lisible : `name: Passerelle {{version}}`.

### Règles (`layout.rules`)

Liste ordonnée ; pour une même clé de style, **la dernière règle vraie l'emporte**. **Une règle l'emporte sur le format posé à la main** (`layout.*.style`), lui-même au-dessus du thème et de l'état.

```yaml
layout:
  rules:
    - name: Exposé sur Internet          # obligatoire : affiché dans « Règles appliquées »
      when: { exposé_internet: true }    # scalaire = « égal à »
      then: { stroke: "#C0392B", width: 2, icon: { name: shield-alert, position: top-right } }
    - name: Version obsolète
      when: { version: { lt: 2 } }
      then: { fill: "#FDEEDB" }
    - name: Critique ou exposé
      any: true                          # au moins une condition vraie (sinon : toutes)
      when: { criticité: haute, exposé_internet: true }
      then: { color: "#A63A2C" }
    - name: Brouillon
      enabled: false                     # règle désactivée, gardée dans le bloc
      when: { statut: brouillon }
      then: { dash: dashed }
```

- **Opérateurs** : `eq`, `ne`, `contains` (texte), `gt`, `gte`, `lt`, `lte` (nombres), `in` (liste de valeurs), `set` (`true` : renseignée ; `false` : vide). Un seul opérateur par condition. Égalité de texte insensible à la casse ; `"2.4"` et `2.4` sont égaux ; `"true"` vaut `true`.
- **Avoir la propriété** : la clé existe sur l'objet ou dans l'une de ses classes, même vide. Un objet qui n'a pas la propriété ne satisfait **aucun** test, `set: false` compris (« Sans responsable » ne vise que les objets qui attendent un responsable).
- **Actions** (`then`) : `fill`, `stroke`, `width`, `dash`, `color` (couleur du texte), et `icon: { name, position, color? }` pour les boîtes, zones et groupes. Sur une relation, seules `stroke`, `width`, `dash` et `color` s'appliquent.
- **Icône** : `position` parmi les 9 positions de `align` ; plusieurs icônes à la même position se rangent côte à côte vers l'intérieur ; une boîte à **taille automatique** réserve leur place (bande libre au-dessus et au-dessous du texte pour une icône en haut ou en bas, sur les côtés pour `left` / `right`), une boîte à taille figée (`w`, `h`) garde sa taille ; `color` vaut par défaut le `stroke` de la règle, sinon l'encre du thème. Noms disponibles (Lucide, licence ISC) : `shield-alert`, `shield-check`, `lock`, `lock-open`, `globe`, `triangle-alert`, `circle-alert`, `circle-check`, `circle-x`, `clock`, `flame`, `star`, `flag`, `bug`, `database`, `cloud`, `server`, `key-round`, `user`, `users`, `eye`, `eye-off`, `zap`, `wrench`, `archive`, `ban`, `info`, `badge-check`, `activity`, `gauge`, `hourglass`, `link`, `unlink`, `wifi-off`, `cpu`, `hard-drive`, `network`, `file-warning`, `trash-2`, `sparkles`.
- **Robustesse** : une règle mal écrite ne bloque jamais le schéma. Sans nom ou sans condition, elle est ignorée ; une action mal formée (couleur qui n'est pas `#RRVVBB`, épaisseur hors 0,5–8, trait ou icône inconnus, champ inconnu) est ignorée seule. Chaque cas donne un avertissement avec son chemin (`/layout/rules/2/then/icon/name`). Les écritures sans ambiguïté sont corrigées : `width: "2"` vaut 2, `enabled: "false"` vaut false.

## Limites de taille

Le rendu tourne chez chaque lecteur (macro Confluence, Obsidian) : un bloc démesuré est refusé avec une erreur plutôt que de bloquer la page.

| Limite | Valeur |
|---|---|
| Taille du bloc | 512 Ko |
| Nom, `tech`, libellé ou protocole de relation, titre | 1 000 caractères |
| Éléments (enfants compris) | 1 500 |
| Relations | 3 000 |
| Classes | 300 |
| Calques | 100 |
| Règles | 200 |

## Placement automatique

Tout élément **sans entrée** dans `layout.nodes` est placé automatiquement au rendu, de façon déterministe (même résultat dans Obsidian, Confluence et la CLI). Il n'est écrit dans `layout` (« figé ») que lorsqu'on le retouche ou qu'on lance « Figer la mise en page ». C'est ce qui garde `layout` court.

Moteur : ELK `layered`, direction donnée par `layout.direction` (`right` par défaut ; `down`, `left`, `up` comme `LR`, `TB`, `RL`, `BT` en Mermaid), hiérarchie incluse, arêtes orthogonales, `randomSeed: 1`, espacement 40 px, marge des conteneurs : haut 40, gauche 20, bas 20, droite 20.

Règles, appliquées récursivement à chaque conteneur (et à la racine) :

1. **Taille d'une boîte libre**, mesurée avec les chasses réelles de la police embarquée **Arimo** (métriquement identique à Arial) : largeur = `max(120, ⌈max(largeur du nom en gras 13 px, largeur de « [tech] » en 11 px)⌉ + 24)` ; hauteur = `max(60, 12 + 18 + (tech ? 15 : 0) + 12)`. Un caractère absent de la table (émoji…) compte pour la chasse moyenne.
2. **Nœud figé** : sa position est respectée telle quelle ; sans `w`/`h` il prend la taille calculée. Un **conteneur figé sans taille** englobe ses enfants (minimum 120 × 60).
3. **Enfants libres d'un conteneur** :
   - s'il n'a aucun enfant figé, ils sont placés par ELK en partant de son coin intérieur `(x + 20, y + 40)` ;
   - s'il a des enfants figés, ils sont placés par ELK à droite de ceux-ci : départ en `(x max des figés + 80, y min des figés)`.
   - À la racine sans aucun nœud figé, ELK place tout le schéma.
4. **Conteneur libre contenant des éléments figés** : sa boîte est calculée pour englober ses enfants (avec les marges ci-dessus).
5. **Arêtes** : voir « Tracé des relations » ci-dessous.
6. Toutes les coordonnées produites sont des entiers.

> Conseil : pour des zones figées empilées, donnez-leur `w` et `h` ; sinon leur hauteur dépend de leur contenu et peut chevaucher la zone suivante. Pour une lecture verticale, utilisez `direction: down` (de haut en bas, comme l'exemple `photo-sharing`) ou `direction: up` (de bas en haut).

## Formes

Le champ `layout.nodes.<id>.shape` choisit la forme d'un élément. Sans `shape`, chaque type a sa forme par défaut :

| `type` | Forme par défaut |
|---|---|
| `person` | `general.person` (icône, libellé dessous — 40 px réservés pour l'icône) |
| `system`, `container`, `external` | `general.rounded` |
| `component` | `uml.component` |
| `datastore` | `general.cylinder` |
| `queue` | `general.queue` |
| `node` | `uml.node` |
| `zone` | `general.zone` (bordure pleine) |
| `group` | `general.group` (fond clair, bordure pointillée) |

Catalogue V1 :

| Famille | Formes |
|---|---|
| Générales | `general.box` Rectangle · `general.rounded` Rectangle arrondi · `general.cylinder` Base de données · `general.queue` File de messages · `general.document` Document · `general.cloud` Nuage · `general.person` Personne · `general.hexagon` Hexagone · `general.folder` Dossier · `general.ellipse` Ellipse · `general.diamond` Losange · `general.note` Note · `general.zone` Zone · `general.group` Groupe |
| C4 | `c4.person` Personne (C4) · `c4.system` Système logiciel (C4) · `c4.container` Conteneur (C4) · `c4.component` Composant (C4) · `c4.database` Base de données (C4) · `c4.external` Système externe (C4) |
| UML | `uml.component` Composant (UML) · `uml.package` Paquetage · `uml.node` Nœud (UML) · `uml.interface` Interface · `uml.actor` Acteur (UML) · `uml.artifact` Artefact |
| Réseau | `network.server` Serveur · `network.firewall` Pare-feu · `network.load-balancer` Répartiteur de charge · `network.router` Routeur · `network.switch` Commutateur · `network.desktop` Poste de travail · `network.laptop` Ordinateur portable · `network.mobile` Mobile · `network.internet` Internet · `network.storage` Stockage |
| Kubernetes (icônes officielles) | `k8s.api` API server · `k8s.c-c-m` Cloud controller manager · `k8s.c-m` Controller manager · `k8s.c-role` ClusterRole · `k8s.cm` ConfigMap · `k8s.control-plane` Control plane · `k8s.crb` ClusterRoleBinding · `k8s.crd` CustomResourceDefinition · `k8s.cronjob` CronJob · `k8s.deploy` Deployment · `k8s.ds` DaemonSet · `k8s.ep` Endpoints · `k8s.etcd` etcd · `k8s.group` Groupe · `k8s.hpa` HorizontalPodAutoscaler · `k8s.ing` Ingress · `k8s.job` Job · `k8s.k-proxy` kube-proxy · `k8s.kubelet` Kubelet · `k8s.limits` LimitRange · `k8s.netpol` NetworkPolicy · `k8s.node` Nœud · `k8s.ns` Namespace · `k8s.pod` Pod · `k8s.psp` PodSecurityPolicy · `k8s.pv` PersistentVolume · `k8s.pvc` PersistentVolumeClaim · `k8s.quota` ResourceQuota · `k8s.rb` RoleBinding · `k8s.role` Role · `k8s.rs` ReplicaSet · `k8s.sa` ServiceAccount · `k8s.sc` StorageClass · `k8s.sched` Scheduler · `k8s.secret` Secret · `k8s.sts` StatefulSet · `k8s.svc` Service · `k8s.user` Utilisateur · `k8s.vol` Volume |

- **Couleurs** : fond blanc, texte et bordure à l'encre du thème (voir « Thèmes ») ; le format posé à la main et les règles d'affichage les changent. Le champ `tags` n'existe plus : un bloc qui le contient est refusé, avec la marche à suivre (une propriété et une règle).
- **Relations** : `mode: async` ou `event` → trait pointillé ; libellé affiché « `label` [`protocol`] ».
- **Forme inconnue** (faute de frappe, ou famille non fournie) : la forme par défaut du type est utilisée et un avertissement `Forme inconnue : "…"` est émis.
- **Pas d'icônes cloud AWS / Azure / GCP en V1.** Les icônes et stencils de draw.io ne peuvent pas être utilisés (leur licence interdit tout usage, même dérivé, dans un produit Atlassian) ; les packs officiels des fournisseurs ne sont pas redistribuables en l'état. Représentez un service cloud par sa forme générique et indiquez-le dans `tech` (ex. `tech: AWS SQS`).
- `layout.nodes.<id>.style` accepte les alias `fill`, `stroke`, `color` (texte) et `width` (épaisseur), ou directement les clés de style maxGraph.

## Tracé des relations

1. Relation entre éléments placés par la même passe ELK → tracé ELK repris tel quel (ELK réserve la place des libellés).
2. Sinon (extrémités figées, ou dans des conteneurs différents), `route: orthogonal` (défaut) → **routage global** :
   - chaque relation est routée par un routeur orthogonal qui contourne les boîtes (grille de voies autour des éléments, marge 14 px, plus court chemin avec pénalité de 40 par coude ; les conteneurs qui contiennent la source ou la cible ne sont pas des obstacles) ;
   - les extrémités qui arrivent sur un même côté d'une boîte sont **réparties** le long de ce côté (pas de 14 px au plus), dans l'ordre de leur direction pour éviter les croisements ;
   - les segments parallèles qui se superposeraient sont **écartés** de 10 px, sans traverser de boîte ni sortir l'accroche de son côté ;
   - une accroche figée (`sourcePort` / `targetPort`) est respectée telle quelle.
   L'éditeur utilise exactement le même calcul : le tracé à l'écran est celui de l'enregistrement.
   Le routage a un **budget de calcul** (par tentative et pour tout le schéma) : au-delà, sur un schéma très chargé, les relations restantes sont tracées en ligne droite plutôt que de bloquer l'affichage.
3. **Sauts** : chaque flèche saute en arc (rayon 5 px) au-dessus des flèches dessinées avant elle, à chaque croisement franc (`jumps: none` sur une flèche ou dans `layout` pour les désactiver).
4. `points` figés → passage par ces points : avec `route: orthogonal` (défaut), tracé orthogonal calculé par core et dessiné tel quel ; avec `route: straight` ou `curved`, ligne brisée ou courbe dessinée par le rendu (tracé approché dans le rapport). Sans points, `route: straight` → segment direct ; `route: curved` → courbe.
5. Relation d'un élément vers lui-même → boucle.

## Rendu

Le rendu produit un **SVG autonome et déterministe** (même octet pour octet à chaque rendu) : polices **Arimo** (sans, métriquement identique à Arial), **Tinos** (serif, ≈ Times New Roman) et **Cousine** (mono, ≈ Courier New), licence OFL, embarquées en `@font-face` (seulement les familles utilisées par le schéma, et l'italique seulement s'il sert), textes en `<text>` SVG (pas de `foreignObject`), marges de 20 px.

### Thèmes

Le thème fixe les couleurs du schéma ; il se choisit **par bloc** (`layout.theme`). Trois thèmes, tous sur fond clair :

| Thème | Caractère | Encre (texte, contours, flèches) |
|---|---|---|
| `ardoise` (défaut) | Encre ardoise | `#243447` |
| `encre` | Noir et gris seulement, sobre, adapté à l'impression | `#161616` |
| `brique` | Encre brun chaud | `#3A2622` |

- **Thème personnalisé** : `theme` peut aussi être une définition, écrite dans le bloc. Toutes les clés sauf `name` sont facultatives ; une couleur absente vient du thème de base.

  ```yaml
  layout:
    theme:
      name: Maison            # nom affiché
      base: ardoise           # thème de départ : ardoise (défaut), encre, brique
      ink: "#1F3A5F"          # texte, contours, flèches
      fill: "#FFFFFF"         # fond des boîtes
      palette: ["#FFFFFF", "#1F3A5F", "#C2410C"]   # nuancier de l'éditeur (24 couleurs au plus)
  ```

  Une couleur mal écrite (attendu `#RRVVBB`) ou une base inconnue donne un avertissement ; la valeur est ignorée. Dans l'éditeur, le menu Thème propose « Créer un thème… » (à partir du thème en cours) puis un petit éditeur : nom, encre, fond, nuancier (ajouter, changer, retirer une couleur).
- Sans `theme`, Ardoise s'applique et rien n'est écrit dans le bloc.
- Un thème inconnu donne l'avertissement « Thème inconnu » et le rendu retombe sur Ardoise.
- Dans l'éditeur, le cartel affiche le thème quand rien n'est sélectionné ; le changer redessine tout le schéma, conserve les réglages de format, écrit `layout.theme` et s'annule comme le reste. Le nuancier propose les couleurs du thème en vigueur, puis les couleurs ajoutées au schéma (`layout.colors` : les 7 dernières choisies hors thème dans l'éditeur, écrites dans le bloc ; le × au survol en retire une, les objets gardent leur couleur) ; la pastille « Thème » revient à la couleur du thème ; « + » ouvre un sélecteur de couleur en HEX.
- Tous les contrastes texte / fond sont au moins AA. Le bleu `#2F5BEA` n'appartient à aucun thème : il est réservé à l'édition (sélection, poignées).

- Aperçu dans le navigateur : `pnpm --filter preview dev` puis `http://localhost:5173/` (les exemples) ou `/?src=<bloc encodé>`.
- En ligne de commande : `node apps/cli/bin/trustdiagram.mjs validate <fichier>…` et `node apps/cli/bin/trustdiagram.mjs render <fichier> -o sortie.svg|sortie.png` (ou `--all <dossier> -o <dossier>`). Rendus de référence : [`examples/renders/`](../examples/renders).

## Vérification de la mise en page

`trustdiagram lint` (et la commande Obsidian *Vérifier*) calcule la mise en page **telle qu'elle est dessinée** et la contrôle : boîtes, textes, tracés et libellés des flèches. Le résultat est un **rapport** déterministe (même source, même rapport) : la géométrie réelle, des anomalies typées avec une indication de correction, et un score. Un agent (ou une personne) corrige, relance, et recommence jusqu'à un score propre.

Dans Obsidian, les commandes *Vérifier*, *Exporter*, *Recalculer* et *Repartir de zéro* lisent la note **sur le disque** et écrivent en dernier un fichier de statut (`noteSha256`, empreinte du texte lu) : voir le [README du plugin](../apps/obsidian-plugin/README.md), « Commandes sur la note active ».

### Les 11 types d'anomalies

| Type | Gravité | Règle | Indication (`hint`) |
|---|---|---|---|
| `node_outside_zone` | erreur | une boîte sort, même en partie, du cadre de son parent | agrandir le parent du côté concerné (`layout.nodes.<parent>.x/y/w/h`) |
| `zone_overlap` | erreur | deux conteneurs frères (zones, groupes) se chevauchent | décaler le second, dessiné au-dessus, de la largeur ou hauteur du recouvrement, avec son contenu figé |
| `text_overflow` | erreur | le texte d'une boîte, tel que dessiné (lignes repliées dans la largeur réelle), sort de la boîte de plus de 0,5 px ; pour une forme à icône au-dessus du texte, aussi quand la place laissée à l'icône tombe sous son minimum | élargir ou agrandir la boîte du nombre de pixels qui manque (`layout.nodes.<id>.w/h`) |
| `edge_crosses_node` | erreur | une flèche traverse une boîte ou un groupe qui n'est ni sa source, ni sa cible, ni un de leurs ancêtres ; le fond d'une **zone** n'est pas un obstacle | accrocher la flèche du côté de la source qui fait face à la cible (`sourcePort`), en retirant les `points` figés ; seulement si cela change le tracé (aucune indication quand la flèche a déjà cette accroche et pas de `points`) |
| `edge_crosses_zone_title` | avertissement | une flèche coupe le titre d'une zone dont ni la source ni la cible ne fait partie | aucune |
| `label_overlaps_node` | avertissement | le libellé d'une flèche couvre une boîte ou un groupe, **sa source et sa cible comprises** (un libellé plus long que sa flèche déborde sur ses extrémités) ; ne comptent pas les zones ni les conteneurs de la flèche (ancêtres de la source ou de la cible) | `labelPos` à 24 px du milieu du tracé, du premier côté libre : ni boîte (extrémités comprises), ni titre de zone ou de groupe, ni autre libellé (aucune indication si aucun côté n'est libre) |
| `label_overlaps_zone_title` | avertissement | le libellé d'une flèche couvre le titre d'une zone ou d'un groupe, **y compris** d'une zone ou d'un groupe qui contient la flèche | idem |
| `label_overlaps_label` | avertissement | deux libellés de flèches se chevauchent | idem, pour le second libellé |
| `edge_detour` | avertissement | le tracé fait plus de 2,5 fois la distance `direct` **et** plus de 200 px de plus qu'elle | aucune |
| `edge_crossings` | information | deux flèches se croisent (c'est là que les sauts en arc sont dessinés) | aucune |
| `edge_geometry_approximate` | information | le tracé n'est pas calculé par core (voir `exact` ci-dessous) | pour une flèche `straight` ou `curved` entre deux éléments qui ne s'emboîtent pas : `route: orthogonal` |

Précisions :

- Tolérances : pour les textes (`text_overflow`) et les libellés de flèches (`label_overlaps_node`, `label_overlaps_zone_title`, `label_overlaps_label`), un débordement ou un chevauchement de 0,5 px ou moins ne compte pas ; pour `node_outside_zone` et `zone_overlap`, tout dépassement ou recouvrement compte, même d'un pixel.
- `direct` est la distance en ligne droite entre les **centres** de la source et de la cible ; `length` est la longueur du tracé.
- Les **calques masqués** figurent dans la géométrie (`hidden: true`) mais ne sont pas contrôlés, sauf avec `--all-layers` (`allLayers` dans l'API).
- Les contrôles sur les tracés (`edge_crosses_node`, `edge_crosses_zone_title`, `edge_detour`, `edge_crossings`) et sur les libellés ne portent que sur les flèches dont le tracé est exact.
- L'ordre des anomalies est stable : par type, puis par identifiants.

### `exact` et `edge_geometry_approximate`

Chaque flèche du rapport porte `exact`. Il est vrai quand le tracé (`points`) est calculé par core (ELK ou routeur) et dessiné tel quel. Il est faux quand le rendu le dessine lui-même, donc que le rapport ne peut qu'en donner une approximation : boucle d'un élément sur lui-même, flèche vers son propre conteneur, route `straight` ou `curved`, échec du routeur. Une flèche inexacte n'est pas examinée par les autres contrôles ; elle est signalée une fois, par `edge_geometry_approximate` (information, qui ne rend pas le schéma « sale »).

### Le rapport

Schéma JSON : [`packages/core/schema/layout-report.schema.json`](../packages/core/schema/layout-report.schema.json) (`reportVersion: 1`).

```json
{
  "reportVersion": 1,
  "diagram": { "id": "payments", "title": "Paiements", "sourceHash": "9f2c4e1ab07d3c5582e6f1a9d40b7c26e3f8a51d9b2c047e6a1f3d58c9e2b704" },
  "valid": true,
  "issues": [],
  "geometry": {
    "nodes": [
      { "id": "api", "kind": "box", "parent": null, "x": 40, "y": 40, "w": 160, "h": 60,
        "label": { "x": 52, "y": 58, "w": 96, "h": 32, "lines": ["Payments API", "[Go]"] }, "auto": true }
    ],
    "edges": [
      { "id": "r1", "from": "web", "to": "api", "points": [[200, 70], [320, 70]],
        "exact": true, "length": 120, "direct": 280, "label": { "x": 230, "y": 58, "w": 60, "h": 16 } }
    ]
  },
  "anomalies": [
    { "type": "text_overflow", "severity": "error", "ids": ["db"],
      "message": "le texte de db déborde de sa boîte", "at": [520, 70],
      "hint": { "text": "élargir db de 24 px", "patch": { "layout.nodes.db.w": 184 } } }
  ],
  "score": { "error": 1, "warning": 0, "info": 0, "clean": false }
}
```

- `valid: false` : le bloc ne passe pas la validation ; `issues` dit pourquoi, `geometry` vaut `null`, il n'y a pas d'anomalie.
- `geometry.nodes[]` : `kind` vaut `box`, `zone` ou `group` ; `parent` ; le cadre `x, y, w, h` (coordonnées du schéma) ; `label` (cadre et lignes du texte, `null` sans texte) ; `auto` (placé par la mise en page automatique, pas figé) ; `hidden` si calque masqué.
- `anomalies[]` : `ids` (l'objet en faute d'abord, puis l'objet de référence), `message`, `at` (point où elle se voit), `hint` facultatif. `hint.patch` associe un chemin pointé du bloc à la valeur proposée (`null` : retirer la clé) ; **il n'est jamais appliqué d'office**, c'est à l'agent ou à la personne de décider.
- `score` : nombres d'erreurs, d'avertissements et d'informations ; `clean` est vrai quand il n'y a ni erreur ni avertissement.
- `sourceHash` : empreinte du bloc (voir « Empreintes »), pour savoir si un rapport correspond encore au texte.

### `labelPos`

`layout.edges.<id>.labelPos` fixe le **centre** du libellé d'une flèche (coordonnées absolues), pas son coin. Sans lui, le libellé est centré sur le milieu de la longueur du tracé. C'est la cible des indications de `label_overlaps_node`, `label_overlaps_zone_title` et `label_overlaps_label`.

### En ligne de commande

```
trustdiagram lint <fichier.md|fichier.trustdiagram.yaml>… [--json] [--all-layers]
trustdiagram render <note.md> -o <dossier> [--format svg|png|svg,png]
```

- `lint` accepte des notes Markdown (tous les blocs ` ```trustdiagram `, sauf ceux montrés en exemple à l'intérieur d'une autre clôture, ` ````markdown ` ou `~~~`) et des fichiers `.trustdiagram.yaml`. En texte, une ligne par anomalie (`fichier#bloc : gravité type — message`), puis un total. Avec `--json`, un objet `{ "files": [{ "file", "fileIds": [identifiant…], "diagrams": [rapport…] }] }` : les rapports sont ceux de l'API `lintLayout` (tels quels, conformes au schéma) ; `fileIds[i]` est l'identifiant de fichier (voir « Noms de fichiers ») du rapport `diagrams[i]`, le même que `fichier#bloc` en texte et que `id` dans le statut du plugin. Une note sans bloc donne l'avertissement « aucun schéma trouvé » (sur la sortie d'erreur) et le code de sortie 0.
- Code de sortie : **0** aucune erreur ; **1** au moins une erreur d'anomalie ou un bloc invalide (les avertissements et informations ne font pas échouer) ; **2** usage incorrect ou fichier introuvable.
- `render` sur une note écrit une image par bloc valide dans le dossier (`svg` par défaut ; `--format svg,png` pour les deux) ; un bloc invalide est signalé et le code de sortie est 1. Le PNG est rendu à l'échelle ×2 (deux fois la largeur et la hauteur du SVG), comme l'export du plugin.

### Noms de fichiers

Les fichiers écrits pour une note (CLI et plugin Obsidian, mêmes règles) s'appellent `<note>.<bloc>.<extension>` :

- `<note>` (`exportBaseName`) : le chemin de la note sans `.md`, avec `/` remplacé par `__`. Dans le plugin, c'est le chemin dans le vault ; dans la CLI, le chemin **relatif au dossier courant** (qu'il soit tapé en absolu ou avec `./`), ou seulement le nom du fichier pour une note hors du dossier courant : `trustdiagram render notes/a.md` comme `trustdiagram render ./notes/a.md` donnent `notes__a.<bloc>.svg` ;
- `<bloc>` (`diagramFileIds`) : l'`id` du bloc, ou `bloc-<n>` (rang du bloc dans la note, à partir de 1) s'il n'en a pas ; un doublon reçoit le suffixe `-2`, `-3`…
- les caractères `\ / : * ? " < > |` et les caractères de contrôle deviennent `_`, et un segment fait uniquement de points (`.`, `..`) aussi : un nom reste toujours dans le dossier de sortie.

Exemple : la note `archi/paiements.md` avec un bloc d'id `payments` et un bloc sans id donne `archi__paiements.payments.svg` et `archi__paiements.bloc-2.svg`.

## Éditeur graphique

L'éditeur (plugin Obsidian, puis app Confluence) se manipule comme draw.io ; il écrit lui-même le YAML, en réécriture ciblée (commentaires, alignements et entrées non concernées restent identiques ; une liste garde son style bloc ou flow) :

- **Ce que l'on voit est ce que l'on enregistre.** Dès qu'un geste touche la géométrie (déplacer, redimensionner, aligner, poser un élément) ou la structure (relier, raccrocher, supprimer, modifier le texte d'une flèche, renommer une boîte qui change de taille), **toutes les boîtes sont figées** à leur position affichée dans `layout.nodes` (`x`, `y` ; `w`, `h` pour les conteneurs et les boîtes redimensionnées). Sinon les boîtes restées libres seraient replacées au rendu et le schéma enregistré ne ressemblerait plus à l'écran.
- **Flèches** : tant qu'on ne touche pas à leur tracé, elles sont retracées en direct (même routeur que le rendu) et rien n'est écrit pour elles. Tirer un segment ou un point de passage fige le tracé (`layout.edges.<id>.points`, coordonnées absolues).
- **Double-clic** sur une boîte → modifie son texte visible « nom [technologie] » : le texte entre crochets final est la `tech` (le retirer la supprime) ; une boîte sans taille imposée s'adapte au nouveau texte ; sur une flèche → tout son texte visible « libellé [protocole] » : le texte entre crochets final est le `protocol` (le retirer supprime le protocole).
- **Raccrocher** une extrémité de flèche à une autre boîte → modifie `from` / `to` ; son tracé redevient automatique.
- **Relier** : au survol d'une boîte, une flèche apparaît sur son bord droit ; la tirer vers une autre boîte crée une relation `id` = `r-<source>-<cible>` (suffixe `-2`, `-3`… si l'id existe).
- **Catalogue** (colonne de gauche) : cliquer une forme l'ajoute au centre de la vue, la glisser la pose au point du lâcher. Le type de l'élément découle de la forme (cylindre → `datastore`, file → `queue`, personne → `person`, réseau → `node`, Kubernetes → `container`…) ; la forme est écrite dans `layout` si ce n'est pas celle du type. `id` = `<type>-<n>` (premier `n` libre), `name: Nouvel élément`, ajouté au conteneur sous le point (sinon à la racine).
- **Supprimer** un élément → l'élément, ses enfants, les relations qui les touchent et leurs entrées `layout`.
- **Position du texte** : grille 3 × 3 de la section Texte du cartel → `style.align` (revenir à la position par défaut retire la clé ; le pinceau la copie).
- **Style du texte** : boutons Gras / Italique / Souligné et liste des tailles de la section Texte → `style.bold`, `italic`, `underline`, `size` ; police de l'objet en mode avancé → `style.font`. Sans sélection, **Police** choisit la police de base du schéma → `layout.font`. Revenir à la valeur par défaut retire la clé.
- **Format** (panneau de droite) → `layout.nodes.<id>.style` / `layout.edges.<id>.style` en clés lisibles.
- **Accroche** : faire glisser l'extrémité d'une flèche le long du contour de la même boîte → `sourcePort` / `targetPort` ; le reste du tracé reste automatique.
- **Premier plan / Arrière-plan / Avancer / Reculer** → `layout.nodes.<id>.z`. **Ranger en ligne / en colonne** : côte à côte ou l'une sous l'autre, écart de 40 px, centres alignés sur la première boîte.
- **Calques** (panneau sous Format) : ajouter, renommer, masquer, verrouiller, monter/descendre, supprimer (les éléments reviennent au calque par défaut), mettre la sélection dans un calque → `layout.layers` et `layout.nodes.<id>.layer`.
- **Forme…** → `layout.nodes.<id>.shape`. **Replacer** → supprime l'entrée `layout.nodes.<id>` et applique tout de suite le placement automatique à l'écran.
- Revenir à l'état initial (Annuler jusqu'au début) rend le texte d'origine à l'identique.
- **Copier / Coller / Dupliquer** (Ctrl/Cmd+C, V, D) : copie les boîtes sélectionnées avec le contenu des zones et les relations **entre** éléments copiés (pas celles vers l'extérieur). Le collage crée de nouveaux éléments (`id` = `<type>-<n>`) avec `name`, `tech`, `description`, `ref`, propriétés, classes, forme, taille et format ; il est décalé de 20 px à chaque collage. Dans le même schéma, la copie va dans le même conteneur (ou le même calque) que l'original ; dans un autre schéma, dans le conteneur sous la copie. Les relations collées ont un tracé automatique.
- **Pinceau** (🖌) : prend le format de la sélection ; l'objet sélectionné ensuite le reçoit, en **remplaçant** son format. Double-clic sur le pinceau : il reste actif jusqu'à Échap. D'une boîte vers une flèche (ou l'inverse), seuls les réglages communs passent (contour, épaisseur, trait, couleur du texte). Ctrl/Cmd+Maj+C / Ctrl/Cmd+Maj+V : copier / coller le style.
- **Flèches** : épaisseur par préréglages (1 à 4) ou valeur libre (`style.width`) ; **Sens** : vers la cible (défaut), double sens (`style.start: block`) ou sans pointe (`style.end: none`) ; **Inverser le sens** échange `from` et `to` dans le modèle. Les coudes des flèches sont légèrement arrondis (rayon 6 px ; `style.rounded: 0` pour des angles vifs).
- **Déplacer la vue** : glisser sur le fond déplace tout le schéma (curseur main) ; Maj + glisser trace le rectangle de sélection. Les boutons main / flèche des commandes de vue inversent ce choix (mémorisé).
- **Changer de zone** : lâcher une boîte sur une zone (ou un groupe) la range dans cette zone (la zone cible est surlignée) ; la lâcher dans le vide hors de sa zone la sort au premier niveau, dans le calque de sa zone d'origine. Dans le YAML, l'élément (avec ses enfants et ses commentaires) est déplacé dans `children` de la nouvelle zone ou dans `model.elements` ; relations et positions ne changent pas.
- Non autorisé à la souris en V1 : dupliquer par Ctrl+glisser.

## Empreintes (hash)

Pour la synchronisation, chaque bloc a trois empreintes SHA-256 calculées sur un **JSON canonique** (clés triées, sans espaces) — elles ignorent donc commentaires, indentation, style flow/bloc et fins de ligne :

- `model` : tout le bloc sauf `layout` ;
- `layout` : la section `layout` ;
- `block` : combinaison des deux.

## Fusion à 3 voies

Quand un bloc a été modifié des deux côtés (vault et Confluence) depuis une version commune `base` :

- **Modèle** : modifié d'un seul côté → ce côté gagne ; modifié des deux côtés à l'identique → rien à faire ; modifié différemment → **conflit**, signalé par élément (`/model/elements/<id>`, à toute profondeur), par relation (`/model/relations/<id>`) ou par champ (`/title`, `/description`…).
- **Mise en page**, par entrée puis par champ : modifiée d'un seul côté → ce côté gagne ; des deux côtés à l'identique → rien à faire ; même champ modifié différemment → **conflit** `{ key, field }` ; supprimée d'un côté et modifiée de l'autre → conflit `{ key, field: "*" }`.
- En cas de conflit, le texte proposé garde la valeur locale (`ours`) et applique tout ce qui est fusionnable.
- Le texte de départ est celui du côté dont le modèle gagne : ses commentaires et sa mise en forme sont conservés.

## Exemple commenté

```yaml
trustdiagram: 1
id: plateforme-llm-conteneurs
title: Plateforme LLM — vue conteneurs
kind: flow
description: Comment les applications métiers consomment les LLM via la passerelle.
model:
  elements:
    - id: app
      type: system
      name: Application métier
    - id: zone-si                # une zone : périmètre d'hébergement
      type: zone
      name: SI interne
      children:
        - id: gw
          type: container
          name: Passerelle LLM
          tech: LiteLLM
          description: Point d'entrée unique, quotas, journalisation.
          ref: "[[Passerelle LLM]]"
        - id: logs
          type: datastore
          name: Journaux
  relations:
    - id: r1
      from: app
      to: gw
      label: Appel d'inférence
      protocol: HTTPS/REST
      mode: sync
    - id: r2
      from: gw
      to: logs
      label: Journalise
      mode: async
layout:
  nodes:
    zone-si: { x: 40, y: 40, w: 600, h: 400 }    # figé
    gw: { x: 320, y: 140, w: 160, h: 60, shape: aws.api-gateway }
    # app et logs n'ont pas d'entrée : placés automatiquement
  edges:
    r1: { route: orthogonal, points: [[400, 260]] }
```

Exemples : [`examples/`](../examples) (pipeline CI/CD entièrement placé, boutique en ligne et contexte d'un système de prêt en placement automatique, application de partage de photos en couches de haut en bas, règles d'affichage). Les schémas en placement mixte (zones figées, contenu placé automatiquement) sont dans les fixtures de test (`packages/core/test/fixtures/`).

## Validation LLM

Contrôle du 2026-11-10 : un agent neuf (modèle léger, sans autre contexte) a reçu **uniquement** [`packages/core/test/fixtures/place-zones.trustdiagram.yaml`](../packages/core/test/fixtures/place-zones.trustdiagram.yaml) (assistant documentaire RAG, devenu fixture de test) et 4 questions.

| Question | Résultat |
|---|---|
| Flux sortant de la zone « Orchestrateurs » vers d'autres zones | ✅ 3 flux exacts (`f-recherche`, `f-generation`, `f-traces`), avec zone cible, libellé et protocole ; flux internes et entrants correctement écartés |
| Protocole entre l'orchestrateur et le proxy LLM | ✅ « API compatible OpenAI » via `f-generation` |
| Composant de journalisation | ✅ `journal` (Datadog), alimenté par `orchestrateur` et `llm-proxy` |
| Validation humaine ? | ✅ « non », justifiée par la `description` du schéma |

Points signalés comme ambigus par l'agent, et correction apportée :
- `mode` absent sur les appels principaux → précisé dans l'exemple (`sync`) et règle « absent = non précisé » ajoutée ci-dessus ;
- contenu du flux de génération non dit → champ `data` ajouté sur `f-recherche` et `f-generation`, et recommandé dans la spécification.
