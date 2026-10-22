# WorkBuddy · 飞牛 fnOS 原生版（x86_64）

把腾讯 WorkBuddy 的 Linux 运行时适配成飞牛 fnOS 的**原生 Web 应用**（非 Docker、非桌面串流）：
服务端跑在 NAS 上，浏览器 / 飞牛桌面里直接用完整工作台——Agent 对话、文件与终端操作、办公文档、专家技能、MCP / Skills，关掉浏览器后任务仍在 NAS 上继续。

- **开发者**：腾讯科技（深圳）有限公司（WorkBuddy，专有软件）
- **飞牛移植 / 打包 / 发布**：[Kasbuky-sudo](https://github.com/Kasbuky-sudo)
- **仅支持 x86_64**，暂不支持 arm64（包内原生模块为 linux-x64）
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
2. 从 [Releases](https://github.com/Kasbuky-sudo/FnDepot/releases/tag/WorkBuddy-v5.6.2.2) 下载 `workbuddy-5.6.2.2-x86.fpk`；
3. 应用中心 → 手动安装 → 选择 fpk；
4. 安装时填写「访问令牌」（8-64 位字母、数字、下划线或中划线）；
5. 安装完成后从桌面图标打开。

> 仅支持 x86_64 设备。arm64 设备暂不可安装。

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

- **x86_64 only**：包内 `better-sqlite3`、`@tencent/qimei-node` 等原生模块为 linux-x64，暂未提供 arm64 构建。
- 依赖桌面 Chromium 的功能（如浏览器自动化 / Computer Use、本地模型管理等）在纯 NAS 环境下不可用或受限。
- 首次启动与登录需要外网；内网离线环境只能打开界面，无法完成账号登录。
- 应用更新仍通过 FnDepot / fpk 升级，不走 WorkBuddy 桌面版自动更新。

## 本版修复（5.6.2.2）

- 修复设备在线：补发 `extensions:activateByEvent('onStartup')`，启动常驻 edge-sync 扩展，生成 device-id / edge-sync-mapping-v4.db 并建立云端长连接。
- 修复发送消息无反应：`__wbInvoke` 之前只转发前两个参数，导致工作空间检查与任务创建参数丢失。
- 补齐 `wb:internal:conversation-event-subscription` 的桌面端聚合与转发，会话事件订阅不再报 `no handler`。
- 修复工作空间相对路径解析、`cmd/main` 把 `TIME_WAIT` 误判为监听，以及桌面主进程专用通道的 no-op 处理。
- 应用内核仍为 WorkBuddy 5.6.2；本版本号是飞牛打包修订版。

## 包信息

- Release：[WorkBuddy-v5.6.2.2](https://github.com/Kasbuky-sudo/FnDepot/releases/tag/WorkBuddy-v5.6.2.2)
- 文件：`workbuddy-5.6.2.2-x86.fpk`
- 大小：257,168,727 bytes（约 246 MiB）
- SHA-256：`84fe6e8af25ee2133f01ecf186630121fa6cde5f10f3f5efa233ff56592c02d9`

## 链接

- 应用源：[Kasbuky-sudo/FnDepot](https://github.com/Kasbuky-sudo/FnDepot)
- 问题反馈：[FnDepot Issues](https://github.com/Kasbuky-sudo/FnDepot/issues)
- WorkBuddy 官网：<https://www.workbuddy.cn>
