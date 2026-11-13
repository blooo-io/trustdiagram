# TrustDiagram for Confluence (Forge app)

A **TrustDiagram** macro displays a TrustDiagram code block (YAML) of the page live, and edits it with the same interface as the Obsidian plugin. Saving rewrites only that block in the page: the rest of the content and the other macros do not move.

The interface follows the Confluence language: French if Confluence is in French, English otherwise.

## Use in a page

1. Paste the diagram YAML into a **code block** of the page (any language, `yaml` recommended), then publish the page. To hide it, put this block in an **Expand** section ("Diagram source"): the macro still finds it, and it stays readable by the API and by LLMs.
2. Insert the **TrustDiagram — architecture diagram** macro. Its settings list the TrustDiagram blocks of the page: choose the one to display.
3. On the published page, **Edit diagram** (top left of the diagram) opens the full-screen editor. **Save** creates a new version of the page where only the block has changed; the editor stays open.
4. Properties, classes and display rules are edited as in Obsidian (see `docs/format.md`); the diagram on the page is recolored according to the rules.

Limits and safeguards:

- **Page being edited** (Confluence editor open): no Edit button. Publish the page first.
- **Unpublished draft**: saving is refused so as not to overwrite it. Publish or discard the draft.
- **Block changed by someone else** since it was opened: a screen offers two choices.
  - *Merge*: the layout from both sides is merged. If the elements or relations changed on both sides, the merge is refused.
  - *Reload*: your changes are discarded.
- **Duplicate identifier** on the page: editing is refused until one of the two is renamed.
