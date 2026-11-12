# iOS 2.2.2: configuration compatibility and server checks

[DoDo APP](../README.md) · [Documentation](README.md) · [Release history](updates-feedback.md)

[English](ios-2.2.2.md) · [Русский](ios-2.2.2.ru.md) · [فارسی](ios-2.2.2.fa.md) · [简体中文](ios-2.2.2.zh-CN.md)

**Status on 12 November 2026:** **iOS 2.2.2 (113)** has been uploaded to App Store Connect for processing. It has not been submitted to App Review and is not announced as available on the App Store. Processing, review and publication are separate stages.

These changes apply only to this iOS build. It requires iOS 15 or later and retains Xray 26.10.31. They do not update Android APK 2.2.0 (2107), macOS or Windows.

## Full JSON: `env` and `version`

Root-level `env` and `version` are preserved with a full profile. A document containing only `outbounds` and these fields is no longer reduced to separate links. An ordinary collection of independent outbounds without full-profile requirements remains a collection.

The pinned core checks `version.min` / `version.max` before startup; incompatible requirements are not ignored. Supported `env` values apply to the relevant helper core or connection and are restored after confirmed cleanup, so one service's settings do not carry into the next session. Resource paths, app settings, Go/TUN, proxy and TLS-trust parameters are protected. An unsupported active requirement is rejected explicitly without including ENV values in the error.

## HTTP GET / HEAD through the profile

Connections and HTTP checks share configuration preparation: the selected proxy or balancer, outbound dependencies, DNS, routing, transport settings and geodata scope. Checks use an isolated helper Xray instance without reconnecting the active VPN. A failed HTTP request is not replaced with a successful direct connection or an open TCP port.

Geodata preparation for Hysteria2 HTTP checks has been fixed. As Hysteria2 uses UDP, TCP mode continues to select an HTTP check automatically for these profiles. Ordinary TCP shows its raw measured time without division; only estimated DoDo Ping divides HTTP GET by 3.3. This is not a speed test or a complete test of an active VPN tunnel.

## DNS, split DoH and limited FakeDNS

This build improves alignment between Xray DNS and the system tunnel, split DoH and local DNS aliases. Alias chains are bounded and checked for cycles. Ambiguous configurations are not forcibly rewritten just to start.

Compatible, explicitly configured provider FakeDNS profiles with standard pools are supported. This is not universal support for arbitrary pools, mixed DNS arrangements or Lua. Separate HTTP checks are unavailable for FakeDNS profiles; an `N/A` result in that check alone does not mean the VPN is broken.

## FinalMask and fragmentation parameters

Compatible URIs can supply `fm`, `fragment` and `fragmentPackets` / `fragmentLength` / `fragmentInterval`. The following parameters are accepted from the actual successful HTTP response:

```http
fragmentation-enable: true
fragmentation-type: xray
fragmentation-packets: tlshello
fragmentation-length: 50-100
fragmentation-delay: 10-20
fragmentation-maxsplit: 100-200
```

`fragmentation-interval` is an alternative HTTP header name for delay, not an instruction to rename fields inside complete JSON. Header-based masking requires `fragmentation-enable: true` and type `xray`.

Precedence is **complete JSON → explicit URI mask → response headers**. Masks are not automatically stacked. Provider JSON is not silently repaired, and unsupported active modes are rejected. Automatic TCP fragmentation is not applied to UDP-only / Hysteria profiles. There is no separate UI editor for every masking parameter.

## What stays unchanged

Valid keys, profile order and server settings do not need to change just because the client is updated. TLS certificate verification is not weakened, and no new SDK is added. Successful import or a check against one URL cannot guarantee every node or network; providers still need to validate their DNS, tags and transport settings.

[Compatibility](compatibility.md) · [Integration](integration.md) · [Routing](routing.md) · [Diagnostics](diagnostics.md)

Public overview: [iOS 2.2.2 preparation](https://dodoproxy.app/docs/#ios-next-update). App Store availability will be announced separately.
