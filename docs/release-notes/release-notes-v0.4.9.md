# Markion v0.4.9

Adds an optional, configurable AI agent — conversation, writing actions, and reviewed workspace operations, off by default — and makes the Linux AppImage start on machines that have no Vulkan driver by carrying a bundled software Vulkan fallback.

## Highlights

### AI agent (new, disabled by default)

- AI is off until you enable it. Open **Preferences → AI Agent**, or press **Ctrl+Shift+A** (Windows/Linux) / **Cmd+Shift+A** (macOS). You supply your own service and key; there is no bundled subscription or account sign-in.
- Presets cover OpenAI, Anthropic, DeepSeek, Gemini, xAI, Mistral, Moonshot, Qwen, Zhipu, OpenRouter, Local/Ollama, and Custom. OpenAI Responses, OpenAI-compatible Chat Completions, and Anthropic Messages are supported, and local or custom servers can be keyless. Cloud presets fill an editable starter model ID.
- API keys go to the operating system's credential store, with an explicit session-only alternative. Keys never enter preferences, conversation history, logs, or AI request content.
- **Test connection** verifies the complete draft with synthetic text and a harmless tool probe, and **Find models** can list candidates before you commit to a model ID; **Save and enable** persists the tested draft before AI turns on. A failed save keeps the draft and leaves AI off.
- The conversation panel is resizable, with a fixed header and composer, streaming Markdown answers, follow-up questions, stop and retry, removable context attachments (selection, document, or an explicitly chosen file), and source references drawn only from real attachments and tool reads. Closing the panel leaves a running indicator; reopen it to stop or review. Optional local history is capped at 20 conversations / 8 MiB and excludes raw attachments, tool payloads, keys, grants, and approvals.
- Twelve writing actions — Draft, Continue, Polish, Correct grammar, Shorten, Expand, Change tone, Translate, Summarize, Outline, Extract tasks, and Custom instruction. Generation never edits the document: review the captured target with Before/After text, then Copy, Replace, Insert, or create a New unsaved note. Applied writing is a single undo group separate from earlier typing, and changed or closed targets are never redirected to another tab.
- Workspace mode adds bounded listing, filename/text search, and exact range reads, plus proposals for text edits, new notes, folders, and individual moves or renames. You grant reads for one folder inside the open workspace, review every operation with its before/after text and dependencies, and can revoke the grant at any time. Grants belong to the session and are never restored from history.
- The agent has no shell, no external OS or browser automation, no Git mutation, no publishing, and no permanent delete. It rejects stale document and disk results, refuses to move dirty files without Save First, respects Git write admission, and validates filesystem containment. File changes are recorded in a durable journal with a **Restore file actions** surface that stays available even when AI is off.
- Every AI string is localized in all seven interface languages (English, Simplified and Traditional Chinese, Japanese, French, German, Spanish), and the AI panel action is a rebindable shortcut.

### AI setup refinements

- Guided basic setup: the base URL and protocol appear in basic setup for Local and Custom presets, protocol is a dropdown, and validation names the failing field instead of showing a generic error.
- Test connection and model discovery work before the master switch is on; a successful test on an unconfigured installation offers a one-click **Enable AI**.
- Limits are edited as individually labeled numeric fields. The factory defaults are raised — 512 KiB context (was 64 KiB), 512 KiB output, and 8192 output tokens — and profiles still on the previous factory tuple are upgraded on load, while any customized profile is left untouched.
- Byte limits are edited in whole kibibytes with an approximate token hint and a **Restore recommended limits** control; the output-token and idle-timeout ceilings are raised so long-form and reasoning-model replies are no longer capped.
- Draft-loss protection: Markion warns before switching provider preset, profile, or tab when the draft has unsaved changes, and writing guidance saves independently of profile commits.
- Dedicated status messages for save, test, discover, forget, and stop replace reused button labels, settings and panel feedback are separated, and an enabled-but-misconfigured profile shows what is missing with an action that jumps to the offending field.
- Profile chips, conversation tabs, attachment chips, and source links are keyboard-focusable and activatable.
- The preferences tab strip now stays on one non-wrapping row and scrolls horizontally when the panel is narrow, instead of pushing a sixth tab onto a second row.

### Linux AppImage: bundled software Vulkan fallback

- The AppImage now starts on hosts with no usable Vulkan driver — VMs, cloud desktops, and minimal installs — where it previously exited within a second with a `NoSupportedDeviceFound` panic. The bundle carries Mesa's lavapipe software driver and the Khronos Vulkan loader under `usr/lib/markion-vulkan/`.
- Hardware Vulkan always stays first: Markion only restarts itself once onto the bundled driver after renderer initialization fails, and machines with a working GPU driver never load it. `MARKION_SOFTWARE_VULKAN=1` forces the bundled driver; `MARKION_SOFTWARE_VULKAN=0` disables the fallback.
- When no renderer can start at all, Markion now logs the error, prints a localized explanation with remediation steps, shows it in a native dialog when `zenity`, `kdialog`, or `xmessage` is available, and exits non-zero instead of a bare panic.
- The bundled stack adds roughly 20 MB compressed to the AppImage. The `.deb` and `.rpm` packages are unchanged and keep using the system Vulkan stack.
- The release pipeline proves this with a catalog-equivalent launch test: the freshly built AppImage is started on ubuntu-22.04 under Xvfb, a window manager, and `firejail --appimage`, with no system Vulkan driver, and must show a window before the release publishes.

### Fixes

- Linux startup failures report a localized explanation and exit code instead of an opaque renderer panic.
- AI settings no longer silently discard unsaved edits, and setup no longer requires discovering an unmarked Advanced toggle to reach the base URL.

## Compatibility

- No migration is required for Markdown files, preferences, or workspace data.
- AI is disabled by default and performs no network or tool activity until you enable it and configure a service.
- AI profiles saved on the previous factory limits are upgraded on load; profiles with any customized limit are left as saved.
- Windows installers remain unsigned, so SmartScreen still requires **More info → Run anyway**. macOS DMGs remain ad-hoc signed (no Developer ID signing or notarization), so Gatekeeper still warns on first launch and is dismissed via *System Settings → Privacy & Security → Open Anyway*.
- The Linux AppImage grows by about 20 MB compressed; DEB and RPM installs are unaffected.

## Downloads

- Windows x64: `markion_0.4.9_x64-setup.exe` (NSIS) and `markion_0.4.9_x64-portable.zip`.
- macOS Apple Silicon: `Markion_0.4.9_aarch64.dmg`.
- macOS Intel: `Markion_0.4.9_x64.dmg`.
- Linux x86_64: `markion_0.4.9_amd64.deb`, `markion_0.4.9_x86_64.rpm`, and `markion_0.4.9_x86_64.AppImage`.

## Verification

- `cargo test --workspace` passed locally (1991 passed, 0 failed, 5 ignored), and the packaged MarkNice workspace bundle verified (23 files).
- Windows, macOS arm64, macOS x86_64 (cross-compiled), and Linux native CI packaging builds passed, including the AppImage smoke test on a host with no Vulkan driver.
- The updater signature, `update.json`, and the Aliyun OSS mirror were verified after publication.

**Full comparison**: https://github.com/willmove/markion/compare/v0.4.8...v0.4.9

---

# Markion v0.4.9（中文说明）

新增可选、可配置的 AI 助手——对话、写作操作与经审核的工作区操作，默认关闭；并让 Linux AppImage 在无 Vulkan 驱动的机器上通过内置软件 Vulkan 回退正常启动。

## 主要更新

### AI 助手（新功能，默认关闭）

- AI 默认关闭，启用后才生效。打开 **偏好设置 → AI Agent**，或按 **Ctrl+Shift+A**（Windows/Linux）/ **Cmd+Shift+A**（macOS）。服务与密钥由用户自行提供，不含内置订阅或账号登录。
- 预设覆盖 OpenAI、Anthropic、DeepSeek、Gemini、xAI、Mistral、Moonshot、Qwen、Zhipu、OpenRouter、Local/Ollama 与 Custom。支持 OpenAI Responses、OpenAI 兼容 Chat Completions 与 Anthropic Messages 协议，本地或自定义服务器可无需密钥。云端预设会填入可编辑的起始模型 ID。
- API 密钥保存在操作系统凭据存储中，也可选择仅本次会话使用。密钥不会写入偏好设置、对话历史、日志或 AI 请求内容。
- **测试连接**使用合成文本与无害的工具探测校验完整草稿，**查找模型**可在确定模型 ID 前列出候选项，**保存并启用**会先持久化已测试的草稿再启用 AI。保存失败会保留草稿且不启用 AI。
- 对话面板可调整宽度，标题栏与输入区固定、消息区自适应；支持流式 Markdown 回答、追问、停止与重试、可移除的上下文附件（选区、文档或显式选择的文件），以及仅来自真实附件与工具读取的来源引用。关闭面板后状态栏保留运行指示，重新打开即可停止或查看。可选的本地历史上限为 20 个对话 / 8 MiB，且不包含原始附件、工具载荷、密钥、授权与审批。
- 提供十二种写作操作：起草、续写、润色、纠正语法、缩短、扩写、调整语气、翻译、摘要、提取大纲、提取任务与自定义指令。生成过程绝不直接修改文档：先审阅目标位置与前后文本，再选择复制、替换、插入或新建未保存笔记。应用后的写作变更是独立于此前输入的单次撤销组，目标变更或已关闭时绝不会改写其他标签页。
- 工作区模式提供有界列目录、文件名/文本搜索与精确范围读取，并可提议文本编辑、新建笔记、创建文件夹以及单个移动/重命名。用户为工作区内某个文件夹授予读取权限，逐条审阅操作及其前后文本与依赖关系，并可随时撤销授权。授权仅属于当前会话，不会从历史中恢复。
- 助手不具备 shell、外部系统或浏览器自动化、Git 写操作、发布与永久删除能力。它会拒绝过期的文档与磁盘结果，文件有未保存修改时要求先保存，遵守 Git 写入准入，并校验文件系统边界。文件变更写入持久操作日志，**恢复文件操作**界面即使在 AI 关闭时也可用。
- 全部 AI 文案已在七种界面语言（英语、简体中文、繁体中文、日语、法语、德语、西班牙语）本地化，AI 面板动作支持自定义快捷键。

### AI 设置改进

- 引导式基础设置：Local 与 Custom 预设的 Base URL 与协议直接显示在基础设置中，协议改为下拉选择，校验失败会指明具体字段而非笼统报错。
- 测试连接与模型发现无需先打开总开关；未配置安装上测试成功后会提供一键**启用 AI**。
- 限制项改为各自带标签的数字输入框。出厂默认值提高——上下文 512 KiB（原 64 KiB）、输出 512 KiB、输出 token 8192——仍处于旧出厂配置的档案会在加载时升级，任何已自定义的档案保持不变。
- 字节限制以整 KiB 编辑，并显示近似 token 提示与**恢复推荐限制**；输出 token 与空闲超时上限提高，长文本与推理模型回复不再被过早截断。
- 防丢失保护：草稿有未保存修改时，切换服务预设、档案或标签页会先提示；写作指引的保存与档案提交相互独立。
- 保存、测试、发现、忘记与停止改用专用状态提示，不再复用按钮文案；设置反馈与面板反馈分离；已启用但配置不可用时会指出缺失项并提供跳转到对应字段的操作。
- 档案、对话标签、附件与来源链接均可用键盘聚焦与激活。
- 偏好设置标签栏现在保持单行不换行，面板过窄时横向滚动，不再把第六个标签挤到第二行。

### Linux AppImage：内置软件 Vulkan 回退

- AppImage 现在可在没有可用 Vulkan 驱动的机器（虚拟机、云桌面、精简安装）上启动，此前这类环境会在一秒内以 `NoSupportedDeviceFound` 崩溃退出。包内 `usr/lib/markion-vulkan/` 携带 Mesa lavapipe 软件驱动与 Khronos Vulkan 加载器。
- 始终优先使用硬件 Vulkan：仅在渲染器初始化失败后，Markion 才会用内置驱动重启一次；有可用 GPU 驱动的机器不会加载内置驱动。`MARKION_SOFTWARE_VULKAN=1` 强制使用内置驱动，`MARKION_SOFTWARE_VULKAN=0` 关闭回退。
- 当完全无法初始化渲染器时，Markion 会记录错误、输出本地化的说明与修复步骤，在系统存在 `zenity`、`kdialog` 或 `xmessage` 时弹出原生对话框，并以非零状态退出，不再是裸崩溃。
- 内置驱动使 AppImage 体积增加约 20 MB（压缩后）。`.deb` 与 `.rpm` 包保持不变，继续使用系统 Vulkan。
- 发布流程新增等价于 AppImage 目录测试的启动验证：在 ubuntu-22.04 上以 Xvfb、窗口管理器与 `firejail --appimage` 启动刚构建的 AppImage，且不安装系统 Vulkan 驱动，必须显示窗口才允许发布。

### 修复

- Linux 启动失败时给出本地化说明与退出码，取代难以定位的渲染器崩溃。
- AI 设置不再静默丢弃未保存的修改，也不再需要先找到未标注的高级选项才能填写 Base URL。

## 兼容性

- Markdown 文件、偏好设置与工作区数据无需迁移。
- AI 默认关闭，在启用并配置服务之前不会产生任何网络或工具活动。
- 使用旧出厂限制的 AI 档案会在加载时升级；任何已自定义限制的档案按原样保留。
- Windows 安装包仍未签名，首次启动仍需 SmartScreen 的**更多信息 → 仍要运行**。macOS DMG 仍为 ad-hoc 签名（无 Developer ID 签名或公证），首次启动会触发 Gatekeeper 警告，可通过*系统设置 → 隐私与安全性 → 仍要打开*放行。
- Linux AppImage 体积增加约 20 MB（压缩后）；DEB 与 RPM 安装不受影响。

## 下载

- Windows x64：`markion_0.4.9_x64-setup.exe`（NSIS）与 `markion_0.4.9_x64-portable.zip`。
- macOS Apple Silicon：`Markion_0.4.9_aarch64.dmg`。
- macOS Intel：`Markion_0.4.9_x64.dmg`。
- Linux x86_64：`markion_0.4.9_amd64.deb`、`markion_0.4.9_x86_64.rpm` 与 `markion_0.4.9_x86_64.AppImage`。

## 验证

- 本地 `cargo test --workspace` 通过（1991 通过、0 失败、5 忽略），打包的 MarkNice 工作区校验通过（23 个文件）。
- Windows、macOS arm64、macOS x86_64（交叉编译）与 Linux 原生 CI 打包构建全部通过，包括在无 Vulkan 驱动主机上的 AppImage 冒烟测试。
- 更新器签名、`update.json` 与阿里云 OSS 镜像已在发布后验证。

**完整变更对比**: https://github.com/willmove/markion/compare/v0.4.8...v0.4.9
