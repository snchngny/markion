## Context

See proposal.md for why this change exists. Inline reveal is already a projection decision, not a second document.

```
MarkdownDocument.text
        │  versioned parse, cached per version
        ▼
VisualBlock { editable_runs, reveal_groups, block_prefix, quote markers }
        │  caret / selection / marked range          preference (interaction state)
        ▼                                            │
build_visual_projection  ◄──────────────────────────┘
        │
        ▼
painted line  +  optional floating editor (link / math payload / raw token)
```

`build_visual_projection` in `src/visual.rs` turns a `reveal_group` into an in-flow source piece when the caret, selection, or IME marked range intersects it. Heading prefixes and blockquote markers are applied after that, from `block_prefix` and `quote_context`, not from `VisualRevealKind`. Parsed blocks stay on the document version. The preference must join the projection inputs and must not bump `MarkdownDocument.version()` or drop the `Arc` block cache.

The link editor in `src/app/editing.rs` already edits label, URL, and title, then writes one Markdown link. `inline_link_at` only parses Markdown links. The README language row is HTML `<a href>`, which today reveals through `VisualRevealKind::InlineHtml`.

## Goals / Non-Goals

**Goals:**

- One persisted boolean gates whether inline `reveal_groups` are spliced into the line.
- Off-flow editors commit one exact source replacement and otherwise leave the document untouched.
- Default-off projection stays covered by the existing reveal tests.

**Non-Goals:**

- See proposal.md. This design also does not add a global shortcut, change link-navigation icon behavior, or retarget heading-prefix and quote-marker reveal.

## Decisions

### 1. Preference flag on the projection, default off

Add `stable_visual_inline: bool` to `AppPreferences`, the TOML file, the in-memory app state, the preferences snapshot, and `reset_preferences`. Missing or non-boolean values deserialize to `false`, matching `show_hidden_files`.

`build_visual_projection_with_marked_range` gains a `reveal_inline: bool` (true when the preference is off). When `reveal_inline` is false, skip every `block.reveal_groups` entry except an empty visible group that currently contains the caret or the marked range. Still apply `block_prefix` and quote-marker ranges exactly as today.

The empty-group exception keeps Markdown auto-pair usable: a just-inserted `**` or `` ` ` `` with the caret between the delimiters and no visible characters stays on screen. As soon as the group has visible text, the markers leave the line again.

Toggling the preference rebuilds the projection on the next paint. It does not reparse.

Alternative considered: a second visual-block cache per preference. Rejected because reveal is already derived from caret state on each paint, and a second cache would duplicate the versioned parse.

Alternative considered: filtering reveal groups out of the cached block. Rejected because the cache is shared per document version and the preference is per app, not per version.

### 2. Do not auto-open an editor on caret motion

Clicking a rendered link label places the caret and edits visible label bytes through the existing hidden-marker mapping. Arrow keys do the same. The Insert Link command still opens the link editor.

While the preference is on, each rendered link also shows a compact edit icon in the same non-flow chrome as the existing navigation icon. The navigation icon still opens the URL. The edit icon opens the link editor. While the preference is off, that edit icon is absent and in-flow reveal remains the editing surface.

Inline math, escapes, entities, emoji shortcodes, `<br>`, and non-anchor inline HTML do not have a label to type into. A pointer click on that rendered atom opens the floating editor. The caret stays on the atom's source boundary. Backspace or Delete on an escape, entity, emoji, or `<br>` atom removes the whole authored token in one source edit.

Alternative considered: open the link editor whenever the caret enters a link. Rejected because arrowing along the language row would open and close the editor on every link, which blocks editing the visible label.

### 3. Reuse the link editor for simple HTML anchors

A simple anchor is one `<a>` whose only attributes are `href` and an optional `title`, with text content and no nested tags. The link editor fills label, URL, and title from that tag. Confirming replaces only that source range with an HTML anchor of the same shape. It does not emit `[label](url)`.

Any other inline HTML element, including an anchor that wraps an image or carries other attributes, uses the raw-token editor: one field containing the exact authored slice, committed as a byte-range replacement. Cancel leaves the slice unchanged.

Markdown links keep the current serializer.

### 4. One floating raw editor for math and opaque tokens

Inline math uses the same floating surface with a monospaced payload field. Confirm replaces the complete delimiter group (`$...$`) and leaves the delimiters in place around the edited payload. Display math, fenced math, code fences, and the image `</>` toggle are not routed here.

Escapes, entities, emoji shortcodes, `<br>`, and non-simple inline HTML use that surface with the complete authored token. Confirm replaces that exact range. The projection is not involved in the edit.

Opening, typing in, or canceling the floating editor does not change document version. Confirm goes through the existing source-mutation path.

### 5. IME stays on the rendered projection

While the preference is on, an IME marked range inside a non-empty inline group does not splice that group into the line. The marked range paints on the rendered segments. The empty-group exception still applies, so composition inside a freshly auto-paired empty delimiter can reveal that pair. Composition remains one undo group.

### 6. Images

Caret-driven reveal of an inline `<img>` or `![...](...)` is suppressed with the other reveal groups. The existing collapsible source toggle stays the way to show that source, whether or not the preference is on. If a Markdown image splices its source on focus through a path other than `reveal_groups`, suppress only that caret-driven splice and leave `visual_source_expanded` alone.

## Risks / Trade-offs

- [Hidden markers make an empty auto-pair invisible] → Reveal only the empty group that contains the caret.
- [HTML anchor rewrite drops attributes] → Only the simple `href`/`title` form uses the structured editor; everything else is a raw exact-range edit.
- [IME geometry depends on revealed source today] → Keep the marked range on rendered segments and add a preference-on composition test before changing the reveal filter.
- [Clicking a math atom both places a caret and opens an editor] → Click opens the editor and parks the caret on the atom boundary; it does not splice `$...$` into the paragraph.
- [Existing tests assert reveal on caret] → They run with the preference off. Add preference-on cases beside them rather than rewriting the default.

## Migration Plan

No document migration. Existing `config.toml` files omit the key and therefore keep today's reveal behavior. Reset restores `false`. Rolling back the change is a matter of ignoring the key; already-written `true` values sit unused.

## Open Questions

None. The preference default, which constructs stay in-flow, and when the floating editors open are fixed by the delta specs.
