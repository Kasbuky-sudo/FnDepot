# Telegram for fnOS

飞牛 fnOS 原生打包的 **Telegram Web 客户端**（非 Docker），基于开源的
[tweb](https://github.com/morethanwords/tweb)（Telegram Web K，官方
web.telegram.org/k/ 同款前端）。

一个安装包同时支持 x86 与 arm64。内置 **NAS 出口转发**：局域网内所有设备
通过 NAS 访问 Telegram，各设备无需自行配置代理。

> 开发者：[Telegram / tweb 上游](https://github.com/morethanwords/tweb) ·
> 发布者：[Kasbuky](https://github.com/Kasbuky-sudo)

## 特性

- **非 Docker 原生应用**：fnOS 应用中心手动安装或经 FnDepot 应用源安装，桌面面板一键打开；
- **零依赖**：Python3 标准库静态托管（fnOS 基于 Debian 12 自带 python3），无 pip 依赖；
- **双架构**：`platform = all`，x86 与 arm64 同一安装包；
- **NAS 出口转发**：MTProto（WebSocket + HTTPS 双通道，域名白名单限制）经 NAS 中继，设备侧零配置；
- **隐私模型**：NAS 只托管静态页面与转发字节，不存储聊天数据；会话保存在浏览器本地，登录与收发由浏览器经加密通道与 Telegram 服务器完成。

## 安装与使用

1. 安装 `Telegram-1.1.0.fpk`；
2. 首次打开会提示自签名证书警告，点「高级 → 继续前往」信任一次即可；
3. **建议通过 HTTPS 访问 fnOS 桌面**；若直接以 HTTP 访问应用端口，页面顶部会给出引导提示；
4. 从 fnOS 桌面面板打开应用，扫码或输入手机号登录。

> 若 NAS 本身无法直连 Telegram，出口转发功能无效——请确保 NAS 具备访问
> Telegram 的网络条件。

## 端口

| 项目 | 值 |
| --- | --- |
| Web 服务端口 | 8970（HTTPS，自签证书） |
| 运行用户 | package（`run-as: package`） |
| 证书位置 | 应用配置目录（`@appconf`，升级不丢失） |

## 免责声明

- 应用本体 [tweb](https://github.com/morethanwords/tweb) 基于 GPL-3.0 开源，
  版权归 Telegram 与上游作者所有；Telegram 是 Telegram Messenger Inc. 的商标，
  本项目与其无隶属关系。
- 本包仅供学习与个人使用，使用外部源安装的应用请自行评估安全性与合规性。
