<div align="center">

<img src="../media/dodo-astronaut.png" width="96" alt="DoDo 宇航员" />

# DoDo APP · 文档

**接入您的服务，保留完整配置。**

[English](README.md) · [Русский](README.ru.md) · [فارسی](README.fa.md) · [简体中文](README.zh-CN.md)

[产品与下载](../README.zh-CN.md) · [官方网站](https://dodoproxy.app/) · [社区](https://t.me/dododeveloper)

</div>

## 从这里开始

DoDo 是基于 Xray 的客户端，用于服务商或自建服务器提供的密钥和配置。应用不附带免费服务器，也不出售 VPN 访问。普通导入和连接无需服务商控制台账户。

用户：通过[官方下载链接](../README.zh-CN.md)安装，用剪贴板、手动输入或 QR 扫描添加支持的密钥，选择服务器并允许系统 VPN 请求。遇到问题请先看[诊断指南](diagnostics.zh-CN.md)。不要公开个人配置 URL 或未脱敏日志。

服务商：先阅读[集成指南](integration.zh-CN.md)，添加[“在 DoDo 中打开”按钮](deep-links.zh-CN.md)，并在每个目标平台测试真实连接。

## 指南目录

| 指南 | 内容 |
| --- | --- |
| [服务商集成](integration.zh-CN.md) | 响应格式、完整 JSON、响应头、自动更新间隔与 HWID |
| [协议与传输](protocols.zh-CN.md) | VLESS、VMess、Trojan、Shadowsocks、Hysteria2、SOCKS5、WireGuard、TLS / REALITY 及限制 |
| [深层链接](deep-links.zh-CN.md) | 机器人与网站按钮、URI 编码、支持命令及常见错误 |
| [加密链接](crypto.zh-CN.md) | 离线 `dodoapp://crypt1/`、加密工具、公钥与 AGE1 响应 |
| [备用地址](fallback.zh-CN.md) | `fallback-url`、`providerid`、重试与安全匹配 |
| [路由与地理数据](routing.zh-CN.md) | Direct / Proxy / Block、`autorouting`、GeoIP / GeoSite、DNS、MTU 和 IP 模式 |
| [服务商控制台](provider-console.zh-CN.md) | Google 登录、DNS TXT、provider ID、品牌、动态及自托管控制台 |
| [延迟与诊断](diagnostics.zh-CN.md) | 各模式测量内容、实时日志、错误报告与数据控制 |
| [平台兼容性](compatibility.zh-CN.md) | iOS、Android、macOS、Windows 上已提供、受限和未完成的功能 |
| [更新与反馈](updates-feedback.zh-CN.md) | 版本历史、建议、故障报告及审核 |

## 状态说明

- **已提供：**已在指定平台和基准版本实现，不保证所有 Xray 配置均可运行。
- **受限 / 抢先体验：**存在格式、设备或发布范围限制。
- **开发中 / 暂不可用：**尚不应向用户承诺可用。

除非另有说明，功能表以 iOS 2.2.0 和 Android APK 2.2.0（2107）为基准。App Store 已于 **2026 年 11 月 9 日**发布 iOS **2.2.1**，但版本号本身不能证明新的控制台功能已部署。请参阅[兼容性](compatibility.zh-CN.md)和[版本历史](updates-feedback.zh-CN.md)。

公开 UUID `x-dodo-provider-id` 与备用地址使用的旧 `providerid` 标识不同，两者都不能证明付费资格。公开品牌和动态、私有受众访问、按 provider ID 划分的设备统计具有不同要求与发布状态。

## 官方链接

[网站文档](https://dodoproxy.app/docs/) · [服务商介绍](https://dodoproxy.app/providers/) · [版本历史](https://dodoproxy.app/updates/) · [反馈](https://dodoproxy.app/feedback/) · [Telegram 社区](https://t.me/dododeveloper) · [Telegram 频道](https://t.me/dodoproxy)

此仓库仅包含公开文档和图片，不发布应用源代码、签名文件或私有配置密钥。

---

**by Jaman Developer**
