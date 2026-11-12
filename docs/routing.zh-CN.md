[DoDo App](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](routing.md) · [Русский](routing.ru.md) · [فارسی](routing.fa.md) · 简体中文

# 路由、地理数据与网络设置

路由规则用于指定哪些目标直连、通过代理或被阻止。DoDo 的简化规则格式与完整 Xray 配置不同，请勿将完整 Xray JSON 填入下面的规则字段。

本页以 iOS 2.2.0 和 Android APK 2.2.0 (2107) 的已记录功能为基准。其他版本的支持情况需要单独确认。

**已准备的 iOS 2.2.2（113）**改进兼容 DNS 协调、split DoH 和带循环检测的本地别名。支持明确配置、使用标准地址池的兼容 FakeDNS，不代表支持任意混合 DNS 或 Lua。简化的 `autorouting` 格式并未扩展，下方 DNS/FakeDNS 限制仍适用。[支持边界与 App Store Connect 处理状态](ios-2.2.2.zh-CN.md)；构建尚未提交 App Review，也尚未宣布在商店上线。

## 静态规则

```json
{
  "Name": "Demo rules",
  "GlobalProxy": true,
  "DomainStrategy": "IPIfNonMatch",
  "DirectSites": ["domain:example.invalid"],
  "DirectIp": [],
  "ProxySites": ["full:proxy.example.invalid"],
  "ProxyIp": [],
  "BlockSites": ["geosite:category-ads-all"],
  "BlockIp": []
}
```

| 字段 | 用途 |
| --- | --- |
| `Name` | 规则配置的显示名称。 |
| `GlobalProxy` | 此规则配置的全局代理设置。 |
| `DomainStrategy` | 域名解析策略。远程格式接受 `AsIs`、`IPIfNonMatch` 和 `IPOnDemand`。 |
| `DirectSites` / `DirectIp` | 直连目标。 |
| `ProxySites` / `ProxyIp` | 通过代理的目标。 |
| `BlockSites` / `BlockIp` | 阻止的目标。 |

域名列表接受域名及受支持的 `domain:`、`full:`、经过校验的 `regexp:` 和 `geosite:` 条目。IP 列表接受 IP 地址、CIDR 网段和 `geoip:` 条目。引用的分类必须存在于可用数据文件中。

### 导入本地规则配置

将 UTF-8 JSON 编码为 Base64 或 Base64URL。链接中建议使用不带末尾 `=` 的 Base64URL；普通 Base64 可以进行一次百分号编码。

```text
dodo://routing/add/{BASE64_UTF8_JSON}
dodo://routing/onadd/{BASE64_UTF8_JSON}
dodo://routing/off
```

`add` 和 `onadd` 都会保存并启用规则。`off` 只停用，不删除。不要加入用户名、密码、端口、查询参数或片段。规则在下次连接时生效，不会自动重连当前 VPN。

从机器人或浏览器打开应用的方法见[深度链接](deep-links.zh-CN.md)。

## 远程规则：`autorouting`

在 HTTP 响应头 `autorouting` 中提供独立规则文件的公开 HTTPS 地址。请使用真实的公网域名；示例中的 `.invalid` 和本地地址不会被接受。

```http
autorouting: https://rules.example.com/routing.json
```

端点必须返回 HTTP 200，以及使用上述字段的 UTF-8 JSON **对象**，不能返回 Base64、routing 链接或完整 Xray JSON。

| 要求 | 接受的内容 |
| --- | --- |
| 结构 | 仅示例字段及可选的字符串 `LastUpdated`；支持对应 camelCase 别名。未知字段和互相冲突的别名将被拒绝。 |
| 值 | 列表必须为字符串数组。`GlobalProxy` 为 JSON 布尔值，或字符串 `"true"` / `"false"`。 |
| 大小 | 单文件不超过 256 KiB，总条目不超过 2048，每条不超过 512 字节。 |
| URL | 使用标准端口的 HTTPS，不含用户信息和片段。重定向次数受限，且只能在同一源内进行。 |
| 请求隐私 | 规则请求不携带订阅 HWID、Cookie 或订阅 API 令牌。 |
| 失败处理 | 网络错误或 JSON 无效时保留上一份可用规则。 |
| 作用范围 | 每个密钥配置单独缓存，新规则在下次连接时生效。 |

| 基准平台 | 成功来源的再次检查间隔 | 请求超时 |
| --- | --- | --- |
| iOS 2.2.0 | 最多每小时一次；失败后延迟重试 | 最长 12 秒 |
| Android APK 2.2.0 | 24 小时 | 8 秒 |

后台执行由操作系统安排，因此不保证精确的后台时刻。此间隔与更新密钥本身的 `profile-update-interval` 无关。

订阅响应正文也可包含带 HTTPS 来源的 `dodo://autorouting/add/…` 或 `dodo://autorouting/onadd/…`。这些是响应正文内的命令，不是用于从浏览器或机器人启动应用的链接。

此简化格式**不支持** DNS/FakeDNS、任意 `Geoipurl` / `Geositeurl` 来源和 `routeOrder`。更复杂的结构请使用兼容的完整 Xray 配置。

## 完整 Xray 配置与负载均衡器

DoDo 会保留受支持完整配置中的关联出站、DNS、路由、负载均衡器和传输设置。一个负载均衡配置不应被拆成多个互不关联的服务器条目。

导入校验不是服务器配置修复工具。不要为了导入而删除 `fallbackTag`、把它改为 `direct` 或移除负载均衡器。不存在的备用目标是否被接受，取决于客户端与核心的组合；能够加载不代表备用路径一定可用。服务商应检查出站标签和实际故障切换行为。

暂时隐藏的 DoDo「自动选择」按钮、Xray JSON 内的负载均衡器，以及未来的服务商 API Auto 服务是三种不同功能。隐藏按钮不会禁用完整配置中的负载均衡器。API Auto 尚未面向所有服务商开放。

参见[格式与集成](integration.zh-CN.md)、[协议](protocols.zh-CN.md)和[兼容性](compatibility.zh-CN.md)。

## 内置 `geoip.dat` 和 `geosite.dat`

上述 iOS 和 Android 基准版本均内置这两个文件，首次添加密钥时无需随订阅下载。更新来源为 [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)。

安装前会检查 SHA-256、文件格式和所需分类。验证失败时保留上一对可用文件。其他数据集合中的分类可能不存在；不支持从服务商 URL 加载任意地理数据文件。

在 iOS 中，可前往 **设置 → 高级用户 → 地理数据** 查看版本或手动检查。自动检查依赖系统允许运行的时间和来源可达性。Android 中，已连接的 VPN 会继续使用本次选定的文件，更新在下次连接时生效，并支持回退至可用版本。

这些文件包含路由分类，不是 DoDo 用户地图，也不是服务商统计的数据来源。

## DNS、MTU 与 IP 偏好

没有明确网络问题时，请保留默认值。修改在下次连接生效，不会改写已保存的密钥。

| 设置 | 含义与限制 |
| --- | --- |
| 隧道 MTU | 默认/Auto、1420 / 1380 / 1280 预设，或 1280 至 1500 字节的整数。它是数据包大小，不是速度上限。DoDo 不会自动寻找最佳 MTU。 |
| IP 偏好 | 对以域名连接的 VPN 服务器选择「按配置」「优先 IPv4」或「优先 IPv6」。这是偏好，不会关闭设备的 IPv6。复杂配置可能继续使用服务商的行为。 |
| 配置中的 DNS | 在可安全适配时，iOS 会协调系统隧道 DNS 与 Xray DNS。含糊的 DNS 结构、FakeDNS 和系统 DNS 依赖不会被强行改写。Android 同样保留服务商 DNS，但隧道实现不同。 |
| VPN 资源 | iOS 的高级页面可显示扩展内存、Go heap、连接和文件描述符，需要开启诊断日志并建立新会话。缺少数据不会被显示为零。 |

对于 IPv6-only、DNS64 或 464XLAT 网络，固定为 IPv4-only 并非通用修复方法。请测试实际网络和服务商配置；IP 偏好或较小 MTU 都不能保证兼容。

本基准版本不提供手动 Xray 内存上限滑块、「No Limit Mode」或全部核心参数的通用编辑器。

## 相关页面

[备用地址](fallback.zh-CN.md) · [加密链接](crypto.zh-CN.md) · [服务商控制台](provider-console.zh-CN.md) · [诊断](diagnostics.zh-CN.md) · [更新与反馈](updates-feedback.zh-CN.md)

来源：[路由](https://dodoproxy.app/docs/#routing)、[地理数据](https://dodoproxy.app/docs/#geodata)、[网络设置](https://dodoproxy.app/docs/#network)、[平台状态](https://dodoproxy.app/docs/#status)。
