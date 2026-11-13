# Licences des composants et contenus tiers

| Composant | Usage dans TrustDiagram | Licence |
|---|---|---|
| [maxGraph](https://github.com/maxGraph/maxGraph) (`@maxgraph/core`) | Rendu et (phase 3) édition des schémas | Apache-2.0 |
| [Lucide](https://lucide.dev) (`lucide`) | Toutes les icônes de l'interface (barre, cartel, calques, boutons Éditer) et les 40 icônes des règles d'affichage | ISC |
| [elkjs](https://github.com/kieler/elkjs) | Placement automatique | EPL-2.0 |
| [svgpath](https://github.com/fontello/svgpath) | Conversion des arcs des icônes de règle en courbes (rendu identique sur tous les canevas) | MIT |
| [yaml](https://github.com/eemeli/yaml) | Lecture et réécriture du YAML | ISC |
| [Ajv](https://github.com/ajv-validator/ajv) | Validation par JSON Schema | MIT |
| [@noble/hashes](https://github.com/paulmillr/noble-hashes) | Empreintes SHA-256 | MIT |
| [Arimo](https://fonts.google.com/specimen/Arimo) (`@fontsource/arimo`) | Police embarquée dans les SVG, métriques de mesure | SIL OFL 1.1 |
| [Tinos](https://fonts.google.com/specimen/Tinos) (`@fontsource/tinos`) et [Cousine](https://fonts.google.com/specimen/Cousine) (`@fontsource/cousine`) | Polices de base serif et mono (embarquées, métriques de mesure) | SIL OFL 1.1 |
| [Kubernetes Icons Set](https://github.com/kubernetes/community/tree/main/icons) | Famille de formes `k8s.*` (39 icônes, `packages/shapes/vendor/k8s`) | Apache-2.0 (au choix avec CC-BY-4.0) — attribution : The Kubernetes Authors ; Kubernetes est une marque de The Linux Foundation |
| Playwright, Vite, Vitest, tsx, opentype.js | Outillage de développement et CLI (non embarqués dans les schémas) | Apache-2.0 / MIT |

## Contenus volontairement exclus

- **draw.io (jgraph/drawio)** : aucune icône, aucun stencil, aucune forme (`stencils`, `Shapes.js`, `shapes/*.js`, `img/lib/*`) n'est repris, converti ou retracé. Leur [licence](https://github.com/jgraph/drawio/blob/dev/src/main/webapp/stencils/LICENSE) interdit leur usage, même sous forme dérivée, dans les produits Atlassian ou l'écosystème Marketplace — or TrustDiagram comporte une app Confluence (Forge). Toutes les formes `general`, `c4`, `uml` et `network` sont dessinées de zéro.
- **Icônes AWS, Azure, GCP** : non incluses en V1 (décision du 2026-11-10) ; les conditions des fournisseurs autorisent les schémas d'architecture mais pas clairement la redistribution dans un outil.
