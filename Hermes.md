---
name: Hermes
description: Sanitized Hermes Android APK project handoff memory for the public repository.
metadata:
  node_type: memory
  type: project
updated_at: 2026-07-12 04:45:00 +08:00
privacy: Public copy; local paths, device identifiers, network addresses, session IDs, and credentials are intentionally omitted.
---

# Hermes Android 项目读档

## 读档规则

1. 新会话先完整读取本文件，以 `updated_at`、`当前状态`、`当前阻塞`、`下一步` 为准。
2. 不要从历史归档中的旧“下一步”恢复任务，除非本文件明确要求回查。
3. 有新的架构决定、源码路径、构建结果、真机结论、阻塞或用户要求时，立即更新本文件；不要等任务结束。
4. 只把可接续结论写入本文件。详细操作流水追加到新的日期归档，避免主文档再次膨胀。
5. 涉及 UI 架构、首屏、导航或交互模式的大改，先向用户说明方案并等待确认。

## 项目目标

- 将完整 Hermes Agent 做成普通用户安装后即可使用的 Android App。
- 用户侧不安装 Termux、不手动部署 Agent、不依赖远程服务器或命令行。
- Hermes Agent、Python、Dashboard、Node/TUI 和必要依赖由 APK 内置，在手机本地初始化和运行。
- UI 延续 Hermes Desktop GUI 的产品气质，并针对手机使用抽屉、底部导航和移动能力页。
- 可以接受 APK 较大，依赖和构建问题尽量在打包阶段解决。

## 当前状态

### 已跑通的底层闭环

- APK 已内置 Python Agent、Dashboard 静态资源、Node.js/TUI runtime 及 ARM64 依赖。
- 本机 Dashboard 可在 `127.0.0.1:9129` 启动。
- 真机验证过 Agent 主进程、Node 子进程和 PTY 同时存活。
- 基础聊天使用 `/api/ws` JSON-RPC：`session.create`、`prompt.submit`、`message.delta`、`message.complete`。
- 已处理英文流式 token 空格、助手透明正文样式和 stale session 自动重建。
- 模型配置、文件选择、附件入口、移动抽屉和基础导航已经可用。
- 2026-07-11 按用户确认完成移动抽屉重构：抽屉改为 Gemini/豆包风格，直接提供“发起新对话”、会话搜索、最近会话、当前会话高亮、切换和删除；工作台、连接及其余能力入口全部迁入左下角齿轮设置页。
- 设置页已重新排版并将 Hermes 社区置顶；工作台、连接、基础设置、Agent 能力和高级入口按组展示，终端模式移至设置页底部高级工具。

### 已真机验证的真实功能页

- `资源中心`：读取 `/api/files`，支持目录浏览和下载。
- 2026-07-11 真机修复资源中心目录浏览：目录按钮不再重复调用 `switchView("resources")`，避免目标目录请求被加载锁丢弃；随后通过 ADB 真实坐标点击和连续截图发现，深入长路径时页面会被路径文本横向撑宽，造成按钮和卡片裁切，用户观感类似“页面崩了”。现已限制资源页横向溢出、让长路径自动换行，并加入“返回上级/根目录”；真实触摸回归通过。
- `定时自动化`：读取 Cron 任务、投递目标和模板，已有触发/暂停/恢复操作。
- `接入`：读取真实平台和 Gateway 状态，已有 Gateway 启停及平台测试。
- `远程实例`：读取 Dashboard/Gateway、配对请求和 Profile 状态。
- `记忆管理`：真实状态读取已通过。
- `Skills Hub`：真实状态、来源和推荐列表读取已通过。

### 已实现但尚未完成真机验收

- `Persona`：曾出现纯黑 WebView；已将可选接口超时降为 5 秒，并增加页面级异常兜底，新 APK 尚未复测。
- `MCP`：已接 `/api/mcp/servers` 和 `/api/mcp/catalog`。
- `工具集`：已接 `/api/tools/toolsets`。
- `用量`：已接 `/api/analytics/usage?days=30`。
- `外观`：已接 `/api/dashboard/themes` 和 `/api/dashboard/font`。
- 上述页面目前主要提供真实读取、刷新和“高级设置”跳转，还不是完整移动控制台。

### 尚未真实接入

- `沙箱`
- `子代理`
- `命令面板`

这些入口已有产品文案，但尚未加入移动端真实数据页和操作闭环。

## 当前阻塞

- 2026-07-12 04:45 +08:00 已按用户要求清空 Windows `C:\000` 文件夹，并将当前最新 APK 复制为 `<local-delivery>/Hermes-latest-arm64-debug.apk`。文件大小 `105,304,410` bytes，APK 生成时间 `2026-07-12 04:37:18 +08:00`。`C:\000` 当前只保留这一份最新 APK，可作为本轮对外交付文件。

- 2026-07-12 04:37:53 +08:00 完成手机会话清理与同步修复。清理前 `state.db` 有 40 个 sessions、216 条 messages，`hermes-home/sessions` 有 2 个 request_dump JSON；已备份为 `state.db.backup-<timestamp>` 和 `session-backup-<timestamp>/`，随后清空会话相关表和活动 dump，最终交付状态为 sessions=0、messages=0、活动 dump=0。移动抽屉“删除会话”和“清空当前”现会先调用手机内 `DELETE /api/sessions/{backendSessionId}`，后端成功后才删除本地记录，失败则保留前端对话并提示。真机回归发现 message.complete 之前仍优先返回短网关 key，现已改为优先 `agent.session_id` 的真实长数据库 ID，确保删除和跨重启 resume 都指向同一 state.db 记录；最终版 APK 于 `2026-07-12 04:37:53` 覆盖安装。语言页从占位改为中文/英文真实选择，取消启动时强制写死 zh，并让终端/高级 Dashboard 路由动态携带 locale；真机确认 localStorage 与 document lang 可切到 en。外观页新增移动浅色/深色、Dashboard 主题应用、字体切换，调用 `/api/dashboard/theme` 与 `/api/dashboard/font`；真机确认移动主题切换生效，主题和字体按钮可调用。当前移动壳静态中文文案尚未完整国际化，语言切换主要影响持久语言状态及 Dashboard/终端页面，这是后续可继续完善项。

- 2026-07-12 04:03:10 +08:00 已通过新无线 ADB `<wireless-adb-address>` 安装最终会话续接根修版。真机回归发现仅持久化 `session.create` 返回的短网关 ID（如 `<gateway-session-id>`）仍无法跨 App 进程恢复；真正写入 `state.db` 的 Hermes 会话 ID 为长格式（如 `<persistent-session-id>`）。现已修改 Android 内置 `tui_gateway/server.py`：每个主 `message.complete` payload 增加当前真实 `session_key`；移动桥收到后立即覆盖保存到当前抽屉会话的 `backendSessionId`。App 重启后使用该真实数据库 ID 调用 `session.resume`，并采用返回的新临时 live `session_id` 提交后续 prompt。此前的异步 WebSocket 保活、下一消息 resume、会话切换隔离也一并保留。APK 构建和覆盖安装成功，设备 `lastUpdateTime=2026-07-12 04:03:10`。升级前或中间测试版已保存的短 ID 会在 resume 失败后自动重建；需在最终版中新建/继续一次对话，让下一次完整回复写入真实长 session_key，之后退出重进即可恢复上下文。异步 `delegate_task` 回传仍待最终版真机复测。

- 2026-07-12 03:36 +08:00 已修复“App 退出重进后仍显示同一对话，但 Agent 认为是新会话”：根因是移动抽屉只持久化消息和标题，后端 `session_id` 仅保存在 JS 运行内存。现为每个移动会话新增持久化 `backendSessionId`，创建后端 session 时立即写入 localStorage；App 重启、抽屉切换后会加载对应 ID，并在发送前通过 `session.resume` 恢复 Hermes 上下文。新建会话清空绑定；切换/删除/清空当前会话会关闭旧异步监听并切换或清除绑定；resume 或 submit 返回 stale session 时清除旧 ID、创建新 session 并持久化。该修复已与异步子代理回传修复合并，两份桥接文件语法通过、SHA-256 一致，APK 构建成功。注意升级前已经存在的旧移动对话没有历史 backendSessionId，安装新版后首次继续发送仍会建立一次新后端绑定；从该次开始以后退出重进即可保持同一 Agent 上下文。待新的无线 ADB 端口覆盖安装并真机验证。

- 2026-07-12 03:27 +08:00 已实现移动异步子代理回传修复：首个 `message.complete` 后不再立即关闭 `/api/ws`，同一 WebSocket 保持为后台任务监听；后续 `message.delta/message.complete` 会创建新的助手消息并持久化显示。下一条用户消息发送前关闭旧监听，并在新 WebSocket 上先调用 `session.resume` 重新绑定既有 session，再提交 prompt，失败则自动新建 session。两份桥接文件语法通过、SHA-256 一致，APK 构建成功。安装时无线 ADB `<wireless-adb-address>` 已失效并连续返回 connection refused，待用户提供新的无线调试端口后覆盖安装并用 `delegate_task` 真机复测。

- 2026-07-12 03:14 +08:00 已通过 Android WebView DevTools 直接操控真实移动聊天页完成工具回归。当前会话暴露 14 个工具：`delegate_task`、`memory`、`patch`、`process`、`project_create`、`project_list`、`project_switch`、`read_file`、`search_files`、`skill_manage`、`skill_view`、`skills_list`、`terminal`、`write_file`。只读调用 7/7 通过：process list、project list、read_file、search_files 两种模式、skill_view 正确错误返回、skills_list、terminal；终端同时验证 Android ARM64 系统、Hermes home 文件、SOUL.md、空记忆目录、Cron 目录和外网 `example.com` HTTP 200。可逆写入链通过：write_file→read_file→patch→read_file→terminal 删除，临时 Skill 创建→skill_view→删除→skills_list 清理均成功，确认无残留。`memory`、`project_create`、`project_switch` 因会留下持久状态未测试。唯一缺陷：`delegate_task` 后端实际成功完成并生成 `DELEGATE_OK`，但异步结果回注时移动 `/api/ws` 已被客户端关闭；日志出现 `ws send failed ... WebSocketDisconnect`，导致子代理结果未显示到移动聊天。需修移动 RPC 在主消息 complete 后保持/重连会话，接收 `[ASYNC DELEGATION BATCH COMPLETE]` 后续事件。

- 2026-07-12 02:43:21 +08:00 已补齐移动“接入”页的平台启停控制：复用手机内后端现有 `PUT /api/messaging/platforms/{platform_id}`，每个平台现在显示“启用/停用”和“测试”；启用已配置平台或停用运行中平台后会调用 `/api/gateway/restart`，未配置平台启用后提示先到高级设置补充必要信息。两份移动桥接文件语法检查通过且 SHA-256 一致；APK 构建成功并通过 `<wireless-adb-address>` 覆盖安装，设备 `lastUpdateTime=2026-07-12 02:43:21`。App 重启后手机 Dashboard `0.18.0` 正常，Gateway 当前仍为未运行，待用户在设置→接入中验证新按钮并选择需要启用的平台。

- 2026-07-12 02:18 +08:00 纠正环境判断：Hermes Agent/Dashboard 完整运行在 Android App 内，与 Windows 本机服务无关。已通过 ADB 将手机 `127.0.0.1:9129` 转发到电脑 `127.0.0.1:19129` 并成功读取手机 `/api/status`：Dashboard 版本 `0.18.0`、配置版本 `33`，手机内服务正常；但 `gateway_running=false`、`gateway_pid=null`、`gateway_platforms={}`。真机“接入测试”返回 `WeCom (app) is disabled. Enable it, then restart the gateway.`，说明移动前端已成功调用手机内后端测试接口，当前故障在手机内消息平台配置/Gateway 运行层，不是 Windows，也不是前后端未连接。截图同时确认新流式回复中的 `WeCom` 和 `Hermes Agent` 已不再被错误拆词，英文 token 空格修复通过初步真机回归。

- 2026-07-12 01:02:00 +08:00 EasyTier 免费方案已判定不可用并完整撤销：官方共享节点 DNS/握手异常，云电脑 IPv4 受上层 NAT 限制，所分配 IPv6 无真实外网路由，手机始终未进入对等列表。EasyTier 进程、虚拟网卡、防火墙规则、程序目录、APK、配置和临时源码均已删除；蒲公英 PgyVPNEntService 已恢复 Automatic 并启动，相关后台进程运行中。后续继续使用蒲公英或另选方案，不要从旧 EasyTier 配置恢复。
- 2026-07-12 01:28:26 +08:00 已通过新无线 ADB `<wireless-adb-address>` 成功连接测试设备，并使用 `adb install -r` 成功覆盖安装最新 debug APK；`com.sesaloy.hermes` 的 `lastUpdateTime` 已更新为 `2026-07-12 01:28:26`。此前大文件传输导致 `offline` 的阻塞本次未复现，真机回归可继续推进。
- 2026-07-11 23:45:00 +08:00 已确认蒲公英隧道是断连根因：127MB ADB 推送约 11 秒后，ADB、ICMP 与 TCP 端口同时不可达；电脑出口 IPv4 38.15.15.6 的入站端口测试被上层 NAT 拦截，不能直接自建 WireGuard 服务端。已改部署 EasyTier 免费共享节点方案，电脑端虚拟地址为 10.88.0.1；Android APK 与私有配置说明已通过企业微信发送，待手机端安装、授权 VPN 并入网后使用其 10.88.0.2 复测 ADB。PgyVPNEntService 已停止并设为 Disabled，蒲公英相关进程已终止。
- 2026-07-11 03:45:00 +08:00 无线 ADB `<wireless-adb-address>` 状态为 `device`，实体测试设备。
- 资源中心“打开”、深层目录布局和返回导航已修复并通过真实坐标点击、截图验证，不再是当前阻塞。
- 下一次真机工作必须先运行 `adb devices -l`；不要假设旧无线端口仍有效。
- 最新 Persona 修复已经打包，但尚未安装到手机验证。
- 新抽屉和设置页已通过真机真实坐标点击及截图验证；会话显示、搜索框、新建入口、设置齿轮和社区置顶均生效。
- 用户反馈当前返回机制不完整：多个能力页进入后无法可靠返回，部分入口会强制进入终端模式，终端模式缺少返回图形界面的可见入口。当前第一优先级改为统一页面返回栈、Android 系统 Back 处理和终端退出闭环。
- 2026-07-11 20:42:22 +08:00 已完成返回闭环源码修复并成功 `assembleDebug`：移动壳已有抽屉优先关闭、能力页返回设置、设置页返回聊天；原生 Back 在 Dashboard 中优先 `WebView.goBack()`，无历史时才回移动首页；终端入口新增明确确认提示，终端页保留“返回移动界面”按钮。两份桥接文件 SHA-256 一致。待无线 ADB 稳定后安装验证。

## 下一步

按以下顺序推进，不要回退到旧 Termux 路线或旧 TTS 主线：

1. **保持真机连接**
   - 每次开始前运行 `adb devices -l`，确认当前无线地址仍为 `device`。
   - 当前最新 debug APK 已安装；继续抓取截图、WebView console 和 logcat 做能力页回归。

2. **回归当前构建**
   - 重点复测 Persona 黑屏修复。
   - 验证 MCP、工具集、用量和外观页面的真实字段、刷新与返回行为。
   - 修复系统 Back 和页面内返回：能力页应优先返回设置页/上一级页面，抽屉打开时先关闭抽屉，设置页再返回聊天，不应直接退出 App。
   - 修复强制进入终端及终端无法返回图形界面：所有终端入口应有明确确认或可见返回按钮，并处理系统 Back。
   - 做冷启动、杀进程重启、锁屏恢复和 stale session 回归。

3. **补齐剩余真实状态页**
   - 沙箱 → 子代理 → 命令面板。
   - 先读取真实状态和错误信息，再增加操作；不要先做破坏性动作。

4. **把状态页升级为操作闭环**
   - 低风险优先：外观/字体、Persona 切换、工具集启停、MCP 启停与测试。
   - 再做 Skills 安装卸载、记忆编辑清理、沙箱生命周期、子代理委派和命令执行。
   - 删除、重置、卸载、销毁等操作必须有确认、结果反馈和失败恢复。

5. **稳定性与发布工程**
   - 后台保活、进程崩溃恢复、首次初始化恢复、网络/API 异常处理。
   - 处理 SQLite FTS5 缺失和插件加载警告。
   - 最后单独处理 TTS/语音遗留，不要覆盖当前能力页主线。
   - 建立 release 签名、版本升级、配置迁移、全新安装和覆盖升级验证。

## 关键路径与工程状态

- 主读档：`<workspace>/Hermes.md`
- 2026-07-10 精简前完整快照：`<private-archive>/Hermes-log-2026-07-10.md`
- 早期历史归档：`<private-archive>/Hermes-history.md`
- Android/APK 工程：`<workspace>/hermes-mobile`
- Agent 参考源码：`<workspace>/hermes-agent`
- 完整 Agent/构建源码：`<workspace>/hermes-agent-full`
- 常见 APK 输出：`<workspace>/hermes-mobile\android\app\build\outputs\apk\debug\app-debug.apk`
- 当前 APK：`127,364,651` bytes，生成时间 `2026-07-11 03:38:07 +08:00`，包名 `com.sesaloy.hermes`，`arm64-v8a` debug；已覆盖安装并通过新抽屉、会话列表、设置齿轮、社区置顶和功能迁移真机回归。
- 最新已安装/交付 APK：`105,304,410` bytes，生成时间 `2026-07-12 04:37:18 +08:00`；已通过无线 ADB `<wireless-adb-address>` 覆盖安装到真机，交付副本位于 `<local-delivery>/Hermes-latest-arm64-debug.apk`。包含会话真实 ID 持久化、跨重启恢复、异步回传保活、移动会话删除/清空与后端同步、接入启停、语言状态和外观控制修复。
- 构建命令：
  - `cd <workspace>/hermes-mobile\android`
  - `. <local-tools>/setenv.ps1; .\gradlew.bat assembleDebug --console=plain`

### 变更保护

- `<workspace>/hermes-mobile` 不是 Git 仓库；大改前必须备份或先建立版本基线。
- 两份桥接文件必须保持同步：
  - `<workspace>/hermes-mobile\www\hermes-mobile-bridge.js`
  - `<workspace>/hermes-mobile\android\app\src\main\assets\public\hermes-mobile-bridge.js`
- 最近一次核对两份文件 SHA-256 相同。
- `hermes-agent` 有 5 个既存未提交的桌面聊天/语音改动。
- `hermes-agent-full` 有 11 个既存未提交的 TTS、Web Server、Web UI 和国际化改动。
- 不要覆盖、还原或清理这些既存改动；修改前先执行 `git status --short`。

## 必须保留的技术约束

### Android 本机运行

- 当前 APK 仅支持 `arm64-v8a`。
- Node 可执行文件作为 `jniLibs/arm64-v8a/libnode_exec.so` 打包，因为 Android 不允许直接执行私有 files 目录中的普通二进制。
- Gradle 必须保留 `packagingOptions { jniLibs { useLegacyPackaging true } }`。
- 启动环境涉及 `HERMES_NODE`、`HERMES_ANDROID_NATIVE_LIB_DIR`、`HERMES_TUI_DIR`、`HERMES_SKIP_NODE_BOOTSTRAP`、`LD_LIBRARY_PATH`。
- Dashboard 静态构建产物必须存在于 APK 的 Chaquopy assets 中，否则启动后会出现 `Frontend not built`。

### WebView 与移动桥

- 不要只用 `/api/status` 判断本机服务是否真正可用。
- WebView 容易受同步桥长请求、透明遮罩、z-index、fixed header、安全区和全局 button CSS 影响。
- 页面请求必须有超时和渲染异常兜底，避免单页异常让整个移动壳黑屏。
- 文件附件应拉起 Android DocumentsUI。
- 不要全局启用 `CapacitorHttp.enabled: true`，否则本地静态资源可能白屏。
- 流式响应处理必须保持字节类型一致，不能把 `Uint8Array` 当字符串直接拼接。

### 安全与产品文案

- 不得把 API 密钥写入源码、`.env.local`、APK、日志、项目记忆或回复。
- 面向普通用户的界面不要出现：Termux、部署、内置模式、core/runtime、filesDir、dataDir、nativeLibraryDir、包名、调试路径。
- 使用普通用户文案，例如：`正在准备`、`服务已启动`、`需要完成模型设置`、`连接失败，请检查 API 端点或密钥`。

## 已知遗留问题

- 2026-07-12 01:48:07 +08:00 根据真机截图确认并修复移动聊天流式英文 token 错误补空格：`Hermes` 曾显示成 `Herm es`、`DeepSeek` 曾显示成 `Deep Se ek`。两份 `hermes-mobile-bridge.js` 已移除 `appendAgentDelta` 自动补空格逻辑并保持 SHA-256 一致；修复版 APK 于 `2026-07-12 01:46:42 +08:00` 构建成功，并通过 `<wireless-adb-address>` 覆盖安装成功，设备 `lastUpdateTime` 为 `2026-07-12 01:48:07`。待用户发送新消息完成显示回归。
- Persona 修复待真机验证。
- Android 系统 Back 缺少移动页面内部返回处理。
- Chaquopy SQLite 缺 FTS5，全文会话搜索目前禁用。
- 部分插件目录缺 `__init__.py`，需区分资源目录警告和真实缺包。
- TTS/语音仍有遗留，但除非用户明确要求，否则不作为当前第一优先级。
- 旧 localStorage/session 可能保留早期坏消息，可后续做一次性迁移或清理。
- 当前是 arm64-only debug APK，尚未完成正式签名、升级迁移和发布验收。

## 测试环境结论

- 当前 Windows 是 KVM 云电脑，约 8 个逻辑处理器、32GB 内存，但没有向客体暴露嵌套虚拟化。
- 当前未安装 Android Studio Emulator、system image 或 AVD。
- Hermes 只有 ARM64 runtime；该云电脑不适合用 Android Emulator 代替真机。
- 项目继续采用实体 Android 手机测试，优先真机 ADB。

## 协作与文件交付

- 用户通常通过企业微信/WeChat Work 的 `cc-connect` 沟通，普通文本会自动送达，不要重复发送。
- 用户无法直接打开本机 Windows 路径。
- 用户要求发送、推送、附加本地文件时，使用：
  - `cc-connect send --file "C:\absolute\path\file" -p "$env:CC_PROJECT" -s "$env:CC_SESSION_KEY"`
- 若返回 `errcode=846607`，等待约 60 秒后只重试一次。

## 历史索引

- `Hermes-log-2026-07-10.md`：精简前主文档的完整 1061 行快照，包含 2026-07-04 至 2026-07-10 的详细操作、构建、ADB、截图和修复流水。
- `Hermes-history.md`：更早的移植历史、Termux 试验、WebView/UI 演进和历史踩坑。
- 历史 Termux 自动部署路线仅供复盘，不是当前产品方向。












## 公开仓库隐私规则

- 本文件为公开副本；不得写入 API 密钥、令牌、个人路径、设备标识、内网地址、真实会话 ID 或企业微信会话信息。
- 不提交本机配置、签名文件、数据库、会话 dump、设备备份、日志、APK 或其他构建产物。
