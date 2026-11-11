[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](fallback.md) · [Русский](fallback.ru.md) · [فارسی](fallback.fa.md) · [简体中文](fallback.zh-CN.md)

# 使用 `fallback-url` 提供备用源

主配置源暂时无法访问时，DoDo 可以通过事先保存的备用地址更新配置。普通 HTTPS 链接、支持的 JSON/完整 Xray 配置、URI 列表和 Base64 均可使用，**不只适用于 crypt1 链接**。

本文以 iOS 2.2.0 和 Android APK 2.2.0 (2107) 为基准。已发布 Mac 版本没有此机制，也不要假设 Windows 已支持。参见[平台兼容性](compatibility.zh-CN.md)。

无需注册 DoDo、付费访问或服务商 API。Remnawave 与自建面板采用相同 HTTP 约定。

## 响应头

以下内容必须是真实 HTTP 响应头，不能是 JSON 字段或配置内注释：

| 头部 | 值 | 规则 |
|---|---|---|
| `fallback-url` | 当前用户备用配置源的完整 HTTPS 地址。 | 路径、令牌和 query 由服务商指定，不能只传域名。 |
| `providerid` | 可选、固定的服务 ID，例如 `demo-vpn`。 | 一旦与备用源一起保存，备用响应必须返回完全相同的值。 |

DoDo 只从真正来自主 HTTPS origin 的成功、可用响应中学习备用地址。其他主机的重定向或错误页不能作为注册备用源的可信依据。备用地址本身可以位于另一个合法 HTTPS 域名。

头部名称不区分大小写。`providerid` 的**值区分大小写**，长度为 1–128 个 ASCII 字符：首字符为字母或数字，后续允许字母、数字、`.`、`-`、`_`、`:`。不要把管理 API 令牌用作 ID。

`providerid` 不是付费权益、Happ 许可证、HWID、服务商控制台账号 ID，也不证明域名所有权。它与[服务商绑定和控制台功能](provider-console.zh-CN.md)所用的 `x-dodo-provider-id` 不同。备用 ID 匹配不会授予故事、通知或其他服务商用户数据的访问权限。

## 主源与备用源示例

下面只是不可用的演示地址：

```text
https://primary.example.com/sub/demo-token?format=json
```

响应体包含该用户的有效配置，响应头例如：

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/demo-token?format=json
```

这个精确备用地址必须提供同一用户的配置。如果主源给出了 `providerid`，备用响应也要带上：

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
providerid: demo-vpn
```

不提供 `providerid` 时，备用源也可按已保存的精确 HTTPS 地址工作。如果已保存预期 ID，而备用响应缺失或更改它，DoDo 会拒绝更新并保留原服务器。

DoDo **不会**把令牌/路径拼接到任意域名，不会按服务 ID 查找地址、猜测 `/json`，也不会追踪镜像链。服务商必须提供完整可用地址。

## 配置 Remnawave 或自建面板

1. 配好备用服务、DNS 和有效 HTTPS 证书。单独增加 DNS 记录不会自动创建备用端点。
2. 备用源应返回该用户的配置，而不是网页。保持认证、到期时间、流量和设备限制一致。
3. 在配置响应的 **Response Headers / Additional Response Headers** 中增加 `fallback-url` 和可选 `providerid`，具体名称取决于面板版本。
4. 对需要的格式与 User-Agent 规则启用这些头部，包括完整 JSON；不要覆盖无关格式规则。

在支持响应头变量的 Remnawave 版本中，模板可能如下：

```http
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/{{SHORT_UUID}}
```

请确认你的版本会展开 `{{SHORT_UUID}}`。真实 HTTP 响应必须包含某一用户的完整 URL，不能保留未展开模板，也不能把同一个个人地址发给所有人。自建后端可以直接生成这个精确地址。

同一服务器上的其他域名可能缓解域名被屏蔽，但服务器或 IP 本身不可用时无济于事。后者需要可独立访问的备用基础设施。

## 何时尝试备用源

| 主源故障 | 行为 |
|---|---|
| 网络故障、超时、HTTP 408 或 5xx | 可在共享时间和尝试次数上限内尝试已保存备用源。 |
| TLS 证书错误 | 不关闭证书验证，也不通过备用源绕过。 |
| 认证拒绝、HWID 限制、明确停用或过期 | 不绕过这些限制。 |
| HTTP 404、410 或 429 | 不通过备用源绕过。 |

只有一个备用源，没有镜像链或无限重试。本文移动端基准的总操作限时为 25 秒。请求使用同一已保存 HWID，重试不能重置设备数量限制。

成功后更新同一张卡片，保留原始源标识和卡片顺序。加载或校验失败时保留既有服务器与设置，不会用不完整结果替换。

## 必须先成功获取备用地址

主源必须先在支持该功能的版本中成功添加或更新过。第一次导入时主源已不可用，应用就无法读取尚未收到的响应头。仅有 `providerid` 不能找到备用地址。

[crypt1 与 AGE1](crypto.zh-CN.md) 也遵循同样规则。`response: "age1-required"` 要求主源与备用源都返回有效 AGE1，备用源不能把策略降级为明文。元数据应留在加密体以外的 HTTP 响应头中。

## 服务商测试清单

- 先完成普通更新，核对响应中的精确 `fallback-url`。
- 确认备用源服务于同一用户，并在设置 ID 时返回同样区分大小写的 `providerid`。
- 只模拟主源的适用故障；断开全部网络不能验证备用机制。
- 测试错误/缺失 ID 与无效备用响应体，原有可用配置必须保留。
- 实际 VPN 连接应单独验证，不要用配置更新成功代替。
- 不要在公开截图、日志或支持帖子中暴露真实令牌。

相关文档：[集成](integration.zh-CN.md) · [深度链接](deep-links.zh-CN.md) · [crypt1 与 AGE1](crypto.zh-CN.md) · [诊断](diagnostics.zh-CN.md)

官方参考：[备用配置：fallback-url](https://dodoproxy.app/docs/#subscription-fallback)。
