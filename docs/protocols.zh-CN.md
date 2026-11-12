[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](protocols.md) · [Русский](protocols.ru.md) · [فارسی](protocols.fa.md) · [简体中文](protocols.zh-CN.md)

# 协议、传输与 TLS

协议名称本身不能保证兼容。协议、传输、安全参数及内置核心必须支持服务器所用的组合。下表以 iOS 2.2.0 功能为基准，Android 差异见下文。

## 连接协议

| 协议 | 导入格式 | 支持范围与限制 |
| --- | --- | --- |
| VLESS | `vless://`、Xray JSON | 支持，包括兼容的 TLS / REALITY 配置 |
| VMess | 含 Base64 JSON 的 `vmess://`、Xray JSON | 支持；会验证传输与安全字段 |
| Trojan | `trojan://`、Xray JSON | 支持；URI 导入默认使用 TLS |
| Shadowsocks | `ss://`、Xray JSON | 常见 SIP002 / Base64 形式；加密算法取决于核心，不支持 URI 中的外部插件 |
| Hysteria2 | `hy2://`、`hysteria2://`、兼容的 Xray JSON | 第 2 版；支持兼容格式的 Salamander 和 hopping 参数 |
| SOCKS5 | `socks://`、`socks5://`、Xray JSON | 支持有或无用户名/密码；SOCKS 本身不提供传输加密 |
| 标准 WireGuard | `wg://`、`wireguard://` | 有限支持：URI 参数或 URI 内的 Base64 wg-quick 配置；不支持任意多 peer 文件或 AmneziaWG 扩展 |
| AmneziaWG、TUIC、SSR、Hysteria v1 | 各自的链接格式 | iOS 不支持；不会将 AmneziaWG 转换成标准 WireGuard |

兼容的完整 JSON 可以包含辅助 HTTP、SOCKS 和 WireGuard 出站，但不代表任意对应 URI 都能单独导入。

**Android APK：**与 iOS 不同，Android 保留了 AmneziaWG。Hysteria2 已实现导入和运行，请测试服务器的具体参数。完整 JSON 需要可识别的主要代理协议及其出站依赖。在向所有设备提供同一配置前，请查看[平台兼容性](compatibility.zh-CN.md)。

## 传输与 URI 字段

| 字段 | 值或用途 |
| --- | --- |
| `type` / `network` | `tcp`、`raw`、`ws`、`websocket`、`grpc`、`xhttp`、`splithttp`、`httpupgrade`、`kcp`、`mkcp` |
| 别名 | `websocket` → WS；`splithttp` → XHTTP |
| `security` | `none`、`tls`、`reality`，仅限协议与核心支持的组合 |
| TLS / REALITY | `sni` / `peer`、`fp` / `fingerprint`、`alpn`；REALITY 还使用 `pbk`、`sid`、`spx` |
| WS | `host`、`path` |
| gRPC | `serviceName`、`authority` |
| XHTTP | `host`、`path`、`mode`、JSON 对象 `extra` |
| 证书验证 | `pcs` / `pinnedPeerCertSha256`、`vcn` / `verifyPeerCertByName` |

请使用服务器实际配置的参数。对于嵌套传输参数、级联出站、负载均衡和服务商 DNS，应优先使用[完整 Xray JSON](integration.zh-CN.md)，而不是缩短的 URI。

**Hysteria2 使用 QUIC，但不需要 `type=quic`。** 应通过 `hy2://` 或 `hysteria2://` 导入。本文所述 iOS 导入器不支持 URI 传输值 `quic`、`h2` 和 `http`；这属于导入器限制，并不意味着核心没有基于 QUIC 的协议。

## 安全与高级选项

- 无效的证书 pin、不支持的强制 ECH 要求、绕过证书验证的请求会使该 URI 被拒绝，而不会降低 TLS 验证强度。
- 兼容的完整 JSON 可携带 TLS 分片，包括 `freedom.settings.fragment`；应用没有独立分片编辑器。
- 已准备的 **iOS 2.2.2（113）**增加兼容的 `fm`、`fragment`、`fragmentPackets` / `fragmentLength` / `fragmentInterval` 及文档列出的 `fragmentation-*` HTTP 响应头。完整 JSON 优先，掩码不叠加，TLS 验证不降低，也不会自动对 Hysteria 应用 TCP 分片。构建已上传至 App Store Connect 以供处理，尚未提交 App Review。[详情与状态](ios-2.2.2.zh-CN.md)。
- `flow`、mux、套接字和传输设置必须匹配服务器与核心。支持 VLESS 不代表保证所有 Vision / REALITY / XHTTP 组合可用。
- 导入成功不能证明服务器可达。[HTTP 测试与诊断日志](diagnostics.zh-CN.md)可帮助区分加载、认证和网络故障。

DoDo 是客户端，不出售 VPN 访问。请向服务商或管理员获取连接密钥与服务器参数。

来源：[DoDo 协议](https://dodoproxy.app/docs/#protocols)、[传输与 TLS](https://dodoproxy.app/docs/#transports)。核心设置的含义见 [Xray 官方配置文档](https://xtls.github.io/config/)。
