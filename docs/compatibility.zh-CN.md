[DoDo App](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](compatibility.md) · [Русский](compatibility.ru.md) · [فارسی](compatibility.fa.md) · 简体中文

# 平台兼容性与功能状态

DoDo 的各平台不会同时获得所有功能。成功导入、实际 VPN 连接成功和应用商店发布是三个独立的检查结果。

下表以 **iOS 2.2.0** 和 **Android APK 2.2.0 (2107)** 为功能基准。**2026 年 11 月 11 日**检查的 App Store 页面显示 **iOS 2.2.1**，于 11 月 9 日发布，最低要求为 iOS 15。其更新说明包含密钥添加/更新以及部分完整 JSON 导入改进，不代表所有更新的服务商集成功能已经发布。

## 已准备的 iOS 2.2.2（113）

构建已于 2026 年 11 月 12 日上传至 App Store Connect 以供处理。**尚未提交 App Review，也尚未宣布在 App Store 上线。**下方基准功能表不代表此次更新已正式发布。

| iOS 2.2.2 改进 | 支持边界 |
| --- | --- |
| 完整 JSON 的 `env` / `version` | 保留字段，由核心检查版本要求；受保护的应用变量会被拒绝 |
| HTTP GET / HEAD | 共用连接准备流程、所选代理/均衡器、ENV 隔离及 Hysteria2 地理数据修复；不重连活动 VPN |
| DNS、split DoH 和别名 | 支持兼容方案和有限别名链，检测循环；不提供任意 DNS 的通用转换 |
| FakeDNS | 明确配置、使用标准地址池的兼容方案；不支持其单独 HTTP 检查，也不支持 Lua |
| FinalMask / 分片 | 兼容 URI 和真实成功的 HTTP 响应头；完整 JSON 优先，不自动叠加掩码 |

[构建详情与限制](ios-2.2.2.zh-CN.md)。Android APK **2.2.0（2107）**及 macOS / Windows 发布状态不变，TLS 验证不会降低。

## 平台

| 平台 | 发布与文档范围 |
| --- | --- |
| iPhone / iPad | App Store 版本，支持 iOS 15+。部分系统功能需要更新的系统或兼容硬件。当前版本请以商店为准。 |
| Android | 本文记录的可下载 APK 为 2.2.0 (2107)。Google Play 发布是独立步骤。从 Google Play 安装的应用请通过商店更新，签名可能不同。不要为了强行安装 APK 删除密钥或应用数据。 |
| macOS | 已发布的 Mac 版本与新的原生 Mac 版本功能不同。原生设计和核心更新单独准备，不能将未发布功能视为 Mac App Store 已有功能。 |
| Windows | 准备中，目前没有可承诺功能对等的公开版本。 |

下载链接和发布公告见[网站](https://dodoproxy.app/)。报告问题时，请提供实际安装的版本。

## 导入、路由与日常使用

「有限支持」指文档所列的格式和限制，并非支持所有可能的配置。

| 功能 | iOS 基准版本 | Android APK 基准版本 | macOS | Windows |
| --- | --- | --- | --- | --- |
| URI / Base64 / 完整 Xray JSON | 可用，需要配置校验 | 可用，保留完整关联配置 | 需检查已安装版本和配置 | 准备中 |
| 完整 JSON 路由与负载均衡 | 支持兼容配置，不拆成独立出站条目 | 支持已识别的主协议和依赖 | 新原生实现单独发布 | 准备中 |
| DoDo crypt1 / 面向 DoDo 的 AGE1 | 可用 | 可用剪贴板和手动输入；部分 HUAWEI 对密集 crypt1 QR 的识别仍不可靠 | 尚未确认新版 Mac 移植已发布 | 已计划 |
| `fallback-url` | 从成功响应提前取得备用地址后可用 | 可用，必须提前取得备用地址 | 已记录的已发布 Mac 基准版本不可用 | 准备中 |
| 密钥元数据与 `profile-update-interval` | 可用 | 可用 | 检查已安装版本 | 准备中 |
| Routing / autorouting / 内置地理数据 | 可用，有格式与检查间隔限制 | 可用，检查间隔不同 | 不应假定与移动端相同 | 准备中 |
| TCP、DoDo Ping、HTTP GET、HTTP HEAD、ICMP | 可用 | 可用，检查方式取决于配置 | 检查已安装版本 | 准备中 |
| MTU 与 IPv4 / IPv6 偏好 | 可用 | 可用 | 检查已安装版本 | 准备中 |
| 实时日志、QR 分享、确认后重置 | 可用 | 可用，上述 crypt1 相机限制仍适用 | 检查已安装版本和界面 | 准备中 |
| 经同意的技术错误报告 | 可用，覆盖范围有限 | 可用，覆盖范围有限 | 移动端支持不代表 Mac 已支持 | 准备中 |

面向所有客户启用功能前，请阅读[集成](integration.zh-CN.md)、[协议](protocols.zh-CN.md)、[路由](routing.zh-CN.md)、[加密链接](crypto.zh-CN.md)、[备用地址](fallback.zh-CN.md)和[诊断](diagnostics.zh-CN.md)。

## 如何理解延迟数值

| 方法 | 结果含义 |
| --- | --- |
| TCP | 实测连接建立时间，不除以 3.3。端口开放不能证明 VPN 认证成功或能够访问互联网。 |
| DoDo Ping | 估算值：通过受支持配置的 HTTP GET 时间除以 3.3，并非原始 HTTP 时间。 |
| HTTP GET / HEAD | 通过配置进行请求的实测时间，不进行除法。不是带宽测速。 |
| ICMP | Echo 响应时间；服务商可能阻止 ICMP，但 VPN 仍然可用。 |

Hysteria 使用 UDP，不适合普通 TCP 检查；支持时会通过配置进行 HTTP 检查。Android 的 TCP 模式对于负载均衡和复杂或不明确的 JSON，也通过完整配置检查，而不是随意选取一个出站。检查数量受限，不会重连当前 VPN。

## 协议与操作系统限制

- Android 保留 AmneziaWG，**iOS 不支持**。它不会转换为普通 WireGuard，也不应假定新的原生 Mac 版本包含它。
- 普通 WireGuard 仅支持文档所列的 URI/配置形式，不代表支持任意 peer 布局或 Amnezia 扩展。
- 完整 JSON 支持并不代表手机可运行任意 Xray 服务器配置。服务入站和启动设置可能需要适配；协议、传输、核心版本和地理数据分类仍然重要。
- iPhone 的 Live Activity / Dynamic Island 需要 iOS 16.1+、兼容设备、相应权限以及启用 DoDo 设置。支持 iOS 15 不代表包含这些系统功能。
- QR 扫描需要相机权限。对于较长的加密链接，手动输入和剪贴板是有用的替代方式。
- 本集成不支持 Happ crypto/深度链接、Lite Mode、通用 Premium API、服务商强制主题/按钮样式以及 No Limit Mode。

## 服务商功能：就绪状态与边界

| 功能 | 状态与限制 |
| --- | --- |
| 普通密钥分发 | 无需 DoDo 账户、管理面板私密 API 令牌或模块；HTTPS 和受支持的响应格式即可。 |
| Google 注册与精确主机 DNS TXT 验证 | 免费早期访问。DoDo 审核与域名控制验证是不同步骤。 |
| 服务商自托管控制台 | 早期访问。它是权限受限的 DoDo API 客户端，不是中央管理后台副本，不能访问其他服务商数据。 |
| 服务商品牌与公开动态 | 集成处于早期访问，需要审核通过、精确 host 的有效 DNS 验证、成功响应中的 `x-dodo-provider-id`，以及兼容的新 iOS 客户端。已发布的版本号本身不能证明包含全部后续改动。 |
| 按公开名称/支持联系方式分组的统计 | iOS 和 Android 已有记录，属于安装与使用估计，不是在线人数或精确订阅用户数。 |
| 按 provider ID 隔离的设备报告 | 与名称/支持信息分组不同。iOS 集成正在验证；Android 的 provider-ID 统计**尚未移植**，暂不可承诺完整跨平台报告。 |
| 特定订阅用户的资格证明 | 独立、可选的签名试点。公开 ID 或 DNS TXT 不能证明用户已付款或账户有效。 |
| 向自己的订阅用户推送 | 尚未普遍开放。公开内容绑定不会创建私有推送受众。 |
| 通过服务商 API 自动选择 | 开发中。不要将面板管理 API 令牌放入用户密钥。 |

iOS 暂时隐藏 DoDo「自动选择」按钮，但完整 Xray JSON 中的负载均衡器不依赖它。公开内容与订阅用户授权的区别见[服务商控制台](provider-console.zh-CN.md)。

## 上线前检查

1. 确认平台和已安装构建号，不要只记录应用名 DoDo。
2. 用实际配置、DNS 和传输测试导入、更新及真实连接。
3. 在应用从成功响应取得备用地址后测试备用路径。
4. 为加密链接保留手动/剪贴板路径，并为旧客户端保留兼容方式。
5. 将早期功能视为正式可用前，检查[版本历史与反馈](updates-feedback.zh-CN.md)。

导入失败不应被悄悄变成仅添加部分条目的「成功」。更新失败时保留之前可用的服务器。这能保护数据，但无法修复服务商不可达、凭据错误或服务端路由损坏。

来源：[平台状态](https://dodoproxy.app/docs/#status)、[协议](https://dodoproxy.app/docs/#protocols)、[延迟检查](https://dodoproxy.app/docs/#ping)、[服务商注册](https://dodoproxy.app/docs/#provider-registration)、[统计](https://dodoproxy.app/docs/#analytics)、[App Store](https://apps.apple.com/app/id6759006686)。
