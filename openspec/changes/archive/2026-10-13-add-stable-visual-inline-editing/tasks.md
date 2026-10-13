## 1. Persist the preference

- [x] 1.1 Add `stable_visual_inline` to `AppPreferences` and `src/storage/preferences.rs`, defaulting to false. Missing and non-boolean values load as false. Verify with a round-trip test and an invalid-value test that the editor still starts.
- [x] 1.2 Copy the field through app startup, the preferences snapshot, and `reset_preferences`. Verify a reset test returns the preference to off without changing the open document's text or version.

## 2. Gate inline reveal in the projection

- [x] 2.1 Add `reveal_inline` to `build_visual_projection_with_marked_range`. When it is false, skip non-empty `reveal_groups` and still reveal `block_prefix` and quote-marker ranges. Pass `reveal_inline: true` from existing callers. Verify a `src/visual.rs` test that `**bold**`, a Markdown link, and `<a href>` stay rendered, while an ATX prefix at the caret still reveals. Verify the existing caret-reveal tests still pass.
- [x] 2.2 While `reveal_inline` is false, still reveal an inline group whose visible text is empty and that contains the caret or the IME marked range. Verify a test that an empty `**` pair with the caret between the markers is visible, and that the markers hide once a visible character is inside the group.
- [x] 2.3 Thread `stable_visual_inline` from app state into the Visual Edit projection in `src/app/preview.rs` as `reveal_inline = !stable_visual_inline`. Verify an app test that toggling the preference does not increment `MarkdownDocument.version()` and keeps the same cached visual-block `Arc`.

## 3. Preferences control and localization

- [x] 3.1 Add the Keep inline syntax hidden toggle to the non-theme display settings, apply it on the next render, and persist it. Add the toggle label, on/off status text, link edit-affordance label, inline-math editor chrome, and exact-token editor chrome to `src/i18n.rs` for every supported language. Verify the i18n exhaustive match compiles and a preferences-panel test shows the toggle reflecting the current value.

## 4. Edit links off the line

- [x] 4.1 While the preference is on, paint an edit icon in the existing non-flow link chrome. Clicking the label places the caret and leaves the editor closed. Clicking the navigation icon still opens the destination. Arrowing through the label does not open the editor. Verify an app test on a line of `<a href>` labels: the projection text contains the labels and not the tags, and the following label stays in the same projected line.
- [x] 4.2 Extend link-target lookup so a simple `<a href title?>text</a>` fills the existing link editor and confirms back to an HTML anchor, not a Markdown link. A nested or otherwise non-simple anchor opens the exact-token editor instead. Verify tests for unchanged confirm, URL replacement plus one undo, and cancel leaving the bytes untouched.

## 5. Edit math and opaque tokens off the line

- [x] 5.1 A pointer click on a rendered inline formula, while the preference is on, opens a floating payload editor and leaves the formula in the line. Confirm replaces the payload inside the existing delimiters as one source edit. Display and fenced math stay on their current payload editor. Verify an app test for confirm and for cancel leaving version unchanged.
- [x] 5.2 The same floating surface edits the exact authored token for an escape, a decoded entity, an emoji shortcode, a `<br>`, and a non-anchor inline HTML element. Backspace or Delete on an escape, entity, emoji, or `<br>` atom removes the whole token in one source edit. Verify a `&amp;` backspace test and a token-editor cancel test.
- [x] 5.3 While the preference is on, an IME marked range inside non-empty strong text paints on the rendered segments and does not splice `**` into the line. Verify a projection test that the marked range stays on the rendered text and that committing the composition is one undo group.

## 6. Images and the coverage guide

- [x] 6.1 While the preference is on, caret entry does not splice an inline `<img>` tag or `![...](...)` span into the surrounding prose. The existing image source toggle still shows that span. Verify a test that focus alone leaves the image rendered, and expanding the toggle still shows the authored span.
- [x] 6.2 Update `docs/visual-editing-quality.md` so the coverage matrix records the preference-on path for inline progressive-reveal constructs and states that structural prefixes stay progressive-reveal. Verify `visual_edit_documentation_contract_tracks_current_strategies` passes.

## 7. Regression

- [x] 7.1 Run the root-package tests that cover visual projection, link editing, preferences parsing, and the documentation contract. Verify the default-off reveal cases still pass and the new preference-on cases pass with them.
