# TrustDiagram for Confluence — Privacy policy

_Last updated: 13 November 2026_

TrustDiagram for Confluence ("the app") is a free app published by Blooo ("Blooo", "we"). This policy explains what the app does with data when it is installed on an Atlassian Confluence Cloud site.

## In short

- **We collect nothing.** The app sends no data to Blooo or to any third party. Blooo operates no server for this app.
- **The app runs entirely on Atlassian.** It is a Forge app with a Custom UI only: no backend function, no remote endpoint, no outbound network access (no `egress` permission in its manifest).
- **The app stores nothing on Atlassian's side.** It does not use the Forge storage API, entity properties or content properties.
- **It writes only the diagram you edit.** When you save, it rewrites the one code block that holds the diagram and leaves the rest of the page untouched.

## What the app accesses, and why

| Data | Access | Purpose |
|---|---|---|
| Content of the current Confluence page (Atlassian Document Format) | Read (`read:page:confluence`) | Find the TrustDiagram code block the macro points to and draw it |
| The same page | Write (`write:page:confluence`), only when a user clicks **Save** in the editor | Replace the edited code block with the new version, as a new page version created by that user |
| Page version and draft status | Read | Refuse to overwrite a newer version or an unpublished draft |
| Confluence language (locale) of the signed-in user | Read, from the Forge context | Show the interface in French or English |

All calls go through the Confluence REST API on behalf of the signed-in user (`requestConfluence` from `@forge/bridge`). The app can therefore never see or change more than that user can.

## Data stored in the browser

The editor remembers two interface preferences in the user's browser (`localStorage`, inside the app's sandboxed frame): the editor mode (Simple or Advanced) and the interaction mode. They contain no personal data and never leave the browser. Clearing the browser's site data removes them.

## Personal data

The app does not collect, store, log or transfer personal data. Diagrams may contain whatever text page authors write in them (for example a person's name); that text stays in the Confluence page, under the site's own Atlassian data controls, like any other page content.

Because the app processes no personal data on Blooo's side, Blooo is neither a controller nor a processor of personal data for this app. Atlassian processes Confluence data under its own terms and [privacy policy](https://www.atlassian.com/legal/privacy-policy).

## Third parties and subprocessors

None. The app loads no script, font, image or analytics from outside the app package. Its fonts (Arimo, Tinos, Cousine — SIL Open Font License) are bundled with the app.

## Data residency

The app stores no data, so it has no data of its own to pin to a region. Page content stays where the Confluence site stores it.

## Uninstalling

Uninstalling the app leaves the code blocks in your pages as plain, readable YAML. Nothing else needs to be deleted.

## Changes and contact

Changes to this policy are published in the [project repository](https://github.com/blooo-io/trustdiagram/blob/main/docs/marketplace/privacy.md), with their history.

Questions: open an issue on [GitHub](https://github.com/blooo-io/trustdiagram/issues) or write to **dev@blooo.io**. Security reports: **security@blooo.io**.

To have personal data in a diagram removed, ask an administrator of your Confluence site: the data lives in your pages, which only your site can edit. Blooo holds no copy.
