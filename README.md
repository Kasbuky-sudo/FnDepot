# FnDepot · 飞牛第三方应用源

自打包的飞牛 fnOS **第三方原生应用源**（非 Docker），多数应用兼容 **x86 与 arm64**。

遵循 [FnDepot 外部应用源 V2 规范](https://github.com/EWEDLCM/FnDepot)，仓库根目录的 `fnpack.json` 即源索引文件。

## 添加本源

在 FnDepot 客户端（≥ 0.0.7）的「外部源」中添加仓库地址：

```
https://github.com/Kasbuky-sudo/FnDepot
```

## 应用列表

| 应用 | 版本 | 架构 | 端口 | 说明 |
| --- | --- | --- | --- | --- |
| [旅行青蛙 · NAS 版](https://github.com/Kasbuky-sudo/frog-nas) | 1.0.6 | all（x86 + arm64） | 8980 | 《旅行青蛙·中国之旅》离线包的 NAS 移植：服务端权威引擎、外部推送（Webhook / 鸿蒙 MeoW）、AI Agent 接口；桌面页内打开，进度保存在应用目录，卸载重装不丢 |
| [网易云音乐 · NAS 版](https://github.com/Kasbuky-sudo/NAS-NEM) | 0.4.1 | all（x86 + arm64） | 8163 | 官方 PC 客户端的 NAS 移植：完整播放器（推荐 / 歌单 / 排行榜 / 歌词 / 搜索）、手机扫码登录、在线播放、歌曲下载到 NAS、登录态跨设备共享、AI 找歌下载（Agent API）；依赖应用中心 nodejs_v22，页内打开 |
| [Telegram](https://github.com/Kasbuky-sudo/fnos-telegram) | 1.1.0 | all（x86 + arm64） | 8970 | Telegram Web 客户端（tweb / Telegram Web K 前端）的 fnOS 原生打包：HTTPS 自签证书托管，零 pip 依赖；内置 NAS 出口转发（MTProto 中继），局域网设备无需各自配置代理。首次打开需信任一次自签证书 |
| [米游签 MiyoQian](https://github.com/Marchen-orz/MiyoQian) | 1.0.2 | all（x86 + arm64） | 8966 | 米游社每日签到工具：扫码登录、游戏社区签到、云游戏签到、米游币任务、商品兑换，带 Web 控制台 |
| [Compare Share](https://github.com/Kasbuky-sudo/CompareShare) | 1.2.5 | all（x86 + arm64） | 11011 / 53317 | 局域网极速互传，基于 LocalSend v2.2 协议：与手机、电脑、平板直接互传文件，无需云端与账号；多播 + 网段扫描双通道发现，纯 Python 实现，一个包通吃双架构。目录授权需 fnOS 1.2.0604 及以上 |
| [WorkBuddy](https://github.com/Kasbuky-sudo/FnDepot/tree/main/WorkBuddy) | 5.6.2.3 | x86 + arm64 | 28471 | 腾讯 WorkBuddy 的 fnOS 原生版（非 Docker）：Agent 对话、文件 / 终端操作、办公文档、专家技能与 MCP/Skills；登录态和工作区保存在 NAS；依赖应用中心 Node.js v22；同时支持 x86_64 与 arm64（两个独立包）；5.6.2.3 修复 sheetagent MCP 与沙盒引擎 |


## 免责声明

本源仅供学习与个人使用。外部源不对外部应用代码、安装包安全性或运行稳定性做审核、担保或背书；使用自动签到工具存在账号风控风险，请低频、保守使用，风险自担。
