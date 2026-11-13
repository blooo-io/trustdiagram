# `trustdiagram` format v1: architecture diagrams as code

Version française : [docs/format.fr.md](format.fr.md)

A ` ```trustdiagram ` block describes **one architecture view** in an Obsidian note or a Confluence page.
It is written in YAML and has two parts that never mix:

| Part | Role | Who writes it |
|---|---|---|
| `model` (with `title`, `description`) | **The meaning**: elements, nesting, flows, protocols, links to notes. Enough for a person or an LLM to understand the architecture. | The architect, or an LLM |
| `layout` | **The drawing**: positions, sizes, shapes, styles. Carries no architecture information. | Mostly the graphical editor |

> **Golden rule.** Moving, resizing or restyling a box changes **only** `layout`.
> `model`, the comments and the formatting of the rest of the block stay identical byte for byte.
> Only explicitly creating or deleting an element in the editor touches `model`.

Formal validation: [`packages/core/schema/trustdiagram.schema.json`](../packages/core/schema/trustdiagram.schema.json) (JSON Schema 2020-12).

## For an LLM reading a block

- Ignore `layout` to understand the architecture: all the meaning is in `model`, `title` and `description`.
- Nesting (`children`) means **membership or hosting**: an element is inside a zone (network, information system, cloud account), a group (tier, domain) or a system.
- A relation `from → to` points in the **direction of the main flow** (who calls, who sends). `label` is the verb, `protocol` the technical means, `mode` the nature of the exchange.
- `ref: "[[Note]]"` points to a vault note that details the element. A status (existing, new, target…) is written as a property (`properties: { status: new }`); a display rule derives the color from it (see "Display rules").

## Structure

```yaml
trustdiagram: 1                       # format version (required)
id: plateforme-llm-conteneurs       # stable diagram identifier (required), used for sync
title: Plateforme LLM — vue conteneurs
kind: flow                     # flow (v1); sequence comes in phase 6
description: Phrase ou paragraphe qui dit ce que montre la vue.
model:
  elements: [...]              # required
  relations: [...]
layout: {...}                  # optional
```

### Identifiers

All `id` values (diagram, elements, relations) match `^[a-z0-9][a-z0-9-_]*$`.
Element and relation identifiers are **unique across the whole block** (an element and a relation cannot share an id).

### Element (`model.elements[]`, recursive through `children`)

| Field | Required | Values | Meaning |
|---|---|---|---|
| `id` | yes | identifier | Referenced by relations and by `layout` |
| `type` | yes | `person`, `system`, `container`, `component`, `datastore`, `queue`, `node`, `zone`, `group`, `external` | Kind of element (see below) |
| `name` | yes | non-empty text | Displayed label |
| `tech` | no | text | Technology, product, exposure protocol |
| `description` | no | text | Responsibility of the element |
| `ref` | no | `"[[Note]]"` | Vault note that details it |
| `children` | no | list of elements | Contained elements |

| `type` | Use |
|---|---|
| `person` | User, role, human actor |
| `system` | System or application seen as a whole |
| `container` | Deployable unit of a system (application, service, container) |
| `component` | Component inside a container |
| `datastore` | Database, index, storage, data lake |
| `queue` | Queue, topic, message bus |
| `node` | Infrastructure element (VM, cluster, device, appliance) |
| `zone` | Hosting or security perimeter (information system, cloud account, network zone). Always a container |
| `group` | Logical grouping (tier, domain). Always a container |
| `external` | Third-party service, out of scope |

An element is a **container** if it has `children` or if its type is `zone` or `group`.

### Relation (`model.relations[]`)

| Field | Required | Values | Meaning |
|---|---|---|---|
| `id` | yes | identifier | Referenced by `layout.edges` |
| `from`, `to` | yes | element id | Direction of the main flow; a container is a valid end |
| `label` | no | text | Verb or nature of the flow |
| `protocol` | no | text | `HTTPS`, `MCP`, `SQL`, `4G`… |
| `mode` | no | `sync`, `async`, `event` | Synchronous call, asynchronous exchange (traces, deferred sends), event. **Missing = not specified**: fill it in as soon as you know it, an LLM does not guess it |
| `data` | no | list of texts | Data exchanged. Recommended on the main flows: it answers "what does this flow carry?" |
| `description` | no | text | Details (authentication, constraints, security) |

### Layout (`layout`)

```yaml
layout:
  theme: encre                                   # diagram theme: ardoise (default, not written), encre or brique
  colors: ["#C03A64", "#5A7A4A"]                # colors added to this diagram's swatches (written by the editor)
  font: serif                                    # base text font: sans (Arimo, default, not written), serif (Tinos), mono (Cousine)
  direction: right                               # auto layout direction: right (default), down, left, up
  jumps: arc                                     # arc jumps where arrows cross: arc (default) or none
  layers:                                        # layers, from back to front (implicit "defaut" layer, at the very back)
    - { id: infra, name: Infrastructure }
    - { id: flux, name: Flux de données, visible: false, locked: true }
  nodes:                                         # key = element id
    gw: { x: 320, y: 140, w: 160, h: 60, shape: aws.api-gateway, style: { fill: "#FFE6CC" } }
  edges:                                         # key = relation id
    r1: { route: orthogonal, points: [[400, 260]], labelPos: [410, 250], style: { stroke: "#FF9900" } }
```

- **Layers**: `layers` lists the layers from back to front (`id`, `name`, `visible`, `locked`); `nodes.<id>.layer` puts a **top-level** element in a layer (its children follow; an arrow follows its source). A hidden layer is **not drawn** in the render (nor are the arrows that touch its elements) but keeps its place in the auto layout; a locked layer cannot be edited in the editor. An unknown layer gives a warning.
- **Stacking order**: `nodes.<id>.z` (integer, 0 by default); between siblings, the drawing order is layer, then `z`, then model order.
- `nodes.<id>`: `x`, `y` required (top-left corner, **absolute coordinates** in pixels, integers); `w`, `h` optional; `shape` in `family.name` form (`k8s.pod`, `c4.person`, `network.firewall`; see "Shapes"); `style` = free key/value pairs.
- `edges.<id>`: `route` (`orthogonal`, `straight`, `curved`), `points` (waypoints), `labelPos` (label center, absolute coordinates; by default, the middle of the path length), `sourcePort` / `targetPort` (fixed attachment on the outline of the source / target, relative position `[rx, ry]` with values from 0 to 1: `[1, 0.25]` = right edge, a quarter of the way down), `style`.
- **Style** (`style` of a node or an arrow), readable keys: `fill` (color or `none`), `stroke` (outline or line color), `width` (thickness, 0.5 to 8), `dash` (`solid`, `dashed`, `dotted`), `color` (text color), for boxes `align` (text position: `top-left`, `top`, `top-right`, `left`, `center`, `right`, `bottom-left`, `bottom`, `bottom-right`; `center` by default, and `top-left` for a zone or a group; no effect on icon shapes, whose text stays below the icon) and, for arrows, `start` / `end` (`none`, `block`, `open`, `diamond`, `oval`; in the editor, drawn buttons in Advanced mode). **Text** (boxes and arrows): `size` (name size in px, 8 to 48; 13 by default for a box, 11 for an arrow; the `[technology]` and line spacing follow the size), `font` (`sans`, `serif`, `mono`; by default the base font `layout.font`), `bold` (`true` / `false`; a box name is bold by default, an arrow label is not), `italic` and `underline` (`true`). Example: `style: { size: 16, italic: true, bold: false }`. An out-of-range value gives a warning; the render clamps or ignores it. Raw maxGraph style keys are still accepted.
- Each entry fits **on one line** (YAML flow style) to keep the section short and diffs readable.
- An entry whose id no longer exists in `model` is **orphaned**: it produces a warning, is ignored in the render and is kept by the merge.

## Properties, classes and display rules

A diagram can carry **data** (name / value properties on objects) and **rules** that derive the display from it, like conditional formatting in Lucidchart. Properties and classes are **meaning** (`model`); rules are **display** (`layout`).

### Properties (`properties`)

On an element (at any depth) or a relation: scalars (text, number, boolean; `null` = not filled in). A value is displayed **as written** (`version: 2.10` is displayed "2.10"); a non-scalar value (list, object) is an error.

```yaml
    - id: gw
      type: container
      name: Passerelle {{version}}
      classes: [composant-applicatif]
      properties: { version: 2.10, exposé_internet: true }
```

### Classes (`model.classes`, `classes`)

A class is a property template with default values; an object carries a `classes` list. **Effective value**: the classes in list order (the last one wins), then the object's `properties`. Inherited values are never copied into the object. An unknown class gives a warning; a duplicate class identifier gives an error.

```yaml
model:
  classes:
    - id: composant-applicatif
      name: Composant applicatif
      properties: { responsable: null, criticité: moyenne, exposé_internet: false }
```

### References in text (`{{…}}`)

In an element's `name` and `tech` and a relation's `label`, `{{name}}` displays the effective value of the property. Pseudo-properties: `{{id}}`, `{{type}}`, `{{tech}}` (element), `{{protocol}}`, `{{mode}}` (relation). Booleans are displayed as "oui" / "non" (yes / no); for an unknown or empty property nothing is displayed, with a warning. The raw text stays readable: `name: Passerelle {{version}}`.

### Rules (`layout.rules`)

Ordered list; for a given style key, **the last true rule wins**. **A rule wins over formatting set by hand** (`layout.*.style`), which itself wins over the theme and the state.

```yaml
layout:
  rules:
    - name: Exposé sur Internet          # required: shown in "Applied rules"
      when: { exposé_internet: true }    # scalar = "equals"
      then: { stroke: "#C0392B", width: 2, icon: { name: shield-alert, position: top-right } }
    - name: Version obsolète
      when: { version: { lt: 2 } }
      then: { fill: "#FDEEDB" }
    - name: Critique ou exposé
      any: true                          # at least one condition true (otherwise: all)
      when: { criticité: haute, exposé_internet: true }
      then: { color: "#A63A2C" }
    - name: Brouillon
      enabled: false                     # disabled rule, kept in the block
      when: { statut: brouillon }
      then: { dash: dashed }
```

- **Operators**: `eq`, `ne`, `contains` (text), `gt`, `gte`, `lt`, `lte` (numbers), `in` (list of values), `set` (`true`: filled in; `false`: empty). One operator per condition. Text equality is case-insensitive; `"2.4"` and `2.4` are equal; `"true"` equals `true`.
- **Having the property**: the key exists on the object or in one of its classes, even if empty. An object that does not have the property satisfies **no** test, `set: false` included (a "No owner" rule only targets objects that expect an owner).
- **Actions** (`then`): `fill`, `stroke`, `width`, `dash`, `color` (text color), and `icon: { name, position, color? }` for boxes, zones and groups. On a relation, only `stroke`, `width`, `dash` and `color` apply.
- **Icon**: `position` is one of the 9 `align` positions; several icons at the same position line up side by side, toward the inside; an **auto-sized** box reserves room for them (a free band above and below the text for a top or bottom icon, on the sides for `left` / `right`), a box with a fixed size (`w`, `h`) keeps its size; `color` defaults to the rule's `stroke`, otherwise the theme ink. Available names (Lucide, ISC license): `shield-alert`, `shield-check`, `lock`, `lock-open`, `globe`, `triangle-alert`, `circle-alert`, `circle-check`, `circle-x`, `clock`, `flame`, `star`, `flag`, `bug`, `database`, `cloud`, `server`, `key-round`, `user`, `users`, `eye`, `eye-off`, `zap`, `wrench`, `archive`, `ban`, `info`, `badge-check`, `activity`, `gauge`, `hourglass`, `link`, `unlink`, `wifi-off`, `cpu`, `hard-drive`, `network`, `file-warning`, `trash-2`, `sparkles`.
- **Robustness**: a badly written rule never blocks the diagram. Without a name or a condition, it is ignored; a malformed action (color that is not `#RRGGBB`, thickness outside 0.5–8, unknown line or icon, unknown field) is ignored on its own. Each case gives a warning with its path (`/layout/rules/2/then/icon/name`). Unambiguous spellings are corrected: `width: "2"` means 2, `enabled: "false"` means false.

## Size limits

The render runs on each reader's side (Confluence macro, Obsidian): an oversized block is rejected with an error rather than blocking the page.

| Limit | Value |
|---|---|
| Block size | 512 KB |
| Name, `tech`, relation label or protocol, title | 1,000 characters |
| Elements (children included) | 1,500 |
| Relations | 3,000 |
| Classes | 300 |
| Layers | 100 |
| Rules | 200 |

## Auto layout

Any element **without an entry** in `layout.nodes` is placed automatically at render time, deterministically (same result in Obsidian, Confluence and the CLI). It is written to `layout` ("pinned") only when you edit it or run "Pin layout". This is what keeps `layout` short.

Engine: ELK `layered`, direction given by `layout.direction` (`right` by default; `down`, `left`, `up` like `LR`, `TB`, `RL`, `BT` in Mermaid), hierarchy included, orthogonal edges, `randomSeed: 1`, spacing 40 px, container padding: top 40, left 20, bottom 20, right 20.

Rules, applied recursively to each container (and to the root):

1. **Size of a free box**, measured with the real advance widths of the embedded **Arimo** font (metrically identical to Arial): width = `max(120, ⌈max(width of the name in bold 13 px, width of "[tech]" in 11 px)⌉ + 24)`; height = `max(60, 12 + 18 + (tech ? 15 : 0) + 12)`. A character missing from the table (emoji…) counts as the average advance width.
2. **Pinned node**: its position is used as is; without `w`/`h` it takes the computed size. A **pinned container without a size** wraps its children (minimum 120 × 60).
3. **Free children of a container**:
   - if it has no pinned child, ELK places them starting from its inner corner `(x + 20, y + 40)`;
   - if it has pinned children, ELK places them to the right of those: starting at `(max x of the pinned ones + 80, min y of the pinned ones)`.
   - At the root with no pinned node at all, ELK places the whole diagram.
4. **Free container holding pinned elements**: its box is computed to wrap its children (with the padding above).
5. **Edges**: see "Relation routing" below.
6. All produced coordinates are integers.

> Tip: for stacked pinned zones, give them `w` and `h`; otherwise their height depends on their content and may overlap the next zone. For a vertical reading, use `direction: down` (top to bottom, like the `photo-sharing` example) or `direction: up` (bottom to top).

## Shapes

The `layout.nodes.<id>.shape` field chooses an element's shape. Without `shape`, each type has its default shape:

| `type` | Default shape |
|---|---|
| `person` | `general.person` (icon, label below; 40 px reserved for the icon) |
| `system`, `container`, `external` | `general.rounded` |
| `component` | `uml.component` |
| `datastore` | `general.cylinder` |
| `queue` | `general.queue` |
| `node` | `uml.node` |
| `zone` | `general.zone` (solid border) |
| `group` | `general.group` (light fill, dotted border) |

V1 catalog:

| Family | Shapes |
|---|---|
| General | `general.box` Rectangle · `general.rounded` Rounded rectangle · `general.cylinder` Database · `general.queue` Message queue · `general.document` Document · `general.cloud` Cloud · `general.person` Person · `general.hexagon` Hexagon · `general.folder` Folder · `general.ellipse` Ellipse · `general.diamond` Diamond · `general.note` Note · `general.zone` Zone · `general.group` Group |
| C4 | `c4.person` Person (C4) · `c4.system` Software system (C4) · `c4.container` Container (C4) · `c4.component` Component (C4) · `c4.database` Database (C4) · `c4.external` External system (C4) |
| UML | `uml.component` Component (UML) · `uml.package` Package · `uml.node` Node (UML) · `uml.interface` Interface · `uml.actor` Actor (UML) · `uml.artifact` Artifact |
| Network | `network.server` Server · `network.firewall` Firewall · `network.load-balancer` Load balancer · `network.router` Router · `network.switch` Switch · `network.desktop` Desktop · `network.laptop` Laptop · `network.mobile` Mobile · `network.internet` Internet · `network.storage` Storage |
| Kubernetes (official icons) | `k8s.api` API server · `k8s.c-c-m` Cloud controller manager · `k8s.c-m` Controller manager · `k8s.c-role` ClusterRole · `k8s.cm` ConfigMap · `k8s.control-plane` Control plane · `k8s.crb` ClusterRoleBinding · `k8s.crd` CustomResourceDefinition · `k8s.cronjob` CronJob · `k8s.deploy` Deployment · `k8s.ds` DaemonSet · `k8s.ep` Endpoints · `k8s.etcd` etcd · `k8s.group` Group · `k8s.hpa` HorizontalPodAutoscaler · `k8s.ing` Ingress · `k8s.job` Job · `k8s.k-proxy` kube-proxy · `k8s.kubelet` Kubelet · `k8s.limits` LimitRange · `k8s.netpol` NetworkPolicy · `k8s.node` Node · `k8s.ns` Namespace · `k8s.pod` Pod · `k8s.psp` PodSecurityPolicy · `k8s.pv` PersistentVolume · `k8s.pvc` PersistentVolumeClaim · `k8s.quota` ResourceQuota · `k8s.rb` RoleBinding · `k8s.role` Role · `k8s.rs` ReplicaSet · `k8s.sa` ServiceAccount · `k8s.sc` StorageClass · `k8s.sched` Scheduler · `k8s.secret` Secret · `k8s.sts` StatefulSet · `k8s.svc` Service · `k8s.user` User · `k8s.vol` Volume |

- **Colors**: white fill, text and border in the theme ink (see "Themes"); formatting set by hand and display rules change them. The `tags` field no longer exists: a block that contains it is rejected, with the way forward (a property and a rule).
- **Relations**: `mode: async` or `event` → dotted line; label displayed as "`label` [`protocol`]".
- **Unknown shape** (typo, or family not provided): the type's default shape is used and the warning `Unknown shape: "…"` is emitted (in French: `Forme inconnue : "…"`).
- **No AWS / Azure / GCP cloud icons in V1.** draw.io icons and stencils cannot be used (their license forbids any use, even derived, in an Atlassian product); the vendors' official packs cannot be redistributed as is. Represent a cloud service with its generic shape and state it in `tech` (e.g. `tech: AWS SQS`).
- `layout.nodes.<id>.style` accepts the aliases `fill`, `stroke`, `color` (text) and `width` (thickness), or maxGraph style keys directly.

## Relation routing

1. Relation between elements placed by the same ELK pass → the ELK path is used as is (ELK reserves room for the labels).
2. Otherwise (pinned ends, or ends in different containers), `route: orthogonal` (default) → **global routing**:
   - each relation is routed by an orthogonal router that goes around the boxes (grid of lanes around the elements, 14 px margin, shortest path with a penalty of 40 per bend; containers that hold the source or the target are not obstacles);
   - ends that arrive on the same side of a box are **spread** along that side (14 px step at most), in the order of their direction to avoid crossings;
   - parallel segments that would overlap are **pushed apart** by 10 px, without crossing a box or moving the attachment off its side;
   - a fixed attachment (`sourcePort` / `targetPort`) is used as is.
   The editor uses exactly the same computation: the path on screen is the one that is saved.
   Routing has a **computation budget** (per attempt and for the whole diagram): beyond it, on a very busy diagram, the remaining relations are drawn as straight lines rather than blocking the display.
3. **Jumps**: each arrow jumps in an arc (5 px radius) over the arrows drawn before it, at each clean crossing (`jumps: none` on an arrow or in `layout` turns them off).
4. Pinned `points` → the path goes through these points: with `route: orthogonal` (default), orthogonal path computed by core and drawn as is; with `route: straight` or `curved`, polyline or curve drawn by the renderer (approximate path in the report). Without points, `route: straight` → direct segment; `route: curved` → curve.
5. Relation from an element to itself → loop.

## Rendering

The render produces a **self-contained, deterministic SVG** (identical byte for byte on every render): fonts **Arimo** (sans, metrically identical to Arial), **Tinos** (serif, ≈ Times New Roman) and **Cousine** (mono, ≈ Courier New), OFL license, embedded as `@font-face` (only the families the diagram uses, and italic only if it is used), text as SVG `<text>` (no `foreignObject`), 20 px margins.

### Themes

The theme sets the diagram colors; it is chosen **per block** (`layout.theme`). Three themes, all on a light background:

| Theme | Character | Ink (text, outlines, arrows) |
|---|---|---|
| `ardoise` (default) | Slate ink | `#243447` |
| `encre` | Black and grays only, sober, suited to printing | `#161616` |
| `brique` | Warm brown ink | `#3A2622` |

- **Custom theme**: `theme` can also be a definition, written in the block. All keys except `name` are optional; a missing color comes from the base theme.

  ```yaml
  layout:
    theme:
      name: Maison            # displayed name
      base: ardoise           # starting theme: ardoise (default), encre, brique
      ink: "#1F3A5F"          # text, outlines, arrows
      fill: "#FFFFFF"         # box fill
      palette: ["#FFFFFF", "#1F3A5F", "#C2410C"]   # editor swatches (24 colors at most)
  ```

  A badly written color (expected `#RRGGBB`) or an unknown base gives a warning; the value is ignored. In the editor, the Theme menu offers "Create a theme…" (starting from the current theme), then a small editor: name, ink, fill, swatches (add, change, remove a color).
- Without `theme`, Ardoise applies and nothing is written to the block.
- An unknown theme gives the warning `Unknown theme: "…"` (in French: « Thème inconnu ») and the render falls back to Ardoise (Slate in the English interface).
- In the editor, the Format panel shows the theme when nothing is selected; changing it redraws the whole diagram, keeps the formatting settings, writes `layout.theme` and can be undone like everything else. The swatches offer the colors of the current theme, then the colors added to the diagram (`layout.colors`: the last 7 colors picked outside the theme in the editor, written to the block; the × on hover removes one, objects keep their color); the "Theme" swatch goes back to the theme color; "+" opens a HEX color picker.
- All text / background contrasts are at least AA. The blue `#2F5BEA` belongs to no theme: it is reserved for editing (selection, handles).

- Browser preview: `pnpm --filter preview dev` then `http://localhost:5173/` (the examples) or `/?src=<encoded block>`.
- Command line: `node apps/cli/bin/trustdiagram.mjs validate <file>…` and `node apps/cli/bin/trustdiagram.mjs render <file> -o output.svg|output.png` (or `--all <folder> -o <folder>`). Reference renders: [`examples/renders/`](../examples/renders).

## Layout check

`trustdiagram lint` (and the Obsidian *Check* command) computes the layout **as it is drawn** and checks it: boxes, text, arrow paths and arrow labels. The result is a deterministic **report** (same source, same report): the real geometry, typed anomalies with a fix hint, and a score. An agent (or a person) fixes, reruns, and repeats until the score is clean.

In Obsidian, the *Check*, *Export*, *Recompute* and *Start over* commands read the note **from disk** and write a status file last (`noteSha256`, hash of the text read): see the [plugin README](../apps/obsidian-plugin/README.md), "Commands on the active note".

### The 11 anomaly types

| Type | Severity | Rule | Hint (`hint`) |
|---|---|---|---|
| `node_outside_zone` | error | a box extends, even partly, outside its parent's frame | enlarge the parent on the side concerned (`layout.nodes.<parent>.x/y/w/h`) |
| `zone_overlap` | error | two sibling containers (zones, groups) overlap | shift the second one, drawn on top, by the width or height of the overlap, with its pinned content |
| `text_overflow` | error | a box's text, as drawn (lines wrapped to the real width), extends outside the box by more than 0.5 px; for an icon shape with the icon above the text, also when the room left for the icon falls below its minimum | widen or enlarge the box by the missing number of pixels (`layout.nodes.<id>.w/h`) |
| `edge_crosses_node` | error | an arrow crosses a box or a group that is neither its source, nor its target, nor one of their ancestors; the background of a **zone** is not an obstacle | attach the arrow on the side of the source that faces the target (`sourcePort`), removing the pinned `points`; only if this changes the path (no hint when the arrow already has this attachment and no `points`) |
| `edge_crosses_zone_title` | warning | an arrow cuts through the title of a zone that neither the source nor the target belongs to | none |
| `label_overlaps_node` | warning | an arrow label covers a box or a group, **its source and target included** (a label longer than its arrow spills over its ends); zones and the arrow's containers (ancestors of the source or the target) do not count | `labelPos` 24 px from the middle of the path, on the first free side: no box (ends included), no zone or group title, no other label (no hint if no side is free) |
| `label_overlaps_zone_title` | warning | an arrow label covers the title of a zone or a group, **including** a zone or a group that contains the arrow | same |
| `label_overlaps_label` | warning | two arrow labels overlap | same, for the second label |
| `edge_detour` | warning | the path is more than 2.5 times the `direct` distance **and** more than 200 px longer than it | none |
| `edge_crossings` | info | two arrows cross (this is where the arc jumps are drawn) | none |
| `edge_geometry_approximate` | info | the path is not computed by core (see `exact` below) | for a `straight` or `curved` arrow between two elements that are not nested: `route: orthogonal` |

Details:

- Tolerances: for text (`text_overflow`) and arrow labels (`label_overlaps_node`, `label_overlaps_zone_title`, `label_overlaps_label`), an overflow or overlap of 0.5 px or less does not count; for `node_outside_zone` and `zone_overlap`, any overflow or overlap counts, even by one pixel.
- `direct` is the straight-line distance between the **centers** of the source and the target; `length` is the path length.
- **Hidden layers** appear in the geometry (`hidden: true`) but are not checked, except with `--all-layers` (`allLayers` in the API).
- Path checks (`edge_crosses_node`, `edge_crosses_zone_title`, `edge_detour`, `edge_crossings`) and label checks only cover arrows whose path is exact.
- The order of anomalies is stable: by type, then by identifiers.

### `exact` and `edge_geometry_approximate`

Each arrow in the report carries `exact`. It is true when the path (`points`) is computed by core (ELK or router) and drawn as is. It is false when the renderer draws it itself, so the report can only give an approximation: a loop from an element to itself, an arrow to its own container, a `straight` or `curved` route, a router failure. An inexact arrow is not examined by the other checks; it is reported once, by `edge_geometry_approximate` (info, which does not make the diagram "dirty").

### The report

JSON Schema: [`packages/core/schema/layout-report.schema.json`](../packages/core/schema/layout-report.schema.json) (`reportVersion: 1`).

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

- `valid: false`: the block does not pass validation; `issues` says why, `geometry` is `null`, and there are no anomalies.
- `geometry.nodes[]`: `kind` is `box`, `zone` or `group`; `parent`; the frame `x, y, w, h` (diagram coordinates); `label` (frame and lines of the text, `null` without text); `auto` (placed by the auto layout, not pinned); `hidden` if the layer is hidden.
- `anomalies[]`: `ids` (the faulty object first, then the reference object), `message`, `at` (point where it shows), optional `hint`. `hint.patch` maps a dotted path in the block to the proposed value (`null`: remove the key); **it is never applied automatically**, the agent or the person decides.
- `score`: numbers of errors, warnings and infos; `clean` is true when there is no error and no warning.
- `sourceHash`: hash of the block (see "Hashes"), to know whether a report still matches the text.

### `labelPos`

`layout.edges.<id>.labelPos` sets the **center** of an arrow label (absolute coordinates), not its corner. Without it, the label is centered on the middle of the path length. It is the target of the hints of `label_overlaps_node`, `label_overlaps_zone_title` and `label_overlaps_label`.

### Command line

```
trustdiagram lint <file.md|file.trustdiagram.yaml>… [--json] [--all-layers]
trustdiagram render <note.md> -o <folder> [--format svg|png|svg,png]
```

- `lint` accepts Markdown notes (all ` ```trustdiagram ` blocks, except those shown as examples inside another fence, ` ````markdown ` or `~~~`) and `.trustdiagram.yaml` files. In text mode, one line per anomaly (`file#block : severity type — message`), then a total. With `--json`, an object `{ "files": [{ "file", "fileIds": [identifier…], "diagrams": [report…] }] }`: the reports are those of the `lintLayout` API (as is, conforming to the schema); `fileIds[i]` is the file identifier (see "File names") of report `diagrams[i]`, the same as `file#block` in text mode and as `id` in the plugin status. A note without a block gives a "no diagram found" warning (on standard error) and exit code 0.
- Exit code: **0** no error; **1** at least one anomaly error or one invalid block (warnings and infos do not cause failure); **2** incorrect usage or file not found.
- `render` on a note writes one image per valid block in the folder (`svg` by default; `--format svg,png` for both); an invalid block is reported and the exit code is 1. The PNG is rendered at ×2 scale (twice the width and height of the SVG), like the plugin export.

### File names

Files written for a note (CLI and Obsidian plugin, same rules) are named `<note>.<block>.<extension>`:

- `<note>` (`exportBaseName`): the note path without `.md`, with `/` replaced by `__`. In the plugin, it is the path in the vault; in the CLI, the path **relative to the current folder** (whether typed as absolute or with `./`), or only the file name for a note outside the current folder: `trustdiagram render notes/a.md` and `trustdiagram render ./notes/a.md` both give `notes__a.<block>.svg`;
- `<block>` (`diagramFileIds`): the block `id`, or `bloc-<n>` (position of the block in the note, starting at 1) if it has none; a duplicate gets the suffix `-2`, `-3`…
- the characters `\ / : * ? " < > |` and control characters become `_`, and so does a segment made only of dots (`.`, `..`): a name always stays inside the output folder.

Example: the note `archi/paiements.md` with a block whose id is `payments` and a block without an id gives `archi__paiements.payments.svg` and `archi__paiements.bloc-2.svg`.

## Graphical editor

The editor (Obsidian plugin, then Confluence app) works like draw.io; it writes the YAML itself, with targeted rewriting (comments, alignment and unaffected entries stay identical; a list keeps its block or flow style):

- **What you see is what you save.** As soon as a gesture touches the geometry (move, resize, align, drop an element) or the structure (connect, reattach, delete, edit an arrow's text, rename a box so that its size changes), **all boxes are pinned** at their displayed position in `layout.nodes` (`x`, `y`; `w`, `h` for containers and resized boxes). Otherwise the boxes left free would be placed again at render time and the saved diagram would no longer look like the screen.
- **Arrows**: as long as you do not touch their path, they are rerouted live (same router as the render) and nothing is written for them. Dragging a segment or a waypoint pins the path (`layout.edges.<id>.points`, absolute coordinates).
- **Double-click** on a box → edits its visible text "name [technology]": the final text in brackets is the `tech` (removing it deletes it); a box without a set size adapts to the new text. On an arrow → edits all its visible text "label [protocol]": the final text in brackets is the `protocol` (removing it deletes the protocol).
- **Reattach** an arrow end to another box → changes `from` / `to`; its path becomes automatic again.
- **Connect**: when you hover over a box, an arrow appears on its right edge; dragging it to another box creates a relation with `id` = `r-<source>-<target>` (suffix `-2`, `-3`… if the id exists).
- **Shapes** (left column): clicking a shape adds it at the center of the view, dragging it drops it where you release it. The element type follows from the shape (cylinder → `datastore`, queue → `queue`, person → `person`, network → `node`, Kubernetes → `container`…); the shape is written to `layout` if it is not the type's default. `id` = `<type>-<n>` (first free `n`), `name: Nouvel élément`, added to the container under the point (otherwise to the root).
- **Delete** an element → removes the element, its children, the relations that touch them and their `layout` entries.
- **Text position**: 3 × 3 grid in the Text section of the Format panel → `style.align` (going back to the default position removes the key; the format painter copies it).
- **Text style**: Bold / Italic / Underline buttons and the size list in the Text section → `style.bold`, `italic`, `underline`, `size`; the object's font in Advanced mode → `style.font`. With nothing selected, **Font** chooses the diagram's base font → `layout.font`. Going back to the default value removes the key.
- **Format** (right panel) → `layout.nodes.<id>.style` / `layout.edges.<id>.style` with readable keys.
- **Attachment**: dragging an arrow end along the outline of the same box → `sourcePort` / `targetPort`; the rest of the path stays automatic.
- **Bring to front / Send to back / Bring forward / Send backward** → `layout.nodes.<id>.z`. **Arrange in a row / in a column**: side by side or one below the other, 40 px apart, centers aligned on the first box.
- **Layers** (panel below Format): add, rename, hide, lock, move up/down, delete (the elements go back to the default layer), move the selection to a layer → `layout.layers` and `layout.nodes.<id>.layer`.
- **Shape…** → `layout.nodes.<id>.shape`. **Reset position** → deletes the `layout.nodes.<id>` entry and applies the auto layout on screen right away.
- Going back to the initial state (Undo all the way) restores the original text exactly.
- **Copy / Paste / Duplicate** (Ctrl/Cmd+C, V, D): copies the selected boxes with the content of zones and the relations **between** copied elements (not those to the outside). Pasting creates new elements (`id` = `<type>-<n>`) with `name`, `tech`, `description`, `ref`, properties, classes, shape, size and formatting; it is offset by 20 px on each paste. In the same diagram, the copy goes into the same container (or the same layer) as the original; in another diagram, into the container under the copy. Pasted relations have an automatic path.
- **Format painter** (🖌): picks up the formatting of the selection; the next selected object receives it, **replacing** its formatting. Double-click the format painter: it stays active until Esc. From a box to an arrow (or the reverse), only the common settings carry over (outline, thickness, line, text color). Ctrl/Cmd+Shift+C / Ctrl/Cmd+Shift+V: copy / paste the style.
- **Arrows**: thickness through presets (1 to 4) or a free value (`style.width`); **Direction**: toward the target (default), both ways (`style.start: block`) or no arrowhead (`style.end: none`); **Reverse direction** swaps `from` and `to` in the model. Arrow bends are slightly rounded (6 px radius; `style.rounded: 0` for sharp corners).
- **Pan the view**: dragging on the background moves the whole diagram (hand cursor); Shift + drag draws the selection rectangle. The hand / arrow buttons of the view controls swap this choice (remembered).
- **Change zone**: dropping a box on a zone (or a group) puts it in that zone (the target zone is highlighted); dropping it on empty space outside its zone moves it to the top level, in the layer of its original zone. In the YAML, the element (with its children and its comments) is moved into the `children` of the new zone or into `model.elements`; relations and positions do not change.
- Not allowed with the mouse in V1: duplicating with Ctrl+drag.

## Hashes

For synchronization, each block has three SHA-256 hashes computed on **canonical JSON** (sorted keys, no spaces). They therefore ignore comments, indentation, flow/block style and line endings:

- `model`: the whole block except `layout`;
- `layout`: the `layout` section;
- `block`: a combination of the two.

## Three-way merge

When a block has been changed on both sides (vault and Confluence) since a common `base` version:

- **Model**: changed on one side only → that side wins; changed identically on both sides → nothing to do; changed differently → **conflict**, reported per element (`/model/elements/<id>`, at any depth), per relation (`/model/relations/<id>`) or per field (`/title`, `/description`…).
- **Layout**, per entry then per field: changed on one side only → that side wins; changed identically on both sides → nothing to do; same field changed differently → **conflict** `{ key, field }`; deleted on one side and changed on the other → conflict `{ key, field: "*" }`.
- In case of conflict, the proposed text keeps the local value (`ours`) and applies everything that can be merged.
- The starting text is the one from the side whose model wins: its comments and formatting are kept.

## Annotated example

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
    - id: zone-si                # a zone: hosting perimeter
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
    zone-si: { x: 40, y: 40, w: 600, h: 400 }    # pinned
    gw: { x: 320, y: 140, w: 160, h: 60, shape: aws.api-gateway }
    # app and logs have no entry: placed automatically
  edges:
    r1: { route: orthogonal, points: [[400, 260]] }
```

Examples: [`examples/`](../examples) (fully placed CI/CD pipeline, online shop and lending system context with auto layout, photo sharing application in top-to-bottom tiers, display rules). Diagrams with mixed layout (pinned zones, content placed automatically) are in the test fixtures (`packages/core/test/fixtures/`).

## LLM validation

Check of 2026-11-10: a fresh agent (lightweight model, no other context) received **only** [`packages/core/test/fixtures/place-zones.trustdiagram.yaml`](../packages/core/test/fixtures/place-zones.trustdiagram.yaml) (a RAG document assistant, turned into a test fixture) and 4 questions.

| Question | Result |
|---|---|
| Outgoing flows from the "Orchestrateurs" zone to other zones | ✅ 3 exact flows (`f-recherche`, `f-generation`, `f-traces`), with target zone, label and protocol; internal and incoming flows correctly excluded |
| Protocol between the orchestrator and the LLM proxy | ✅ "API compatible OpenAI" via `f-generation` |
| Logging component | ✅ `journal` (Datadog), fed by `orchestrateur` and `llm-proxy` |
| Human validation? | ✅ "no", justified by the diagram `description` |

Points the agent flagged as ambiguous, and the fix applied:
- `mode` missing on the main calls → specified in the example (`sync`) and the "missing = not specified" rule added above;
- content of the generation flow not stated → `data` field added on `f-recherche` and `f-generation`, and recommended in the specification.
