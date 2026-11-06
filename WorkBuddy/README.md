# WorkBuddy · 飞牛 fnOS 原生版（x86_64 / arm64）

把腾讯 WorkBuddy 的 Linux 运行时适配成飞牛 fnOS 的**原生 Web 应用**（非 Docker、非桌面串流）：
服务端跑在 NAS 上，浏览器 / 飞牛桌面里直接用完整工作台——Agent 对话、文件与终端操作、办公文档、专家技能、MCP / Skills，关掉浏览器后任务仍在 NAS 上继续。

- **开发者**：腾讯科技（深圳）有限公司（WorkBuddy，专有软件）
- **飞牛移植 / 打包 / 发布**：[Kasbuky-sudo](https://github.com/Kasbuky-sudo)
- **同时支持 x86_64 与 arm64**，两个独立安装包；原生模块按架构提供 linux-x64 / linux-arm64
- 原生 Node.js 运行，复用应用中心 **Node.js v22**（`nodejs_v22`，manifest 里声明了 `install_dep_apps`，安装时作为前置依赖自动装上）
- 端口 **28471**，桌面图标页内（iframe）打开
- 登录态与工作区保存在 NAS 上，升级 / 重装后保留
- 安装向导设置**访问令牌**；桌面入口自动携带（`/<令牌>`），局域网内其他设备没有令牌无法使用

## 功能

- Agent 对话与任务执行：文件读写、终端命令、Web 搜索 / 抓取、任务与计划模式、子 Agent
- 办公与内容场景：Word / Excel / PPT / PDF 处理、深度研究、专家技能与内置专家
- 扩展体系：MCP、Skills、插件、自定义 Agent、知识库
- 服务端权威：关掉浏览器，长任务继续在 NAS 上跑
- 访问令牌鉴权：入口路径携带令牌，并通过 cookie 保护静态资源与 WebSocket

## 安装与依赖

1. 在飞牛应用中心确认应用商店可用（本应用依赖 **Node.js v22**，应用中心会按 `install_dep_apps` 自动安装 / 启用）；
2. 从 [Releases](https://github.com/Kasbuky-sudo/FnDepot/releases/tag/WorkBuddy-v5.6.2.4) 按设备架构下载 `workbuddy-5.6.2.4-x86.fpk`（x86_64）或 `workbuddy-5.6.2.4-arm64.fpk`（arm64）；
3. 应用中心 → 手动安装 → 选择 fpk；
4. 安装时填写「访问令牌」（8-64 位字母、数字、下划线或中划线）；
5. 安装完成后从桌面图标打开。

> 支持 x86_64 与 arm64 设备；请按设备架构选择对应 fpk，不能混装。
>
> 想让 FnDepot 显示「可更新」并接收后续版本提示，请通过 FnDepot 安装或覆盖安装本应用；直接手动安装的 fpk 会被 FnDepot 视为「本地应用」，按 FnDepot 的规则不提供更新入口。
>
> 卸载时请选择「保留数据」（默认选项），登录态、MCP / 技能配置和会话会保留在应用数据目录，重新安装后可直接恢复。

## 访问令牌

- 安装向导里的令牌会自动写进入口路径，平时无需手动输入。
- 想更换：应用中心 → 已安装 → WorkBuddy → 应用设置 → 修改「访问令牌」→ 重启应用。
- 令牌只保护入口与 WebSocket；请勿把带令牌的完整链接发给不可信的人。

## 登录

- 首次打开点「登录」后，浏览器会尝试打开 WorkBuddy 登录页；如果被拦截，页面顶部会出现「打开登录页」提示条，也可以点「复制登录链接」手动打开。
- 登录采用浏览器登录 + NAS 端轮询 token 的方式，登录页可以在手机 / 电脑上打开；**NAS 本身需要能访问 `www.workbuddy.cn`**，否则会一直停留在「登录中...」直到超时。
- 局域网 HTTP 入口已内置 `crypto.randomUUID` 兼容，点击登录不会再因为非 HTTPS 环境而无反应。

## 工作区与数据

- 默认工作区：安装时创建共享目录 `workbuddy/workspace`（文件管理器可见）。
- 要操作已有目录，在应用设置的「访问权限」里授权目录，然后重启应用。
- 登录态、配置、会话数据保存在应用数据目录（`TRIM_PKGVAR`），升级 / 重装后保留。

## 已知限制

- **双架构**：x86_64 与 arm64 分别打包；ARM 包内 `better-sqlite3`、`node-pty`、`@tencent/qimei-node`、腾讯文档引擎等均为 aarch64。
- 依赖桌面 Chromium 的功能（如浏览器自动化 / Computer Use、本地模型管理等）在纯 NAS 环境下不可用或受限。
- 首次启动与登录需要外网；内网离线环境只能打开界面，无法完成账号登录。
- 应用更新仍通过 FnDepot / fpk 升级，不走 WorkBuddy 桌面版自动更新。

## 本版修复（5.6.2.4）

- 修复启动脚本进程存活误判：`cmd/main` 的 `pid_alive` 原来用 shell `case` glob 匹配 ERE（含 `?`、`|`、`()`、`[.]`），永远匹配不上，导致启动后先打印 `process exited before listening`，应用中心随后把仍在运行的服务收掉。现改为 `grep -Eq` 正确走正则；启动日志正常出现 `started, port 28471 listening`，daemon ready，端口 28471 正常监听。
- 修复 MCP / `sheetagent` 未连接：NAS 版没有 Electron 主进程，daemon 之前以 `connect-only` 等待一个永远不会被拉起的 `sandbox-center`。本版让 daemon 自行拉起 `sandbox-center`（`spawn-if-missing`）、补齐 `executablePath`，并统一 daemon 与 `sandbox-cli` 的 socket 地址；`editor_sdk` 沙盒恢复，`connector-proxy` / `agent-mail` / `genie-baas` / `weixinpay` / `sheetagent` 全部 connected。
- 修复 `restart` 停不掉 daemon：`cmd/main` 按真实路径匹配 adapter/daemon，并清理 `editor_sdk` / `sandbox-center` / `sandbox-cli` 孤儿进程。
- 启动时继续 chown 应用数据目录，自动修复 `security/<uid>` 被其他用户占用导致的“建安全目录失败 / Operation not permitted”。
- 保留 5.6.2.2 的设备在线与发送修复：`extensions:activateByEvent('onStartup')`、`__wbInvoke` 全参数转发、会话事件订阅聚合、工作空间相对路径解析、只把 LISTEN 当监听。
- 应用内核仍为 WorkBuddy 5.6.2；本版本号是飞牛打包修订版。

## 包信息

- Release：[WorkBuddy-v5.6.2.4](https://github.com/Kasbuky-sudo/FnDepot/releases/tag/WorkBuddy-v5.6.2.4)
- x86_64：`workbuddy-5.6.2.4-x86.fpk`，257,172,200 bytes（约 246 MiB），SHA-256 `a049049f194a6a40290fa1ca9987e306bdbed7dd85449e5b7fd8f577e0b34103`
- arm64：`workbuddy-5.6.2.4-arm64.fpk`，252,656,058 bytes（约 241 MiB），SHA-256 `2c15037f1bcb9786974419252a9f363498394cb224dcd17a2b1b74d9bb47a92d`

## 致谢

- 感谢酷安 **@城北徐公Doreen** 反馈并定位启动脚本 `pid_alive` 的 glob / 正则 bug。

## 链接

- 应用源：[Kasbuky-sudo/FnDepot](https://github.com/Kasbuky-sudo/FnDepot)
- 问题反馈：[FnDepot Issues](https://github.com/Kasbuky-sudo/FnDepot/issues)
- WorkBuddy 官网：<https://www.workbuddy.cn>
