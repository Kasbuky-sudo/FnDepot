# 网易云音乐 · NAS 版

把网易云音乐官方 PC 客户端跑在飞牛 fnOS 上的**原生应用**（非 Docker）：
服务端解析官方乐库接口，浏览器 / fnOS 桌面里直接用完整播放器 ——
歌单、每日推荐、排行榜、歌词、手机扫码登录、在线播放、**歌曲下载到 NAS**。

- **开发者**：[YUCLing](https://github.com/YUCLing)（上游 [open-orpheus](https://github.com/YUCLing/open-orpheus)，MIT）
- **NAS 移植 / 打包 / 发布**：[Kasbuky-sudo](https://github.com/Kasbuky-sudo)
- 原生 Node.js 运行，复用应用中心 **Node.js v22**（`nodejs_v22`，安装时自动作为前置依赖装上），包内原生模块数量为 0，一个包通吃 x86_64 / arm64
- 端口 **8163**，桌面图标页内（iframe）打开
- 多浏览器会话隔离：每个会话独立登录态 / 曲库 / 缓存

## 功能

- 完整官方前端：首页推荐 / 歌单广场 / 排行榜 / 每日推荐 / 播客 / 歌词 / 搜索
- 手机扫码登录（页内浮层窗口，扫码后自动关闭并刷新登录态）
- 在线播放：`audioplayer.*` 完整桥接 + HTML5 `<audio>`，无需浏览器扩展
- 歌曲下载：落地 `/vol1/1000/网易云音乐`，服务端代抓 CDN 音源，支持进度查询 / 暂停 / 取消
- 图标为官方红底白图样式

## 说明

- 歌曲下载需先登录；会员 / 付费歌曲按账号自身权限处理，与官方客户端一致
- 首次构建需准备官方前端资源，详见源码仓库 README

## 链接

- 源码与问题反馈：<https://github.com/Kasbuky-sudo/NAS-NEM>
