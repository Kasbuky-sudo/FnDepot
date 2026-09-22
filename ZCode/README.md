# ZCode · NAS 版

把 [Z.ai 开源的 AI 编程工作台 ZCode](https://github.com/zai-org/ZCode) 跑在飞牛 fnOS 上的**原生应用**（非 Docker）：
服务端跑在 NAS 上，浏览器 / 飞牛桌面里直接用完整工作台 ——
**Agent 对话、代码工作区、内置终端**，关掉浏览器任务照常继续。

- **开发者**：[Z.ai (zai-org)](https://github.com/zai-org/ZCode)（上游 ZCode，Apache-2.0）
- **飞牛移植 / 打包 / 发布**：[Kasbuky-sudo](https://github.com/Kasbuky-sudo)
- 原生 Node.js 运行，复用应用中心 **Node.js v22**（`nodejs_v22`，安装时自动作为前置依赖装上）；原生模块随官方预编译件分发（linux-x64 / linux-arm64），一个包通吃 x86_64 / arm64
- 端口 **8988**，桌面图标页内（iframe）打开
- 登录态与工作区保存在 NAS 上：全设备共用一份；应用数据目录在升级 / 重装后保留
- 默认工作区：安装时创建共享目录 `zcode/workspace`；要操作已有目录，在应用设置的「访问权限」里授权后重启应用
- 基于 ZCode 官方 Web 运行时（`zcode --web`），与桌面版共用同一套前端组件

## 功能

- 完整工作台界面：Agent 对话、任务编排、代码差异视图（与 ZCode 桌面版同一套前端）
- 内置终端：服务端 node-pty，浏览器里直接用，与桌面端体验一致
- 服务端权威：关掉浏览器长任务继续跑，回来接着看
- 内置访问 token：首启自动生成，浏览器经飞牛桌面入口带 token 打开

## 说明

- 首次使用需在 ZCode 里登录 Z.ai 账号（或配置模型 API），登录态保存在 NAS 上
- 浏览器自动化（CUA）等依赖本机桌面的功能在 NAS 上不可用
- 首个飞牛版本为真机验收版，遇到问题请到 [NAS-ZCode](https://github.com/Kasbuky-sudo/NAS-ZCode/issues) 反馈

## 链接

- 源码与问题反馈：<https://github.com/Kasbuky-sudo/NAS-ZCode>
- 上游项目：<https://github.com/zai-org/ZCode>
