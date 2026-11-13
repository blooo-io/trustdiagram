# TrustDiagram: Obsidian plugin

Displays the ` ```trustdiagram ` blocks of a note as architecture diagrams, and lets you edit them with the mouse.
The format is described in [`docs/format.md`](../../docs/format.md) (French version: [`docs/format.fr.md`](../../docs/format.fr.md)).

The interface follows the Obsidian language: French if Obsidian is in French, English otherwise.

## Installation

From the Obsidian community plugin directory (submission in progress): *Settings → Community plugins → Browse* → **TrustDiagram**.

By hand, from a [GitHub release](https://github.com/blooo-io/trustdiagram/releases): copy `main.js`, `manifest.json` and `styles.css` into `<your vault>/.obsidian/plugins/trustdiagram/`, then enable the plugin.

## Usage

- **Display**: every ` ```trustdiagram ` block is rendered as a diagram (Reading view and Live Preview). An invalid block shows the error and its line, with the text of the block.
- **Edit**: hover over the diagram → **Edit** button → full-screen editor.
  - Move, resize, rectangle selection, waypoints on arrows (drag the middle of an arrow).
  - **Connect**: hover over a box, a small arrow appears on its right edge; drag it to another box. Dragging the box itself moves it.
  - **Format** on the right (with a selection): fill, outline color and thickness, solid / dashed / dotted line, text color; for an arrow, start and end arrowheads.
  - **Attachments**: drag an arrow end along the edge of its box to choose where it attaches. By default, arrows spread themselves along the sides, without overlapping.
  - **Rectangle selection**: drag on the background.
  - **Arrange in a row / in a column**, **Bring to front / Send to back / Bring forward / Send backward** (toolbar).
  - **Layers** (below the Format panel): 👁 hide, 🔒 lock, ⇲ move the selection into it, ▲▼ reorder, double-click to rename.
  - Arrows **jump in an arc** where they cross.
  - **Shapes** on the left: click a shape to add it, or drag it onto the diagram (search at the top of the list).
  - Toolbar: Undo / Redo, Align (left, center, right, top, middle, bottom), Distribute, Shape… (change the shape of the selection), Delete, Reset position (back to auto layout), Fit.
  - **Save** rewrites only the lines of the block in the note.
- **Commands** (command palette, Ctrl/Cmd+P): *TrustDiagram: Insert diagram*, *TrustDiagram: Pin the layout of the diagram under the cursor*, and the four commands on the active note described below.

| Shortcut | Action |
|---|---|
| Ctrl/Cmd+S | Save |
| Ctrl/Cmd+Z / Ctrl/Cmd+Shift+Z | Undo / Redo |
| Delete | Delete the selection |
| Ctrl/Cmd+C / V / D | Copy / Paste (also into another diagram) / Duplicate |
| Ctrl/Cmd+Shift+C / Ctrl/Cmd+Shift+V | Copy / paste the style (format painter 🖌 in the toolbar) |
| Drag on the background / Shift + drag | Pan the diagram (hand) / selection rectangle |
| Esc | Stop the format painter, otherwise close without saving |

## The editor interface

- **Top bar**: title and identifier of the block, **Simple / Advanced** switch (remembered), undo / redo, *Save* (icon; a red dot signals unsaved changes; the editor stays open after saving), *Close* (× icon).
- **Shapes** (left column): search, families; clicking adds a shape, dragging it drops it on the diagram.
- **The diagram** in the center; at the bottom, the selection action bar (Copy, Paste, Duplicate, Format painter, and in Advanced mode Align, stacking order, Reset position); at the top right, the zoom.
- **The Format panel** (right column): name and kind of the selection, its **properties** (own values and values inherited from its classes, "Applied rules"), shape (Advanced), fill, outline, text, arrowheads; with nothing selected, the diagram summary and the basic gestures, and in Advanced mode the diagram's **classes** and **display rules**. The **layers** appear at the bottom in Advanced mode.
- The interface is the same in Obsidian and Confluence; it does not follow the dark theme. The diagram keeps its own theme (Ardoise, Encre or Brique).

## What the editor changes

- **Double-click** on a box: edit "name [technology]"; on an arrow: "label [protocol]". The text in brackets at the end is the technology or the protocol; erasing it deletes it.
- **Reattach** an arrow: drag its end to another box.
- **Drag an arrow segment**: its path is pinned as is; otherwise arrows reroute themselves when boxes move.
- **What you see is what you save**: as soon as you move or resize a box, all boxes are pinned at their displayed position.
- The alignment buttons need at least 2 selected boxes (3 to distribute): Shift+click or rectangle selection. Buttons that do not apply are grayed out, with a tooltip that says why.
- **Properties**: name / value on a box or an arrow (`true`, `false` and numbers are recognized); a value inherited from a class can be overridden on the object, ↺ restores it. A text can display a value: `Passerelle {{version}}`.
- **Display rules** (Advanced mode, nothing selected): *if* a property equals…, *then* fill, outline, thickness, line, text color, or an icon at one of the 9 positions. A rule wins over formatting set by hand; the Format panel shows "Set by the rule …".
- The description and the type of an element are still edited in the text.
- Closing the editor (Esc, click outside, *Close*) with unsaved changes asks for confirmation; *Save* without changes does not touch the note.
- The diagram is found in the note by its content, including inside a list or a callout.
- If the note was changed during editing (another tab, sync), saving is refused: *"The block was changed during editing: reload before saving."*

## Commands on the active note (check and export)

Four commands process **all the blocks** of the active note, without asking anything (so they can be driven remotely). They appear in the command palette only if the active note is a Markdown file.

| Identifier | Name | Effect |
|---|---|---|
| `trustdiagram:export` | Export the note's diagrams (SVG, PNG) | writes, for each valid block, an SVG (embedded fonts) and a PNG (×2); an invalid block gets no image (old ones are deleted) but gets its report |
| `trustdiagram:lint` | Check the layout of the note's diagrams | writes the layout report of each block: the same as `trustdiagram lint --json` (see [`docs/format.md`](../../docs/format.md), "Layout check") |
| `trustdiagram:relayout` | Recompute the layout of the note's diagrams | places and pins everything that is still automatic, then rewrites the blocks in the note |
| `trustdiagram:relayout-reset` | Start over: recompute the whole layout of the note's diagrams | first removes all the pinned layout (`layout.nodes`, `layout.edges`), then recomputes and pins |

- **Output folder**: *Output folder* setting (*Settings → TrustDiagram*), `_trustdiagram` by default, created if needed. It is relative to the vault; an empty path or one with a `.` or `..` segment is replaced by the default (`exports.v2` is accepted, `exports/../x` is not).
- **File names**: `<folder>/<note>.<block>.<extension>` (`svg`, `png`, `layout-report.json`) and `<folder>/<note>.status.json`. `<note>` is the note path without `.md`, with `/` becoming `__`; `<block>` is the block `id`, or `bloc-<n>` without an id, with `-2`, `-3`… for a duplicate; characters not allowed in a file name become `_`. The rules are the same as the CLI's (`docs/format.md`, "File names").
- **Rewriting**: *Recompute* and *Start over* change only `layout`; the model and the comments do not move, an invalid block is left as is, and they write no file other than the status. If the note is open in a tab, the change goes through the editor (**Ctrl/Cmd+Z undoes it**), then the note is **saved to disk before the status is written**. If the tab is closed during the computation, the rewrite goes through the file.
- **Reading: always the file on disk.** The commands read the note as it is on disk (what an agent just wrote, what Obsidian will show after reloading), even if it is open: *Check* and *Export* ignore unsaved typing or a tab not yet reloaded. *Recompute* and *Start over* write to the note: if it is open and the tab differs from the file (unsaved changes, or note changed outside Obsidian and not yet reloaded), they **refuse** without touching anything (`ok: false`, message in `errors`); just try again a little later.
- **One at a time**: commands started one after the other run one after the other, in order. A notice summarizes each run.

### The status file

`<folder>/<note>.status.json` is written **last**, after all the other files of the command: when it exists, the work is done.

```json
{
  "command": "lint",
  "note": "archi/paiements.md",
  "noteSha256": "3a7bd3e2360a3d29eea436fcfb7e44c735d117c42d1c1835420b6b9942dd4f1b",
  "startedAt": "2026-11-12T09:30:00.000Z",
  "finishedAt": "2026-11-12T09:30:01.200Z",
  "ok": true,
  "diagrams": [
    {
      "id": "payments",
      "files": ["_trustdiagram/archi__paiements.payments.layout-report.json"],
      "score": { "error": 0, "warning": 1, "info": 2, "clean": false },
      "valid": true
    }
  ],
  "errors": []
}
```

- `command`: `export`, `lint`, `relayout` or `relayout-reset`; `note`: path of the note in the vault.
- `noteSha256`: SHA-256 (hexadecimal, text encoded as UTF-8) of the note text **read by the command** (for *Recompute*, before rewriting); `null` if the note was not read (refusal, error). It lets you check that the command saw the latest change.
- `diagrams[]`: one item per block, in note order; `id` is the file identifier of the block; `files` lists the paths (relative to the vault) written for it; `score` is given only by *Check*; `valid` is false for a block that does not pass validation.
- **`ok` and `valid` are distinct**: `ok` is false only on an unexpected error (recorded in `errors`); an invalid block leaves `ok` true and only sets `valid` to false. `valid` only reports validation: if the export of a valid block then fails (render, PNG conversion, writing), its SVG and PNG are both removed (never a new image next to a stale one), it keeps `valid: true` with `files: []`, and the error is in `errors` (`ok: false`).

### An agent's loop

1. Open the note in Obsidian (it must be the active note).
2. Write the note to disk if needed, and compute the SHA-256 (hexadecimal) of this exact text, encoded as UTF-8.
3. Delete the old `<note>.status.json`.
4. Run the command (for example `trustdiagram:lint`, through the Obsidian command or the command palette).
5. Wait for the new `<note>.status.json` to appear, then read it. **If the JSON does not parse** (file read while being written), wait a moment and read it again.
6. Compare `noteSha256` with the hash from step 2: if they **differ**, the command read another text (write not finished yet, another change): restart at step 3. With `ok: false` (for example *Recompute* refused because the tab is not reloaded yet): wait a little and restart at step 3.
7. Read the listed files (reports, images). After *Recompute*, the note is already saved to disk: read it again before changing it again.
8. Fix the note (following the anomalies' `hint`), then start again at step 2 until `score.clean`.

## V1 limits

- Desktop only (`isDesktopOnly`); mobile is not tested yet.
- Moving a box does not yet move it to another zone (planned with the properties panel).
- No AWS / Azure / GCP cloud icons (see `docs/LICENSES.md`).
- No network access; publishing to Confluence comes in phase 5.
