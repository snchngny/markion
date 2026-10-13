## MODIFIED Requirements

### Requirement: Source-backed Visual Edit mode
The editor SHALL provide a Visual Edit mode whose default presentation contract is WYSIWYG (what you see is what you get): every Markdown construct SHALL be presented as close to its rendered result as the editor can edit through an exact, lossless source mutation. `MarkdownDocument.text` SHALL remain the single canonical editable representation — Visual Edit is a presentation and editing contract over that text, not a parallel rendered document model. Every Visual Edit mutation SHALL flow through the existing source-mutation path (dirty-state, undo/redo, autosave, recovery, per-tab isolation), and SHALL NOT edit an inferred rendered tree. Constructs that the editor currently cannot present in rendered form are classified as **WYSIWYG coverage gaps** under the `WYSIWYG coverage roadmap` requirement, not as accepted end states; each gap SHALL show raw source only as a transitional measure until a future change closes it. While Keep inline syntax hidden is off, math SHALL be rendered while unfocused and SHALL reveal its complete authored delimiter group when focused. While Keep inline syntax hidden is on, inline math SHALL stay rendered in the line and its payload SHALL be edited only through the off-flow editor in `Stable visual inline editing`; display and fenced math SHALL keep the payload editor beside the rendered formula. Math SHALL NOT be mutated through an inferred rendered formula tree.

#### Scenario: Visual prose editing updates Markdown source
- **WHEN** the user edits visible prose inside a paragraph, heading, blockquote, or list item in Visual Edit mode
- **THEN** the corresponding Markdown source text is updated
- **AND** the document dirty flag and undo history are updated through the existing document mutation path

#### Scenario: Visual formatting actions remain source-backed
- **WHEN** the user applies bold, italic, inline code, link, image, heading, list, task list, blockquote, or fenced-code formatting in Visual Edit mode
- **THEN** the editor updates the underlying Markdown markers in `MarkdownDocument.text`
- **AND** switching to Source mode shows Markdown source that represents the visual result

#### Scenario: Focused syntax can be exposed for editing
- **WHEN** Keep inline syntax hidden is off and the cursor enters visually formatted inline content whose hidden Markdown syntax is needed for precise editing
- **THEN** the editor SHALL reveal the smallest complete source syntax group for that focused content (progressive-reveal WYSIWYG)
- **AND** the construct SHALL NOT be mutated through an ambiguous rendered-tree edit

#### Scenario: Unfocused math is rendered in Visual Edit
- **WHEN** valid inline, display, or fenced math is visible in Visual Edit and neither its source range nor delimiter group is focused
- **THEN** inline math appears as a baseline-aligned formula atom and display math appears as a typeset block
- **AND** the authored Markdown remains the canonical content

#### Scenario: Focused inline math reveals one complete source group
- **WHEN** Keep inline syntax hidden is off and the caret or a selection endpoint enters an inline math source range in Visual Edit
- **THEN** the complete byte-exact delimiter group is revealed as one editable source range
- **AND** unrelated prose in the same block remains rendered

#### Scenario: Focused display math uses a source edit island
- **WHEN** the user focuses `$$...$$` or fenced `math` content in Visual Edit
- **THEN** that formula presents an editable payload containing its exact authored syntax alongside the rendered formula
- **AND** moving focus away restores formula rendering without changing the document version

#### Scenario: Complex constructs use conservative edit islands
- **WHEN** the user focuses a construct that the `WYSIWYG coverage roadmap` classifies as an open gap (for example a front-matter region, an indented code block, an unclosed code fence, or a paragraph containing decoded HTML entities)
- **THEN** the editor SHALL show the authored source as a transitional editing affordance and SHALL classify the construct against the roadmap
- **AND** the construct SHALL NOT be mutated through an ambiguous rendered-tree edit
- **AND** the gap SHALL be tracked for closure by a future change that moves the construct into rendered or progressive-reveal WYSIWYG

#### Scenario: Visual-only interaction does not reparse unnecessarily
- **WHEN** the user moves the cursor, changes selection, hovers text, or focuses a rendered editor or transitional source view without changing document text
- **THEN** the document version SHALL remain unchanged
- **AND** derived Markdown caches SHALL NOT be invalidated

### Requirement: Progressive Markdown marker reveal in Visual Edit
Visual Edit SHALL keep supported paragraph, heading, list-item, and blockquote content visually rendered while it is focused. When precise editing requires Markdown syntax and Keep inline syntax hidden is off, the editor SHALL reveal only the smallest complete inline syntax group whose source mapping is proven exact, while `MarkdownDocument.text` remains the canonical representation. While Keep inline syntax hidden is on, inline reveal groups SHALL stay out of the text flow as specified by `Stable visual inline editing`. Structural prefixes of headings and list items SHALL be revealed when the caret or a selection endpoint is inside the prefix range or at the prefix end, and SHALL be revealed for the whole prefix when a heading or list item with no visible content runs owns the caret, whether or not Keep inline syntax hidden is on. Display-to-source and source-to-display mappings SHALL remain UTF-8-safe and monotonic for pointer placement, selection, keyboard navigation, platform text input, and IME caret geometry. Syntax whose mapping is nested, overlapping, byte-inexact, or otherwise ambiguous MUST use a conservative source-backed edit island. Inline caret-reveal scenarios elsewhere in this capability apply only while Keep inline syntax hidden is off; `Stable visual inline editing` governs the preference-on path.

#### Scenario: Focusing plain prose preserves visual rendering
- **WHEN** the user places the caret in plain text inside a supported visual paragraph, heading, list item, or blockquote
- **THEN** the block remains in its rendered visual style
- **AND** the entire block is not replaced by raw Markdown source

#### Scenario: Active inline syntax is revealed locally
- **WHEN** Keep inline syntax hidden is off and the caret enters exactly mapped strong, emphasis, strikethrough, or inline-code content in a supported visual block
- **THEN** the complete markers for that active inline construct are revealed together with its content
- **AND** other supported content in the same block remains visually rendered

#### Scenario: Active link exposes its destination
- **WHEN** Keep inline syntax hidden is off and the caret enters an exactly mapped inline link label or its hidden source syntax
- **THEN** the local link syntax, including its destination and optional title, becomes visible and editable
- **AND** editing it mutates the corresponding canonical Markdown source range

#### Scenario: Leaving a reveal group hides its markers without mutation
- **WHEN** the caret or selection endpoints leave a locally revealed syntax group without editing document text
- **THEN** that group returns to its rendered representation
- **AND** the document version, dirty state, undo history, and derived Markdown caches remain unchanged

#### Scenario: Selection remains source-accurate across hidden markers
- **WHEN** a Visual Edit selection crosses rendered runs separated by hidden Markdown markers
- **THEN** the visual highlight represents the selected canonical source content across projected segments
- **AND** replacement, copy, cut, and formatting actions operate on the exact source selection

#### Scenario: Keyboard navigation into a hidden marker reveals it
- **WHEN** Keep inline syntax hidden is off and source-based keyboard navigation moves the caret into a currently hidden marker range
- **THEN** the next Visual Edit render reveals the owning syntax group
- **AND** subsequent caret geometry and input use an identity-mapped visible source position

#### Scenario: Caret at heading prefix end reveals the marker
- **WHEN** the Visual Edit caret is at the end of an ATX heading prefix (the first title position, including on an empty heading)
- **THEN** the heading prefix is revealed together with any visible title text
- **AND** the heading is not replaced by a whole-block source island

#### Scenario: Title-interior caret still hides the heading prefix
- **WHEN** the Visual Edit caret is inside the visible title of a non-empty ATX heading and not inside the prefix range
- **THEN** the heading prefix remains hidden
- **AND** the heading stays in heading typography

#### Scenario: Ambiguous inline syntax remains conservative
- **WHEN** an inline construct is nested, overlapping, escaped, transformed, or otherwise lacks a proven byte-exact mapping
- **THEN** Visual Edit uses a source-backed edit island for the affected block or construct
- **AND** it does not guess a rendered-tree mutation

### Requirement: Links SHALL have an exact source-backed visual editor
Creating an exactly mapped inline link SHALL provide a visual editor for label, URL, and optional title. While Keep inline syntax hidden is off, focusing that link SHALL reveal its source in the line. While Keep inline syntax hidden is on, caret focus SHALL keep the link rendered, and the same editor SHALL open from the Insert Link command or the explicit edit affordance, including for a simple HTML anchor as specified by `Stable visual inline editing`. Submitting the editor SHALL serialize one valid inline Markdown link for a Markdown link, or preserve the HTML anchor form for a simple HTML anchor, and apply it as one source mutation. Canceling or changing focus SHALL leave the authored source byte-for-byte unchanged. Reference-style, malformed, or crossing links SHALL retain conservative source editing.

#### Scenario: Selected text creates a link
- **WHEN** the user selects exactly mapped visual prose, opens the link editor, enters a URL and optional title, and confirms
- **THEN** the selected text becomes the link label in one canonical-source mutation
- **AND** the resulting selection and source ranges remain UTF-8 safe

#### Scenario: Existing inline link is edited
- **WHEN** the caret is within an exactly mapped inline link and the user changes its URL or title
- **THEN** the complete link source is replaced once while preserving its visible label unless edited
- **AND** one Undo restores the complete prior link and selection

#### Scenario: Link edit is canceled
- **WHEN** the visual link editor is dismissed without confirmation
- **THEN** document text, version, dirty state, selection, undo history, and derived cache identity remain unchanged

### Requirement: Maintained Visual Edit support classification
The repository SHALL maintain a current Visual Edit WYSIWYG coverage matrix that classifies every user-visible Markdown construct into exactly one of three classes: **rendered WYSIWYG** (the construct is shown in its rendered form, including dedicated field/payload editors for code, math, diagrams, images, YAML front matter, HTML blocks, and tables whose editors ARE the rendered form), **progressive-reveal WYSIWYG** (the construct is rendered by default and reveals its smallest complete source syntax group when the caret enters it — inline formatting, links, inline math, structural prefixes), or **WYSIWYG coverage gap** (the construct currently shows raw source and is tracked under the `WYSIWYG coverage roadmap` for closure by a future change). The matrix SHALL name the canonical editable range and the verification evidence for each rendered/reveal class, and SHALL name the roadmap priority and implementation seam for each gap. The matrix SHALL agree with the stable requirements and the implemented `VisualBlock`/`VisualBlockEditor` behavior. Empty ATX headings and empty list items SHALL be classified as rendered WYSIWYG with progressive-reveal structural prefixes, not as coverage gaps. The matrix SHALL record that Keep inline syntax hidden, when on, keeps inline progressive-reveal constructs rendered and edits their non-visible source through the off-flow editors in `Stable visual inline editing`, while structural prefixes stay progressive-reveal.

#### Scenario: Contributor evaluates current WYSIWYG coverage
- **WHEN** a contributor reads the Visual Edit WYSIWYG coverage matrix
- **THEN** it distinguishes rendered WYSIWYG constructs (prose, headings including empty ATX headings, code including indented code blocks and unclosed or malformed fences, math including Pending/Error payload editors, diagrams, images including reference-style block images, tables including ragged grids, YAML front matter, lists and task lists including empty list items and clickable Visual Edit checkboxes, footnote definitions and references, definition lists, blockquotes, alerts, rules, HTML blocks), progressive-reveal WYSIWYG constructs (inline formatting, links, inline math, escaped punctuation, decoded HTML entities, supported inline HTML, structural prefixes, heading attributes), and open WYSIWYG gaps (multiline or otherwise unprovable images, residual unsupported gaps)
- **AND** it explains that canonical Markdown remains the single persisted representation and that no construct is edited through a parallel rendered tree
- **AND** it explains that Keep inline syntax hidden leaves inline progressive-reveal constructs rendered and moves their source editing off the line

#### Scenario: A new visual block behavior is proposed
- **WHEN** a proposal changes how a Markdown construct is presented or edited in Visual Edit
- **THEN** the proposal selects one of the three coverage classes for the construct
- **AND** if the proposal moves a construct out of the gap class, it updates the matrix and the `WYSIWYG coverage roadmap`
- **AND** implementation and documentation cannot be considered complete until the matrix and invariant evidence are updated

## ADDED Requirements

### Requirement: Stable visual inline editing
Visual Edit SHALL offer Keep inline syntax hidden, defaulting to off. While it is off, caret and selection entry SHALL continue to splice the smallest proven inline syntax group into the line. While it is on, caret movement, selection, and an IME marked range SHALL NOT splice a non-empty inline reveal group into the paragraph flow. Visible prose, styled runs, link labels, and rendered atoms SHALL keep the same line breaks they have when unfocused. `MarkdownDocument.text` SHALL remain the only editable representation. Toggling the preference, or opening and dismissing an off-flow editor without confirming it, SHALL NOT change document text, document version, dirty state, undo history, or derived Markdown caches.

An inline group whose visible text is empty and that contains the caret or the IME marked range SHALL still be revealed, so a freshly auto-paired delimiter remains visible until it has visible content.

While the preference is on, non-visible source SHALL be edited only through surfaces that do not participate in line breaking:

- A Markdown link, a reference-style link, an angle-bracket autolink, or a simple HTML anchor (`<a>` with only `href` and an optional `title`, text content, and no nested tags) SHALL open the link editor from the Insert Link command or from an explicit edit affordance drawn outside the text flow. Arrowing through the link or clicking its label SHALL place the caret and SHALL NOT open the editor. Clicking the existing navigation icon SHALL still open the resolved destination. Confirming a Markdown link SHALL write one Markdown link. Confirming a simple HTML anchor SHALL write an HTML anchor of the same shape and SHALL NOT rewrite it as Markdown.
- Inline math SHALL stay a rendered atom. A pointer click on that atom SHALL open a floating payload editor for the delimiter-free formula. Confirming SHALL replace the payload inside the existing delimiters as one source edit.
- An escape, a decoded entity, an emoji shortcode, a `<br>`, or any other supported inline-HTML element that is not a simple anchor SHALL stay rendered. A pointer click on that atom SHALL open a floating editor containing the exact authored token. Confirming SHALL replace that exact source range. Backspace or Delete on an escape, entity, emoji shortcode, or `<br>` atom SHALL remove the whole authored token in one source edit.
- Emphasis, strong, strikethrough, inline code, highlight, superscript, and subscript SHALL stay styled. Their visible characters SHALL remain editable through the display-to-source mapping. Adding or removing their markers SHALL stay on the existing format commands.
- Inline images SHALL NOT splice their tag or `![...](...)` source into the line on caret entry. The existing image source toggle SHALL remain the control that shows that source.

Heading prefixes, blockquote markers, callout title lines, TOC tokens, code fences, display and fenced math, diagrams, tables, front matter, and HTML blocks SHALL keep their existing reveal and editor behavior while the preference is on. Source, Split, and Read modes SHALL be unchanged.

#### Scenario: Preference off still reveals the active inline group
- **WHEN** Keep inline syntax hidden is off and the caret enters `**bold**` in a Visual Edit paragraph
- **THEN** the complete `**bold**` source is shown in the line
- **AND** the rest of the paragraph stays rendered

#### Scenario: Preference on keeps a styled run from changing line breaks
- **WHEN** Keep inline syntax hidden is on and the caret enters strong, emphasis, strikethrough, or inline-code content
- **THEN** the markers stay hidden and the run keeps its visual style
- **AND** the paragraph's line breaks match the unfocused rendering
- **AND** document version and derived Markdown caches stay unchanged

#### Scenario: Preference on keeps an HTML anchor from wrapping the line
- **WHEN** Keep inline syntax hidden is on and the caret enters the label of an inline `<a href="…">` on a line of several such anchors
- **THEN** the label stays rendered without the surrounding tag or destination in the line
- **AND** the other anchors on that line stay on the same line

#### Scenario: Arrowing through a link does not open the editor
- **WHEN** Keep inline syntax hidden is on and keyboard navigation moves the caret through a rendered link label
- **THEN** the link editor stays closed
- **AND** the link syntax stays out of the line

#### Scenario: Explicit edit opens the link editor for a simple HTML anchor
- **WHEN** Keep inline syntax hidden is on and the user activates the edit affordance on `<a href="guide.md" title="Guide">Guide</a>`
- **THEN** the link editor shows the label `Guide`, the URL `guide.md`, and the title `Guide`
- **AND** confirming without changes leaves the authored HTML bytes unchanged

#### Scenario: Confirming a simple HTML anchor preserves HTML
- **WHEN** the user changes that anchor's URL in the link editor and confirms
- **THEN** the source range is replaced by an HTML anchor with the new URL, the same label, and the same title
- **AND** the replacement is not a Markdown `[label](url)` link
- **AND** one Undo restores the previous anchor bytes

#### Scenario: A nested HTML anchor uses the exact-token editor
- **WHEN** Keep inline syntax hidden is on and the user activates editing on `<a href="https://example.com"><img src="b.png" alt="B"></a>`
- **THEN** the editor presents the complete authored tag slice
- **AND** confirming a changed slice replaces only that range
- **AND** canceling leaves the slice byte-for-byte unchanged

#### Scenario: Clicking inline math opens the payload editor off the line
- **WHEN** Keep inline syntax hidden is on and the user clicks a rendered inline formula
- **THEN** the formula remains rendered in the paragraph
- **AND** a floating editor shows the delimiter-free payload
- **AND** confirming replaces the payload inside the existing `$` delimiters as one source edit

#### Scenario: Empty auto-pair stays visible
- **WHEN** Keep inline syntax hidden is on and Markdown auto-pair inserts an empty `**` pair with the caret between the markers
- **THEN** those markers are visible until the group has visible text
- **AND** after the user types a visible character, the markers leave the line and the character stays styled

#### Scenario: Deleting a decoded entity removes the whole token
- **WHEN** Keep inline syntax hidden is on and the caret is on a rendered `&amp;` atom and the user presses Backspace or Delete
- **THEN** the complete authored `&amp;` token is removed in one source edit
- **AND** surrounding prose stays rendered

#### Scenario: IME inside styled text does not splice markers
- **WHEN** Keep inline syntax hidden is on and an IME composition updates a marked range inside non-empty strong text
- **THEN** the strong markers stay out of the line
- **AND** the marked range is painted on the rendered text
- **AND** committing the composition is one undoable source edit

#### Scenario: Heading prefix reveal is unchanged
- **WHEN** Keep inline syntax hidden is on and the caret is at the end of an ATX heading prefix
- **THEN** the heading prefix is still revealed with the title
- **AND** a caret inside the title, outside the prefix, still hides the prefix

#### Scenario: Image source toggle still shows image source
- **WHEN** Keep inline syntax hidden is on and the caret enters an inline image
- **THEN** the image source is not spliced into the surrounding prose
- **AND** the existing image source toggle can still show the authored image span
