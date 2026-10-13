# Markion v0.4.10

Backup and Sync now refreshes only when something actually changes and no longer stalls when a file sits outside the approved sync scope, and Visual Edit gains an option that keeps inline Markdown syntax hidden while you edit.

## Highlights

### Backup and Sync: event-driven, quieter background work

- Git details refresh when something actually changes — workspace file changes, window activation, saving, completed Git operations, workspace switches, and opening Backup and Sync — instead of a fixed five-second poll. A slow fallback poll runs only while the window is active, and bursts of triggers coalesce into a single refresh.
- Refreshes spawn far fewer Git processes: repository discovery uses one process, capability checks run once per repository instead of inside every status read, nested-repository detection reuses Git's own status output instead of walking the whole worktree, and ignore/tracked checks for changed paths and attachment references are batched into one invocation per refresh.
- Unchanged files are no longer re-hashed on every refresh — content fingerprints are cached by file size and modification time on disk, and by document version for dirty buffers — and a refresh captures the text of dirty documents only. Background Git activity, battery drain, and Sync Now latency drop accordingly.

### Backup and Sync: out-of-scope files no longer block syncing

- A file outside the approved sync scope — a new PDF attachment, `.DS_Store`, `Thumbs.db`, anything under a hidden folder — is now held back and listed as "not included in sync" while the rest of your notes save, commit, and upload normally. Previously a single such file stopped Sync Now completely.
- Each held-back new file offers three actions: **Include once** (commits exactly this content on the next sync; void if the file changes first), **Always sync .ext files** (adds the extension to your notes policy), and **Don't sync on this device** (a device-local exclusion written under a Markion marker in the repository's `info/exclude`, touching no repository state other devices can see). Settings show the extra file types and device exclusions, and can remove them.
- Well-known operating-system artifacts — `.DS_Store`, `._*`, `Thumbs.db`, `desktop.ini`, Office/LibreOffice lock files, and similar — are ignored by default: never proposed as new notes and never shown as exceptions.
- A tracked deletion whose content exactly matches a held-back new file is held back with it, so a move can never propagate as a deletion alone.
- The status stays truthful: while held-back items exist you always see a separate "N files are not included in sync" notice with a Review entry, never folded into a plain "synchronized" message. Held-back tracked changes pause applying incoming updates (which need a clean tracked state) but never block committing or uploading your other notes.

### Visual Edit: optional stable inline editing (off by default)

- New preference **Keep inline syntax hidden** in the Preferences panel. With it on, moving the caret or selecting no longer splices inline Markdown or HTML source into the line — line breaks stay put while editing, even on dense lines such as a README's language row of `<a href>` labels ([#20](https://github.com/willmove/markion/issues/20)).
- Non-visible source is edited through surfaces that do not affect the layout: Markdown links and simple HTML anchors open the existing link editor from a pointer activation or the Insert Link command; inline math opens a floating formula editor from a click on the rendered formula; escapes, decoded entities, emoji shortcodes, `<br>`, and other inline-HTML elements open a floating editor for the exact authored token from an explicit edit action.
- Bold, italic, strikethrough, inline code, highlight, superscript, and subscript stay styled and remain editable in place; adding or removing those markers stays on the existing format commands.
- The preference defaults to off, so the existing progressive-reveal behavior is unchanged, and toggling it never modifies document text.

## Compatibility

- No migration is required for Markdown files, preferences, or workspace data.
- The sync policy file gains two optional fields (extra allowed file types and include-once approvals). Older Markion builds ignore them but drop them when they rewrite the policy, so switching back and forth can lose those settings. Device-local exclusions live in the repository's own `info/exclude` file and survive.
- Windows installers remain unsigned, so SmartScreen still requires **More info → Run anyway**. macOS DMGs remain ad-hoc signed (no Developer ID signing or notarization), so Gatekeeper still warns on first launch and is dismissed via *System Settings → Privacy & Security → Open Anyway*.

## Downloads

- Windows x64: `markion_0.4.10_x64-setup.exe` (NSIS) and `markion_0.4.10_x64-portable.zip`.
- macOS Apple Silicon: `Markion_0.4.10_aarch64.dmg`.
- macOS Intel: `Markion_0.4.10_x64.dmg`.
- Linux x86_64: `markion_0.4.10_amd64.deb`, `markion_0.4.10_x86_64.rpm`, and `markion_0.4.10_x86_64.AppImage`.

## Verification

- `cargo test --workspace` passed locally (2050 passed, 0 failed, 5 ignored), and the packaged MarkNice workspace bundle verified (23 files).
- Windows, macOS arm64, macOS x86_64 (cross-compiled), and Linux native CI packaging builds passed, including the AppImage smoke test on a host with no Vulkan driver.
- The updater signature, `update.json`, and the Aliyun OSS mirror were verified after publication.

**Full comparison**: https://github.com/willmove/markion/compare/v0.4.9...v0.4.10

---

# Markion v0.4.10（中文说明）

备份与同步改为事件驱动、后台安静得多，同步范围之外的文件不再阻塞同步；Visual Edit 新增可选的“编辑时保持隐藏行内语法”模式。

## 主要更新

### 备份与同步：事件驱动的后台刷新

- Git 详情只在真正发生变化时刷新——工作区文件变动、窗口激活、保存、Git 操作完成、切换工作区、打开备份与同步面板——取代固定 5 秒轮询。仅在窗口处于活动状态时运行较慢的兜底轮询，突发触发会合并为一次刷新。
- 每次刷新产生的 Git 进程大幅减少：仓库发现只用一个进程；能力检测每个仓库只做一次，不再出现在每次状态读取中；嵌套仓库检测直接复用 Git 自身的状态输出，不再遍历整个工作区；变更路径与附件引用的忽略/跟踪检查按次合并为一处批量调用。
- 未变更的文件不再在每次刷新时重新计算内容指纹——磁盘文件按大小与修改时间缓存，脏缓冲按文档版本缓存——刷新只复制脏文档的文本。后台 Git 活动、耗电与 Sync Now 延迟随之下降。

### 备份与同步：范围外文件不再阻塞同步

- 同步范围之外的文件——新 PDF 附件、`.DS_Store`、`Thumbs.db`、隐藏文件夹里的任何东西——现在会被扣留并列为“未纳入同步”，其余笔记照常保存、提交并上传。此前单个此类文件会让 Sync Now 完全停止。
- 每个被扣留的新文件提供三种处理：**仅包含这一次**（下次同步提交该内容；文件先变更则失效）、**始终同步 .ext 文件**（把扩展名加入笔记策略）、**此设备不同步**（写入仓库 `info/exclude` 中带 Markion 标记的设备本地排除，不产生其他设备可见的仓库变更）。设置界面会显示额外文件类型与设备排除项，并可移除。
- 常见操作系统产物——`.DS_Store`、`._*`、`Thumbs.db`、`desktop.ini`、Office/LibreOffice 锁文件等——默认忽略：不会作为新笔记提示，也不会显示为异常。
- 被跟踪删除的文件若与某个被扣留的新文件内容完全一致，会与其一并扣留，移动绝不会以单独的删除传播。
- 状态显示保持真实：存在被扣留项时，始终单独显示“N 个文件未纳入同步”及“审阅”入口，绝不并入“已同步”；被扣留的已跟踪变更只会暂停应用传入更新（需要干净的跟踪状态），不会阻塞其他笔记的提交与上传。

### Visual Edit：可选的稳定内联编辑（默认关闭）

- 偏好设置新增**编辑时保持隐藏行内语法**。开启后，移动光标或选择文本不再把内联 Markdown/HTML 源码拼接进文本行——编辑时断行保持稳定，即使是 README 语言链接行那种密集的 `<a href>` 标签也不受影响（[#20](https://github.com/willmove/markion/issues/20)）。
- 不可见的源码通过不影响排版的方式编辑：Markdown 链接与简单 HTML 锚点通过指针点击或“插入链接”命令打开现有链接编辑器；行内公式点击渲染后的公式打开浮动公式编辑器；转义、解码实体、emoji 短代码、`<br>` 及其他行内 HTML 元素通过显式编辑操作打开针对原始记号的浮动编辑器。
- 粗体、斜体、删除线、行内代码、高亮、上标、下标保持样式并可原地编辑；增删这些标记仍使用现有格式命令。
- 该偏好默认关闭，既有渐进显示行为不变；切换开关绝不会改动文档内容。

## 兼容性

- Markdown 文件、偏好设置与工作区数据无需迁移。
- 同步策略文件新增两个可选字段（额外允许的文件类型与“仅包含这一次”批准）。旧版 Markion 会忽略它们，但在重写策略文件时会将其丢弃，来回切换版本可能丢失这些设置。设备本地排除保存在仓库自身的 `info/exclude` 文件中，不受影响。
- Windows 安装包仍未签名，首次启动仍需 SmartScreen 的**更多信息 → 仍要运行**。macOS DMG 仍为 ad-hoc 签名（无 Developer ID 签名与公证），首次启动会触发 Gatekeeper 警告，可通过*系统设置 → 隐私与安全性 → 仍要打开*放行。

## 下载

- Windows x64：`markion_0.4.10_x64-setup.exe`（NSIS）与 `markion_0.4.10_x64-portable.zip`。
- macOS Apple Silicon：`Markion_0.4.10_aarch64.dmg`。
- macOS Intel：`Markion_0.4.10_x64.dmg`。
- Linux x86_64：`markion_0.4.10_amd64.deb`、`markion_0.4.10_x86_64.rpm` 与 `markion_0.4.10_x86_64.AppImage`。

## 验证

- 本地 `cargo test --workspace` 通过（2050 通过、0 失败、5 忽略），打包的 MarkNice 工作区校验通过（23 个文件）。
- Windows、macOS arm64、macOS x86_64（交叉编译）与 Linux 原生 CI 打包构建全部通过，包括在无 Vulkan 驱动主机上的 AppImage 冒烟测试。
- 更新器签名、`update.json` 与阿里云 OSS 镜像已在发布后验证。

**完整变更对比**: https://github.com/willmove/markion/compare/v0.4.9...v0.4.10
