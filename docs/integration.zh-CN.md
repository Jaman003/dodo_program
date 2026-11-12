[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](integration.md) · [Русский](integration.ru.md) · [فارسی](integration.fa.md) · [简体中文](integration.zh-CN.md)

# 服务商集成

向用户提供 HTTPS 配置链接和[“在 DoDo 中打开”](deep-links.zh-CN.md)按钮即可。普通导入和 VPN 连接不需要 DoDo 账户、API 令牌或指定面板。Remnawave、3x-ui 及自建服务都可以返回支持的格式。

本文以 iOS 2.2.0 和 Android APK 2.2.0（2107）的功能为基准。平台差异与发布状态见[兼容性表](compatibility.zh-CN.md)。

**已准备的 iOS 2.2.2（113）：**完整 JSON 保留根级 `env` 和 `version`，由核心在启动前检查 `version.min` / `version.max`。兼容 URI 参数和真实成功的 HTTP 响应头可提供 FinalMask / 分片设置。构建已上传至 App Store Connect 以供处理，尚未提交 App Review，也尚未宣布在商店上线。[参数、优先级与限制](ios-2.2.2.zh-CN.md)。这些改动不会自动应用于 Android。

## 1. 返回正确的配置

iOS 使用 `User-Agent: DoDoApp`。Android 使用 `DoDoApp/` 前缀，例如 `DoDoApp/2.2.0+2107 (Android 15; Demo Model)`。仅精确匹配 `DoDoApp` 的规则无法覆盖 Android。

返回 HTTP 200 和完整的受支持内容。JSON 应使用 `application/json`。以下是不可用于连接的纯文本示例：

```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
profile-title: Demo Provider
profile-update-interval: 1
support-url: https://example.invalid/support

vless://00000000-0000-4000-8000-000000000001@example.invalid:443?security=tls&type=ws&path=%2Fdemo#Demo
```

| 响应内容 | DoDo 的处理方式 |
| --- | --- |
| URI 列表 | 独立服务器，通常每行一个 |
| Base64 | 支持的编码列表或 JSON 容器；Base64 不是加密 |
| 完整 Xray JSON | 保留 DNS、路由、负载均衡器和出站依赖，作为一个配置 |
| 完整 JSON 配置数组 | 每个独立配置对应一个选项 |
| 独立 outbounds 集合 | 展示为多个服务器，不凭空添加负载均衡器 |

不要把级联或负载均衡配置拆成独立链接。DoDo 保留兼容的标签、依赖、`observatory`、`burstObservatory`、`policy` 和 `stats`。内部入站、端口等可能在运行时调整；实际能力仍取决于内置核心。

无效或不支持的配置会被拒绝，而不是静默简化。更新失败时保留原服务器列表。匹配服务器的 ID、顺序、置顶、选择及最后一次延迟值会保留。

### 临时兼容 Happ JSON 响应

DoDo 先请求原始 URL。如果已收到完整可用的 JSON，就不会额外请求；否则可能使用 `User-Agent: Happ/DoDoApp-compat` 重试**同一 URL**，并用 `X-DoDo-Client` 标明真实客户端。不同响应不会合并。

建议添加 DoDo 响应规则，或由面板提供明确的完整格式 URL。应用不会给任意链接自动添加 `/json`。兼容请求失败时仍保留可用的主要响应；已保存的完整配置不会降级成不完整的 URI 列表。这不代表支持 Happ 深层链接或其私有加密格式。

## 2. 添加卡片信息

使用 HTTP 响应头传递这些字段；字段名不区分大小写。

| 响应头 | 值与效果 |
| --- | --- |
| `profile-title` | 服务名称，纯文本或 `base64:` UTF-8；缺省依次使用 `subscription-name`、`Content-Disposition` 的 filename / filename* |
| `x-dodo-provider-logo-url` | 服务徽标的公开 HTTPS 图片地址；不更改应用图标 |
| `support-url` | 支持页面或机器人；存在时显示支持按钮 |
| `profile-web-page-url` | 账户、机器人或续期页面；别名 `homepage`；付款在服务商网站完成 |
| `subscription-userinfo` | `upload=…; download=…; total=…; expire=…`；流量单位为字节，到期为 Unix 秒 |
| `profile-update-interval` | 1–168 的整数，单位为**小时**，默认 1 小时；后台运行时间由操作系统安排 |
| `announce` | 卡片公告的文本或支持的 `base64:` 内容；不是推送或动态 |
| `routing` / `autorouting` | 静态或远程更新的[路由规则](routing.zh-CN.md) |
| `fallback-url` / `providerid` | 预先提供的[备用地址](fallback.zh-CN.md)及其匹配标识 |
| `x-dodo-provider-id` | 用于独立[服务商控制台集成](provider-console.zh-CN.md)的公开 UUID |

普通元数据优先读取真实响应头，然后读取受支持的正文注释、解码后的 Base64 和 JSON 元数据，使用首个有效值。静态文件可使用 `#profile-title: Demo Provider` 等受支持注释。

**例外：**`x-dodo-provider-id` 只能来自精确验证主机的成功 HTTPS 响应，不能来自正文注释。它与备用地址的旧 `providerid` 标识不同。两者都不能证明付款或用户对配置的所有权。

不要把用户的秘密令牌放在徽标、支持页面等公开 URL 中。HTTP 403 本身不能定义到期时间。

## 3. 在服务端执行设备限制

请求配置时，DoDo 会发送安装标识和设备信息：

```http
User-Agent: DoDoApp
Accept: */*
x-hwid: 00000000-0000-4000-8000-000000000001
x-device-os: iOS
x-ver-os: 18.5.0
x-device-model: iPhone 14 Pro
```

Android 使用同样的技术请求头名称，但发送自己的机型、系统版本、Android 系统名称及带版本的 User-Agent。HWID 不是序列号或 IDFA，重试不会更换它；重置应用设置也不会重置服务商的设备配额。它只放在请求头，不附加到 URL。

| 服务端响应 | 结果 |
| --- | --- |
| `x-hwid-max-devices-reached: true` | 提示已达到设备上限 |
| `x-hwid-limit: true` | 兼容旧版的相同标志 |
| `x-hwid-not-supported: true` 或明确的 `HWID_REQUIRED` | 显示加载错误，不生成替代 ID |

授权和允许设备数由服务商决定，HWID 不能代替身份验证。跨 origin 重定向时，iOS 移除敏感请求头，Android 拒绝重定向；不允许 HTTPS 降级至 HTTP。

## 限制与上线前检查

| 限制 | iOS | Android APK |
| --- | --- | --- |
| 单次响应 | 8,000,000 字节 | 2 MiB |
| 整个操作 | 12,000,000 字节 | 4 MiB |
| 单个 JSON 配置 | 2,000,000 字节 | 受响应上限约束 |
| AGE1 加密正文 | 2 MiB | 2 MiB |
| 总加载时间 | 25 秒，尝试次数有限 | 25 秒，尝试次数有限 |

加载失败不会关闭 TLS 验证。存在网络错误的残缺响应不会导入。

向用户开放前，请在各目标平台检查导入、更新、配置数量与顺序、到期与流量、设备上限错误和真实连接。测试面板实际输出的传输与认证参数；导入成功或 TCP 延迟正常不代表互联网访问可用。

下一步：[协议](protocols.zh-CN.md) · [深层链接](deep-links.zh-CN.md) · [加密链接](crypto.zh-CN.md) · [诊断](diagnostics.zh-CN.md)

来源：[DoDo 集成说明](https://dodoproxy.app/docs/#quickstart)、[响应头](https://dodoproxy.app/docs/#headers)、[HWID](https://dodoproxy.app/docs/#device-hwid)。
