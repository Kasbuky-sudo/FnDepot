# FnDepot · 飞牛第三方应用源

自打包的飞牛 fnOS **第三方原生应用源**（非 Docker），兼容 **x86 与 arm64**。

遵循 [FnDepot 外部应用源 V2 规范](https://github.com/EWEDLCM/FnDepot)，仓库根目录的 `fnpack.json` 即源索引文件。

## 添加本源

在 FnDepot 客户端（≥ 0.0.7）的「外部源」中添加仓库地址：

```
https://github.com/Kasbuky-sudo/FnDepot
```

## 应用列表

| 应用 | 版本 | 架构 | 端口 | 说明 |
| --- | --- | --- | --- | --- |
| [旅行青蛙 · NAS 版](https://github.com/Kasbuky-sudo/frog-nas) | 1.0.3 | all（x86 + arm64） | 8980 | 《旅行青蛙·中国之旅》离线包的 NAS 移植：服务端权威引擎、外部推送（Webhook / 鸿蒙 MeoW）、AI Agent 接口；桌面页内打开，进度保存在应用目录，卸载重装不丢 |
| [网易云音乐 · NAS 版](https://github.com/Kasbuky-sudo/NAS-NEM) | 0.4.1 | all（x86 + arm64） | 8163 | 官方 PC 客户端的 NAS 移植：完整播放器（推荐 / 歌单 / 排行榜 / 歌词 / 搜索）、手机扫码登录、在线播放、歌曲下载到 NAS、登录态跨设备共享、AI 找歌下载（Agent API）；依赖应用中心 nodejs_v22，页内打开 |
| [Telegram](https://github.com/Kasbuky-sudo/fnos-telegram) | 1.1.0 | all（x86 + arm64） | 8970 | Telegram Web 客户端（tweb / Telegram Web K 前端）的 fnOS 原生打包：HTTPS 自签证书托管，零 pip 依赖；内置 NAS 出口转发（MTProto 中继），局域网设备无需各自配置代理。首次打开需信任一次自签证书 |
| [米游签 MiyoQian](https://github.com/Marchen-orz/MiyoQian) | 1.0.1 | all（x86 + arm64） | 8966 | 米游社每日签到工具：扫码登录、游戏社区签到、云游戏签到、米游币任务、商品兑换，带 Web 控制台 |
| [Compare Share](https://github.com/Kasbuky-sudo/CompareShare) | 1.2.2 | all（x86 + arm64） | 11011 / 53317 | 局域网极速互传，基于 LocalSend v2.2 协议：与手机、电脑、平板直接互传文件，无需云端与账号；多播 + 网段扫描双通道发现，纯 Python 实现，一个包通吃双架构。目录授权需 fnOS 1.2.0604 及以上 |

## 打包说明

- 非 Docker 原生应用：内置 uv 0.7.3（x86_64 / aarch64 musl 静态二进制）与 cp311 双架构离线 wheel，安装时按设备架构（`TRIM_SYS_ARCH`）自动选择，**无需联网下载依赖**。
- **旅行青蛙 · NAS 版**依赖应用中心商店的 Node.js v22（`nodejs_v22`），安装时会作为前置依赖一并装上（manifest 里声明了 `install_dep_apps`）；应用本体只含纯 JavaScript 与游戏运行时资源，包内原生模块数量为 0。它的 `.fpk` 有 237MB，**超过 GitHub 单文件 100MB 上限**，因此和米游签一样走 Release 资产分发，不进版本库。
- **网易云音乐 · NAS 版**同样依赖应用中心商店的 Node.js v22（`nodejs_v22`，manifest 里声明了 `install_dep_apps`）；应用本体只含纯 JavaScript，包内原生模块数量为 0，一个包 37MB。它的 `.fpk` 同样走 Release 资产分发，不进版本库（tag: `NETEASE_CLOUD_MUSIC-v0.4.1`）。
- **Compare Share** 的目录授权功能需要飞牛系统 **1.2.0604 及以上**：低版本系统不会把目录授权下发给第三方应用，实测 1.2.0302 不支持。不授权的功能不受影响，仍可使用默认收件目录。
- **Compare Share** 是纯 Python 标准库实现，不依赖任何第三方包与预编译二进制，因此一个包直接通吃 x86_64 / arm64；依赖系统 Python 3（飞牛 fnOS 基于 Debian 12，默认满足）。它同时监听 53317（LocalSend 协议，HTTPS）与 11011（Web 界面），包体仅约 112 KB，配置与证书存放在应用配置目录，重装后设备指纹保持不变。
- **Telegram** 基于 [tweb](https://github.com/morethanwords/tweb)（GPL-3.0，Telegram 官方 Web K 前端）构建：Python3 标准库静态托管 + HTTPS 自签证书（openssl 不可用时用 cryptography 包兜底生成，证书存应用配置目录，升级不重生成）。首版注意：tweb 在纯 HTTP 下崩溃（`caches is not defined`，CacheStorage/WebCrypto 需 secure context），故服务端强制 HTTPS；ServiceWorker 对自签证书的校验独立于浏览器例外（必失败），安装时通过注入脚本自动跳过（`?noServiceWorker=1`）。出口转发仅中继 Telegram 官方域名（白名单），不缓存任何聊天数据。43.5MB，走 Release 资产分发（tag: `v1.1.0`，仓库 `Kasbuky-sudo/fnos-telegram`）。
- 米游签依赖系统 Python 3.11（飞牛 fnOS 基于 Debian 12，默认满足）。
- 应用以专用包用户（`run-as: package`）运行；配置、凭证与日志保存在应用配置目录（`@appconf`），升级与重装不丢失。
- 修改自上游项目 [Marchen-orz/MiyoQian](https://github.com/Marchen-orz/MiyoQian)，应用本体版权归原作者所有。

## 免责声明

本源仅供学习与个人使用。外部源不对外部应用代码、安装包安全性或运行稳定性做审核、担保或背书；使用自动签到工具存在账号风控风险，请低频、保守使用，风险自担。
