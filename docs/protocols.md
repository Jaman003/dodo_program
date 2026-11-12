[DoDo APP](../README.md) · [Documentation](README.md)

[English](protocols.md) · [Русский](protocols.ru.md) · [فارسی](protocols.fa.md) · [简体中文](protocols.zh-CN.md)

# Protocols, transports and TLS

A protocol name alone does not guarantee compatibility. The protocol, transport, security settings and bundled core must support the same combination as your server. This table describes the iOS 2.2.0 feature baseline; Android differences follow below.

## Connection protocols

| Protocol | Import format | Availability and limits |
| --- | --- | --- |
| VLESS | `vless://`, Xray JSON | Supported, including compatible TLS / REALITY profiles |
| VMess | `vmess://` with Base64 JSON, Xray JSON | Supported; transport and security fields are validated |
| Trojan | `trojan://`, Xray JSON | Supported; URI import uses TLS by default |
| Shadowsocks | `ss://`, Xray JSON | Common SIP002 / Base64 forms; cipher support depends on the core. External URI plugins are not supported |
| Hysteria2 | `hy2://`, `hysteria2://`, compatible Xray JSON | Version 2; Salamander and hopping parameters in the supported format |
| SOCKS5 | `socks://`, `socks5://`, Xray JSON | With or without username/password. SOCKS alone does not add transport encryption |
| Plain WireGuard | `wg://`, `wireguard://` | Limited: URI parameters or a Base64 wg-quick profile inside the URI. Not arbitrary multi-peer files or AmneziaWG extensions |
| AmneziaWG, TUIC, SSR, Hysteria v1 | Their own schemes | Not supported in iOS; AmneziaWG is not converted into plain WireGuard |

Compatible full JSON may include auxiliary HTTP, SOCKS and WireGuard outbounds. That does not make arbitrary standalone URI schemes importable.

**Android APK baseline:** AmneziaWG remains available on Android, unlike iOS. Hysteria2 import and runtime are implemented; test your server's exact parameters. Full JSON requires a recognized primary proxy protocol and its outbound dependencies. See [platform compatibility](compatibility.md) before offering the same profile to every device.

## Transports and URI fields

| Field | Accepted values / purpose |
| --- | --- |
| `type` / `network` | `tcp`, `raw`, `ws`, `websocket`, `grpc`, `xhttp`, `splithttp`, `httpupgrade`, `kcp`, `mkcp` |
| Aliases | `websocket` → WS; `splithttp` → XHTTP |
| `security` | `none`, `tls`, `reality`, only in combinations supported by the protocol and core |
| TLS / REALITY | `sni` / `peer`, `fp` / `fingerprint`, `alpn`; REALITY also uses `pbk`, `sid`, `spx` |
| WS | `host`, `path` |
| gRPC | `serviceName`, `authority` |
| XHTTP | `host`, `path`, `mode`, JSON-object `extra` |
| Certificate checks | `pcs` / `pinnedPeerCertSha256`, `vcn` / `verifyPeerCertByName` |

Use the actual values configured on your server. For nested transport settings, chained outbounds, balancers or provider DNS, prefer [complete Xray JSON](integration.md) over a shortened URI.

**Hysteria2 uses QUIC without `type=quic`.** Import it with `hy2://` or `hysteria2://`. URI transport values `quic`, `h2` and `http` are not supported by the documented iOS importer. This is an importer limitation, not a claim that the core has no QUIC-based protocols.

## Security and advanced options

- Invalid certificate pins, unsupported mandatory ECH requirements and certificate-verification bypass requests cause rejection of that URI, not weakened TLS checks.
- Compatible full JSON can carry TLS fragmentation, including `freedom.settings.fragment`. There is no dedicated fragmentation editor.
- Prepared **iOS 2.2.2 (113)** adds compatible `fm`, `fragment`, `fragmentPackets` / `fragmentLength` / `fragmentInterval` and the documented `fragmentation-*` HTTP headers. Complete JSON takes precedence; masks are not stacked, TLS is not weakened and automatic TCP fragmentation is not applied to Hysteria. The build is uploaded to App Store Connect for processing, not yet submitted to App Review. [Details and status](ios-2.2.2.md).
- Supported `flow`, mux, socket and transport settings must match the server and core. “VLESS supported” does not certify every Vision / REALITY / XHTTP combination.
- Importing a profile does not prove its server is reachable. [HTTP tests and diagnostic logs](diagnostics.md) help separate loading, authentication and connectivity failures.

DoDo is a client, not a VPN subscription provider. Get your keys and server settings from your service or administrator.

Sources: [DoDo protocols](https://dodoproxy.app/docs/#protocols), [transports and TLS](https://dodoproxy.app/docs/#transports). For the meaning of core settings, see the [official Xray configuration documentation](https://xtls.github.io/en/config/).
