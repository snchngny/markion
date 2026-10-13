## Why

Visual Edit hides Markdown and inline-HTML markers until the caret enters them, then splices that source back into the line. On a dense line — the README language row of `<a href="…">` labels is the reported case — the inserted syntax is much wider than the visible text, so the rest of the line wraps until the caret leaves. [Issue #20](https://github.com/willmove/markion/issues/20) asks for an option that keeps those markers hidden while editing.

## What Changes

- Add a Visual Edit preference, default off, that keeps inline syntax out of the text flow. With the preference off, caret entry still reveals the smallest proven syntax group, exactly as today.
- With the preference on, caret movement and selection do not splice inline reveal groups into the projection. Visible prose, styled runs, link labels, and rendered atoms keep the unfocused layout, so line breaks do not change merely because the caret entered a construct.
- Edit destinations and other non-visible source through surfaces that do not participate in line breaking:
  - Markdown links and simple HTML `<a href>…</a>` anchors open the existing link editor from the Insert Link command or a pointer activation on the rendered link. Arrowing through a link does not open it and does not rewrite an HTML anchor into Markdown.
  - Inline math opens a floating payload editor from a pointer activation on the rendered formula. The formula stays rendered in the line.
  - Escapes, decoded entities, emoji shortcodes, `<br>`, and other supported inline-HTML elements that are not simple anchors open a floating editor for the exact authored token, from an explicit edit action. They stay rendered in the line.
- Emphasis, strong, strikethrough, inline code, highlight, superscript, and subscript stay styled. Their visible characters remain editable through the existing display-to-source mapping. Adding or removing those markers stays on the existing format commands.
- Toggling the preference, and opening or dismissing an editor without applying it, does not change document text or version.
- Persist the preference in `config.toml`, default it off when missing or invalid, and include it in preferences reset. Labels and status text go through `src/i18n.rs`.

**Non-goals**: a second rich-text document model; changing the default progressive-reveal behavior; reserving width for hidden markers; turning a whole block into a source island; rewriting HTML links into Markdown; changing Source, Split, or Read mode; changing in-flow editing for code fences, block math, diagrams, tables, front matter, HTML blocks, or the image source toggle; changing when heading prefixes and blockquote markers reveal.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `markdown-editing`: Progressive inline reveal becomes preference-conditional. The preference-on path keeps inline groups rendered and edits non-visible source through off-flow editors. Heading-prefix and blockquote-marker reveal stay as they are.
- `theme-preferences`: The Preferences panel gains the stable-inline-editing toggle, applied immediately and persisted with a safe default of off.
- `ui-i18n`: The new preference label, status feedback, and off-flow editor chrome are localized in every supported language.

## Impact

- Presentation only. `build_visual_projection` already decides which reveal groups become source pieces from the caret; the preference is another input to that decision. Parsed `VisualBlock` caches stay keyed by document version and are not invalidated by the preference or by cursor-only movement. Syntax highlighting and the per-version text handle are unchanged.
- `AppPreferences` and `src/storage/preferences.rs` gain one boolean. The Preferences panel in `src/app/root_view.rs` gains one display-setting control.
- `src/inline_edit.rs` link lookup today parses Markdown links only. Simple HTML anchors need a source-preserving edit path into the existing link editor in `src/app/editing.rs`.
- Inline math, escapes, entities, emoji, `<br>`, and non-anchor inline HTML need a floating editor that commits one exact source replacement.
- Existing reveal tests keep the default-off behavior. New tests cover the preference-on projection, the explicit editors, and preference persistence.
- `docs/visual-editing-quality.md` must mention the preference, because the visual-editing documentation contract test tracks that guide.
