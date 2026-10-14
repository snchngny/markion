# Markion v0.4.11

Visual Edit now renders a single blank line between blocks as paragraph spacing instead of an extra empty row, and the floating link/token editors become real text fields you can select in, with links opening directly on Ctrl/Cmd-click.

## Highlights

### Visual Edit: one blank line is now paragraph spacing ([#19](https://github.com/willmove/markion/issues/19))

- A single newline between two blocks — for example between two paragraphs, or a paragraph and a heading — collapses into the same paragraph spacing you see in Read mode, so the document no longer looks like it carries one extra blank row per separator ([#19](https://github.com/willmove/markion/issues/19)).
- Authored spacing still shows: the second and further consecutive blank lines keep their full body-line height, and blank runs at the start or end of the document render as ordinary empty rows.
- The separator stays fully interactive: clicking it places the caret, Up/Down still stop on it, and typing on it grows a real line and pushes the following content down.
- The empty insertion row at the end of the document no longer reserves a phantom line, and the typewriter-mode caret still centers correctly while you type at the document end.

### Visual Edit: selectable overlay editors and Ctrl/Cmd-click links

- The floating editors for links, HTML anchors, and inline tokens (math, escapes, entities, and similar hidden syntax) now support a real caret and text selection — click, drag, and double-click work the way they do in an ordinary text field instead of replacing the whole draft on typing.
- Clicking anywhere outside an overlay editor now closes it, so stray overlays no longer linger over the document.
- Ctrl/Cmd-click on a Markdown link, an inline HTML link, a footnote reference, or an in-document heading anchor opens or jumps to its destination directly. The per-line navigation icons these constructs used to paint (the link open icon, the footnote jump icon, and the `#` anchor icon) are gone, so prose lines stay clean.

## Compatibility

- No migration is required for Markdown files, preferences, or workspace data. Documents keep their newlines exactly as authored; only Visual Edit geometry changes.
- Typing into a collapsed separator intentionally moves the following content down, because a spacing-only gap no longer reserves a body line in advance.
- Platform notes are unchanged: the Windows installers are unsigned and may require a SmartScreen bypass, and the ad-hoc-signed macOS DMGs still show a Gatekeeper warning on first launch that you dismiss via *System Settings → Privacy & Security → Open Anyway* (or the documented `xattr -cr` / `curl -LO` alternatives).

## Downloads

- Windows x64: NSIS installer and portable `.zip`.
- macOS Apple Silicon: DMG.
- Linux x86_64: DEB, RPM, and AppImage.

## Verification

- Local workspace test suite passes on all crates (`cargo test --workspace`), and the packaged MarkNice workspace bundle verification succeeds.
- Native CI builds and packaging pass on Windows, macOS (Apple Silicon and x86_64 cross-compiled), and Linux, including the no-GPU AppImage smoke test; the updater installers are signed and mirrored with a version-matched `update.json`.

**Full comparison**: https://github.com/willmove/markion/compare/v0.4.10...v0.4.11

---

# Markion v0.4.11（中文说明）

Visual Edit 中块与块之间的单个空行现在渲染为段落间距而不再多占一行，链接与语法浮动编辑器成为可选择文本的真正输入框，链接可通过 Ctrl/Cmd-click 直接打开。

## 主要更新

### Visual Edit：单个空行改为段落间距（[#19](https://github.com/willmove/markion/issues/19)）

- 两个块之间的单个换行——例如两个段落之间、段落与标题之间——折叠为与阅读模式一致的段落间距，文档不再看起来每个分隔处多出一个空行（[#19](https://github.com/willmove/markion/issues/19)）。
- 手动留白仍然可见：连续空行从第二行起保持完整行高，文档开头或结尾的空行照常渲染为空白行。
- 分隔符完全可交互：点击可放置光标，Up/Down 仍会在其上停靠，在上面输入会生长出真实文本行并将后续内容下移。
- 文档末尾的空插入行不再预留幻影行，打字机模式下在文档末尾输入时光标仍能正确居中。

### Visual Edit：可选择的浮动编辑器与 Ctrl/Cmd-click 链接

- 链接、HTML 锚点以及行内语法（数学公式、转义、实体等隐藏语法）的浮动编辑器现在支持真实光标与文本选择——点击、拖选、双击与普通文本框一致，不再是一输入就整体替换草稿。
- 点击编辑器外部即可关闭，浮动编辑器不会再残留在文档上。
- 在 Markdown 链接、行内 HTML 链接、脚注引用或文档内标题锚点上 Ctrl/Cmd-click 可直接打开或跳转到目标。这些语法此前画在行内的导航图标（链接打开图标、脚注跳转图标、`#` 锚点图标）全部移除，正文行保持干净。

## 兼容性

- Markdown 文件、偏好设置、工作区数据均无需迁移。文档保留原始换行，仅 Visual Edit 几何布局变化。
- 在折叠的分隔符上输入会刻意将后续内容下移，因为纯间距空隙不再预先保留一整行。
- 平台说明不变：Windows 安装包未签名，可能需要绕过 SmartScreen；macOS DMG 为 ad-hoc 签名，首次启动仍会触发 Gatekeeper 警告，可通过“系统设置 → 隐私与安全性 → 仍要打开”放行（或使用文档中的 `xattr -cr` / `curl -LO` 替代方式）。

## 下载

- Windows x64：NSIS 安装程序与便携 `.zip`。
- macOS Apple Silicon：DMG。
- Linux x86_64：DEB、RPM 与 AppImage。

## 验证

- 本地全部 crate 的 workspace 测试套件通过（`cargo test --workspace`），打包的 MarkNice 工作区 bundle 校验成功。
- Windows、macOS（Apple Silicon 与交叉编译的 x86_64）及 Linux 原生 CI 构建与打包通过，包括无 GPU 的 AppImage 冒烟测试；更新器安装包已完成签名，并与版本匹配的 `update.json` 一同镜像分发。

**完整变更对比**: https://github.com/willmove/markion/compare/v0.4.10...v0.4.11
