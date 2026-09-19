# Compare Share

飞牛 fnOS 上的局域网极速互传应用，基于 [LocalSend 协议](https://github.com/localsend/protocol) v2.2 实现。

手机、电脑、平板装个 LocalSend 就能和 NAS 直接互传文件，**不经过云端、不需要账号、不依赖外网**。应用以飞牛原生 fpk 形式安装，一个包同时支持 x86 与 arm64。

> 开发者：[LocalSend](https://github.com/localsend/localsend) 与 [Kasbuky](https://github.com/Kasbuky-sudo) ·
> 发布者：[Kasbuky](https://github.com/Kasbuky-sudo)

## 功能

- **局域网设备发现** —— 多播广播 + 网段扫描双通道。家用路由器常过滤无线到有线的多播，扫描作为兜底，确保手机总能被发现。
- **双向收发** —— 既能接收手机发来的文件，也能把 NAS 上的文件发出去。
- **浏览器上传页** —— 手机不装任何应用，浏览器打开 `http://<NAS>:11011/upload` 即可传文件；设置页可显示二维码供扫码直达。
- **飞牛目录授权** —— 目录选择走飞牛官方 SDK，授权由系统完成；应用只回读系统下发的授权结果，不自行提升权限。
- **Web 管理界面** —— 设备列表、收件箱、传输记录、设置四块，深浅色自适应，在飞牛桌面中以 iframe 打开。
- **传输安全** —— HTTPS + 自签证书指纹固定（与官方实现一致），可选 PIN 码校验（含失败限流）。

## 安装

### 通过 FnDepot 应用源（推荐）

在 FnDepot 客户端的「外部源」中添加：

```
https://github.com/Kasbuky-sudo/FnDepot
```

### 手动安装

从 [Releases](https://github.com/Kasbuky-sudo/CompareShare/releases) 下载 `CompareShare-<版本>.fpk`，
在飞牛「应用中心 → 手动安装」中上传。

安装后应用监听：

| 用途 | 端口 |
| --- | --- |
| LocalSend 协议（HTTPS） | 53317 |
| Web 管理界面（HTTP） | 11011 |

## 浏览器上传（手机免安装）

不想在手机上装应用时，用浏览器直接传：

1. 手机浏览器打开 `http://<NAS_IP>:11011/upload`
2. 选择文件（支持多选、拍照上传），点击发送
3. 文件保存到设置的收件目录

管理界面的「设置 → 浏览器上传」里有二维码，手机扫码即可直达，无需手输地址。
该功能可关闭；关闭后仍可通过「浏览器上传」开关控制，且不影响 LocalSend 客户端使用。

> 上传走与 LocalSend 相同的 `prepare-upload` / `upload` 端点，因此 PIN 校验、
> 体积校验、文件名清洗等保护一并生效。浏览器无法在明文 HTTP 下计算 SHA-256，
> 故省略该字段（协议允许），服务端会跳过校验和比对。

## 目录授权

应用以独立的包用户身份运行，访问个人目录需要在飞牛系统中授权。

> **系统版本要求：fnOS 1.2.0604 及以上**
>
> 目录授权需要飞牛系统支持把授权下发给第三方应用。实测 **1.2.0302 不支持**——
> 即使已在系统设置中勾选了目录，系统也不会把授权传给应用，应用无法访问。
> 应用会检测系统版本并给出提示。在满足要求的系统上：

- **应用内授权**：设置页点「选择并授权目录」，直接唤起系统原生选择器。
- **系统内授权**：**应用中心 → 已安装 → Compare Share → 应用设置 → 访问权限 → 选择允许访问的文件夹**。
  授权后需重启应用（授权信息由系统在启动时注入）。

默认收件目录为 `/vol1/@appshare/CompareShare/inbox`，可在设置页改为任意已授权目录。
保存时会校验目录是否存在、是否可写，不可用会当场提示原因，不会等到收文件时才失败；
已授权的目录在面板中可直接点击填入。

## 从源码构建

需要 [fnpack](https://developer.fnnas.com/)（飞牛官方打包工具）。

```bash
fnpack build -d CompareShare
```

产物为 `CompareShare.fpk`。应用本身是纯 Python 3 实现，只依赖标准库，无需编译，这也是它能同时支持 x86 与 arm64 的原因。

### 目录结构

```
CompareShare/
├── manifest                 # 应用元数据
├── config/
│   ├── privilege            # 运行身份（专用包用户）
│   └── resource             # 共享目录与开放接口权限声明
├── cmd/                     # 生命周期脚本（安装/启动/停止/升级/卸载）
├── app/
│   ├── ui/                  # 飞牛桌面入口配置与图标
│   └── server/
│       ├── main.py          # 入口
│       └── compareshare/
│           ├── protocol.py      # LocalSend v2.2 数据模型与文件名清洗
│           ├── discovery.py     # 多播发现（逐网卡绑定）
│           ├── receiver.py      # 接收端（流式落盘 + 校验）
│           ├── sender.py        # 发送端（证书指纹固定）
│           ├── fnos.py          # 飞牛开放接口客户端（Unix socket）
│           ├── tls.py           # 自签证书与指纹
│           ├── core.py          # 状态聚合与网段扫描
│           ├── http_server.py   # 协议端点 + Web API + 静态资源
│           └── webui/           # 前端（原生 HTML/CSS/JS，内置飞牛 SDK）
└── wizard/                  # 安装向导（当前为空）
```

## 实现要点

几个容易踩坑、文档里不显眼但必须处理的地方：

- **多网卡多播**：每个物理网卡单独建 socket 并绑定出口（`IP_MULTICAST_IF`）。默认路由未必是业务网卡（本机实测默认路由走 Meta tun 口），不显式绑定会广播到错误方向。容器/虚拟接口需要排除。
- **发现要双向**：按协议，收到广播后必须回一个 HTTP `register`，否则对方学不到你的存在——只记录不回复会造成单向可见。
- **TLS 不索要客户端证书**：LocalSend 双方都用自签证书，身份靠指纹固定而非 CA 链。服务端若用 `CERT_OPTIONAL`，Python 会拿系统 CA 校验自签证书并回 `unknown_ca`；TLS 1.3 下客户端证书在握手之后发送，表现为握手正常但一发数据就被拒。
- **上传按剩余字节读取**：socket 上的 `read(n)` 会阻塞到凑满 n 字节，而对端发完就等响应，直接读会死锁。
- **文件名当作不可信输入**：只取最后一段阻断路径穿越，替换非法字符，处理 Windows 保留名，按 UTF-8 字符边界截断到 255 字节。临时文件用独立随机名，避免长文件名加后缀后超出文件系统上限。
- **证书放在配置目录**：指纹是本机在 LocalSend 网络里的身份，放数据目录会在卸载时丢失，导致已配对的设备失效。
- **透传飞牛环境变量**：以专用用户启动应用时 `runuser` 会重置环境，`TRIM_API_TOKEN` 等变量必须显式传递，否则开放接口调用被拒。

## 许可

本项目基于 [Apache-2.0](LICENSE) 发布。

LocalSend 协议由 [localsend/protocol](https://github.com/localsend/protocol) 定义，参照实现 [localsend/localsend](https://github.com/localsend/localsend) 亦为 Apache-2.0。

- **开发者**：[LocalSend](https://github.com/localsend/localsend) 与 [Kasbuky](https://github.com/Kasbuky-sudo)
- **发布者**：[Kasbuky](https://github.com/Kasbuky-sudo)

本应用为独立实现，与 LocalSend 官方无隶属关系。
