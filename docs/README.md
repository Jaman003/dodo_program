<div align="center">

<img src="../media/dodo-astronaut.png" width="96" alt="DoDo astronaut" />

# DoDo APP · Documentation

**Connect your service. Keep your configurations intact.**

[English](README.md) · [Русский](README.ru.md) · [فارسی](README.fa.md) · [简体中文](README.zh-CN.md)

[Product and downloads](../README.md) · [Website](https://dodoproxy.app/) · [Community](https://t.me/dododeveloper)

</div>

## Start here

DoDo is an Xray-based client for keys and configurations from your provider or your own server. It does not include free servers or sell VPN access. Ordinary import and VPN use do not depend on a provider-console account.

For users: install from the [official download links](../README.md#download), add a supported key using the clipboard, manual input or QR scanner, choose a server and approve the system VPN request. If something fails, start with [diagnostics](diagnostics.md). Never publish your personal configuration URL or an unredacted log.

For providers: start with [integration](integration.md), add an [Open in DoDo button](deep-links.md), and test a real connection on each target platform.

## Guides

| Guide | What you will find |
| --- | --- |
| [Provider integration](integration.md) | Response formats, full JSON, headers, auto-update interval and HWID |
| [Protocols and transports](protocols.md) | VLESS, VMess, Trojan, Shadowsocks, Hysteria2, SOCKS5, WireGuard, TLS / REALITY and limits |
| [Deep links](deep-links.md) | Bot and website buttons, URI encoding, supported commands and common mistakes |
| [Encrypted links](crypto.md) | Autonomous `dodoapp://crypt1/`, encryption tools, public keys and AGE1 bodies |
| [Backup addresses](fallback.md) | `fallback-url`, `providerid`, retries and safe matching |
| [Routing and geodata](routing.md) | Direct / Proxy / Block, `autorouting`, GeoIP / GeoSite, DNS, MTU and IP modes |
| [Provider console](provider-console.md) | Google sign-in, DNS TXT, provider ID, branding, stories and self-hosted console |
| [Ping and diagnostics](diagnostics.md) | What each ping measures, live logs, error reports and data controls |
| [Platform compatibility](compatibility.md) | Available, limited and unfinished features on iOS, Android, macOS and Windows |
| [Updates and feedback](updates-feedback.md) | Release history, suggestions, bug reports and moderation |

## Read the status before integrating

- **Available** means implemented for the specified platform and baseline, not every possible Xray profile.
- **Limited / early access** means there are format, device or rollout restrictions.
- **In development / not available** means it should not be promised to customers yet.

Feature tables use the iOS 2.2.0 and Android APK 2.2.0 (2107) baseline unless stated otherwise. The App Store lists iOS **2.2.1**, released **9 November 2026**; that alone does not confirm deployment of newer provider-console features. See [compatibility](compatibility.md) and [release history](updates-feedback.md).

The public `x-dodo-provider-id` UUID and the older `providerid` backup-address label are different fields. Neither confirms a paid account. Public branding/stories, private audience access and provider-specific device statistics have separate requirements and rollout status.

## Official links

[Website documentation](https://dodoproxy.app/docs/) · [Provider overview](https://dodoproxy.app/providers/) · [Version history](https://dodoproxy.app/updates/) · [Feedback](https://dodoproxy.app/feedback/) · [Telegram community](https://t.me/dododeveloper) · [Telegram news](https://t.me/dodoproxy)

This repository contains public documentation and media only, not application source code, signing files or private configuration keys.

---

**by Jaman Developer**
