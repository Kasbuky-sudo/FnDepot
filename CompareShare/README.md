# Compare Share

飞牛 fnOS 上的局域网极速互传应用，基于 [LocalSend 协议](https://github.com/localsend/protocol) v2.2 实现。

手机、电脑、平板装个 LocalSend 就能和 NAS 直接互传文件，**不经过云端、不需要账号、不依赖外网**。
飞牛原生打包（非 Docker），纯 Python 标准库实现，**一个包同时支持 x86_64 与 arm64**。

- **开发者**：[LocalSend](https://github.com/localsend/localsend) 与 [Kasbuky](https://github.com/Kasbuky-sudo)
- **发布者**：[Kasbuky](https://github.com/Kasbuky-sudo)
- 端口 **11011**（Web 管理界面）与 **53317**（LocalSend 协议，HTTPS），桌面图标页内（iframe）打开

## 功能

- **局域网设备发现** —— 多播广播 + 网段扫描双通道。家用路由器常过滤无线到有线的多播，
  扫描作为兜底，确保手机总能被发现。
- **双向收发** —— 既能接收手机发来的文件，也能把 NAS 上的文件发出去。
- **飞牛目录授权** —— 目录选择走飞牛官方 SDK，授权由系统完成；
  应用只回读系统下发的授权结果，不自行提升权限。
- **传输安全** —— HTTPS + 自签证书指纹固定（与官方实现一致），可选 PIN 码校验（含失败限流）。

## 端口

| 用途 | 端口 |
| --- | --- |
| LocalSend 协议（HTTPS） | 53317 |
| Web 管理界面（HTTP） | 11011 |

## 目录授权

应用以独立的包用户身份运行，访问个人目录需要在飞牛系统中授权。授权方式取决于系统版本：

- **较新版本**：应用设置页点「选择并授权目录」，直接唤起系统原生选择器。
- **较旧版本**：进入 **应用中心 → 已安装 → Compare Share → 应用设置 → 访问权限 → 选择允许访问的文件夹**。
  授权后需重启应用（授权信息由系统在启动时注入）。

默认收件目录为 `/vol1/@appshare/CompareShare/inbox`，可在设置页改为任意已授权目录。

## 链接

- 源码与问题反馈：<https://github.com/Kasbuky-sudo/CompareShare>
- 协议规范：<https://github.com/localsend/protocol>
