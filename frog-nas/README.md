# 旅行青蛙 · NAS 版

把《旅行青蛙·中国之旅》的**离线安卓包**移植成一个跑在 NAS 上的网页应用。
游戏逻辑**一行未改**——随包的离线引擎是唯一权威，本项目只做三件事：把引擎搬到服务器上跑、
把浏览器变成它的一个视图、在引擎外面接出推送与 REST。

> 飞牛 fnOS 原生打包版（非 Docker）。NAS / Docker 移植：**Kasbuky** ·
> 游戏著作权归 **Hit-Point Co., Ltd.** · 离线版由 **Balticx** 制作

## 装完长什么样

在飞牛桌面点「旅行青蛙」图标，游戏**直接嵌在桌面窗口里打开**（页内，不跳新标签页）。
首次进入会看到原版的权利归属声明，点「我已阅读，进入游戏」开始。

| 地址 | 用途 |
|---|---|
| `http://<NAS_IP>:8980` | 游戏本体 |
| `http://<NAS_IP>:8980/admin` | 设置页：推送、API token、引擎时长、存档 |
| `http://<NAS_IP>:8980/api/health` | 健康检查（免认证） |
| `http://<NAS_IP>:8980/api/openapi.json` | 接口文档 |

进度存在 `/vol1/@appdata/frog-nas/`，**卸载重装不会丢**。

## 两个原版没有的能力

### 外部消息推送

支持两个通道，可同时开、可分别订阅事件。

- **自定义 Webhook**：填 URL 即可，可自定义请求头（JSON）与 body 模板，占位符
  `{{event}} {{title}} {{body}} {{timestamp}} {{url}} {{image}} {{data}} {{dataJson}}`。
- **鸿蒙 MeoW**：用**昵称**作为收件标识，没有 token，填对昵称就行。
  判据是响应体里的 `status === 200`（服务端会用 HTTP 200 包着"昵称不存在"这类业务错误，
  只看 HTTP 状态码会误判成功）。

可订阅：`depart` 出发、`postcard` 明信片、`return` 回家（附带回清单）、
`visitor_arrive` 访客到访、`visitor_gift` 访客回礼、`mail` 新邮件、`lottery` 券够了、
`title_unlock` 新称号、`furniture_finish` 家具完成。默认开前五个。

免打扰默认 23:00–07:00：**期间的事件排队，窗口结束后补发，不丢**。
失败重试 3 次，指数退避 2s/4s/8s。发送记录在 `data/logs/push.jsonl`，设置页可看最近 100 条。

推送里的 `image` 指向 `/asset/postcard/<picId>`，由服务端用游戏自己的美术现场合成
500×350 的 PNG（与客户端 `drawToTexture` 同尺寸）。这个地址不带鉴权，因为
MeoW / webhook 接收方要自己来取图。

### AI Agent 操作接口

带 5 个自包含的 `SKILL.md`，agent 知道地址就能来玩：

| 技能 | 何时用 |
|---|---|
| `frog-status` | **每次操作前先查**：蛙在哪、草熟没熟、券够不够、有没有客人 |
| `frog-harvest` | 收三叶草（先查总览判断有没有熟的） |
| `frog-prepare` | 备行李：查库存 → 缺便当先买 → 装背包 → 可选放桌 |
| `frog-visitor` | 发现访客并按口味投喂、追踪回礼 |
| `frog-lottery` | 券够 5 张则抽，否则说明券的来源 |

```bash
export FROG_API_BASE="http://<NAS_IP>:8980"   # 只需要这一个
curl -s http://<NAS_IP>:8980/api/skills       # 技能清单
```

**默认不需要 API token**。只有当 8980 可能被不信任的设备访问时，才去 `/admin`
打开「要求 token」。

## 鸿蒙版飞牛 App

以前在鸿蒙版飞牛 App 里点开图标，游戏可能起不来：桌面页把游戏指向 `fnconnect` 域名下的
`:8980`，整局游戏的资源都得走中继，限速链路上抢不到带宽就卡在加载。

**飞牛 App 1.37.0 起已修复，可以正常打开游戏了。** 如果你的 App 还停在更早的版本，
升级到 1.37.0 或更高再试；仍然打不开时，用手机浏览器直接开 `http://<NAS_IP>:8980`
（内网直连）可以确认是服务端还是链路问题。

## 关于「服务端权威」

引擎只在 NAS 上跑一份，手机和电脑同时打开看到的是**同一个世界**：一个标签页收草，
另一个 30 秒内也会变。关掉浏览器蛙也照常出门、回家、收邮件——这正是推送有意义的前提。

默认 `FROG_FAITHFUL=1`，按 `define.json` 里恢复的原版服务端数值旅行（小时级）；
想在一次坐下里玩完，去设置页把它改成 `0`。

## 双架构

一个包同时支持 x86_64 与 arm64（`platform = all`）。这不是"看起来应该可以"，
而是对产物逐条扫过的：

- 包内 `*.node` / `*.so` / `*.dll` / `*.dylib` / `*.exe`：**0 个**
- `prebuilds/` 或 `build/Release/` 目录：**0 个**
- 顶层依赖里带 `gypfile` 或 `install`/`postinstall` 脚本的：**0 个**
- 对 `node_modules/` 下所有文件按文件头判 ELF / PE / Mach-O：**0 个命中**

运行时依赖只有 `express` 与 `ws`，两个都是纯 JavaScript，运行时由应用中心的
`nodejs_v22` 提供。x86NAS 与 armNAS 均已完成真机安装验收。
