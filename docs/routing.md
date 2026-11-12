[DoDo App](../README.md) · [Documentation](README.md)

English · [Русский](routing.ru.md) · [فارسی](routing.fa.md) · [简体中文](routing.zh-CN.md)

# Routing, geodata and network settings

Use routing rules to send selected destinations directly, through the proxy, or to a block rule. DoDo's small routing format is separate from a full Xray configuration. Do not put a complete Xray JSON into the rule fields below.

This page describes the documented iOS 2.2.0 and Android APK 2.2.0 (2107) baseline. Availability in another build must be checked separately.

**Prepared iOS 2.2.2 (113)** improves compatible DNS alignment, split DoH and local aliases with cycle protection. It supports explicit compatible FakeDNS profiles with standard pools, not arbitrary mixed DNS graphs or Lua. This does not extend the small `autorouting` format: its DNS/FakeDNS restrictions below remain. [Support boundaries and App Store Connect processing status](ios-2.2.2.md); the build has not been submitted to App Review or announced as available in the store.

## Static rules

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

| Field | Purpose |
| --- | --- |
| `Name` | Display name of the rule profile. |
| `GlobalProxy` | Global proxy setting for this rule profile. |
| `DomainStrategy` | Domain resolution strategy. The remote format accepts `AsIs`, `IPIfNonMatch` and `IPOnDemand`. |
| `DirectSites` / `DirectIp` | Destinations to reach directly. |
| `ProxySites` / `ProxyIp` | Destinations to send through the proxy. |
| `BlockSites` / `BlockIp` | Destinations to block. |

Domain lists accept domains and supported `domain:`, `full:`, validated `regexp:` and `geosite:` entries. IP lists accept IP addresses, CIDR networks and `geoip:` entries. Referenced geodata categories must exist in the available files.

### Import a local rule profile

Encode the UTF-8 JSON as Base64 or Base64URL. Base64URL without trailing `=` is easier to use in a link; ordinary Base64 can be percent-encoded once.

```text
dodo://routing/add/{BASE64_UTF8_JSON}
dodo://routing/onadd/{BASE64_UTF8_JSON}
dodo://routing/off
```

Both `add` and `onadd` save and enable the profile. `off` disables it without deleting it. Do not add a username, password, port, query or fragment to these links. Rules take effect on the next connection; the current VPN is not reconnected automatically.

See [deep links](deep-links.md) for opening DoDo from a bot or browser.

## Remote rules: `autorouting`

Return an `autorouting` HTTP response header containing a public HTTPS URL for a separate rules file. Use your real public hostname: example `.invalid` addresses and local addresses are not accepted.

```http
autorouting: https://rules.example.com/routing.json
```

The endpoint must return HTTP 200 and a UTF-8 JSON **object** using the fields above. It is not a Base64 payload, a routing link, or a full Xray JSON.

| Requirement | Accepted behavior |
| --- | --- |
| Schema | Only the example fields and an optional string `LastUpdated`; supported camelCase aliases are accepted. Unknown fields and conflicting aliases are rejected. |
| Values | Lists are arrays of strings. `GlobalProxy` is a JSON boolean or the string `"true"` / `"false"`. |
| Size | At most 256 KiB per file, 2,048 entries in total and 512 bytes per entry. |
| URL | HTTPS on the standard port, without user information or a fragment. Redirects are limited and must stay on the same origin. |
| Request privacy | The rules request does not carry subscription HWID, cookies or subscription API tokens. |
| Failure | The last working rules remain available after a network error or invalid JSON. |
| Scope | Rules are cached separately for each key profile and applied on the next connection. |

| Platform baseline | Successful-source recheck | Request timeout |
| --- | --- | --- |
| iOS 2.2.0 | No more often than once an hour; retry backoff after errors | Up to 12 seconds |
| Android APK 2.2.0 | 24 hours | 8 seconds |

Background execution depends on the operating system. These intervals are not a promise of an exact background schedule and are unrelated to `profile-update-interval`, which updates the key itself.

The subscription body can also contain `dodo://autorouting/add/…` or `dodo://autorouting/onadd/…` with an HTTPS source. These are subscription-content commands, not links for opening DoDo from a browser or bot.

DNS/FakeDNS settings, arbitrary `Geoipurl` / `Geositeurl` sources and `routeOrder` are **not** supported by this small rule format. Use a compatible full Xray profile for a more complex graph.

## Full Xray profiles and balancers

DoDo keeps a full supported profile together with its linked outbounds, DNS, routing, balancers and transport settings. A balancer profile is not meant to become several unrelated server rows.

Import validation is not a server-configuration repair tool. Do not strip `fallbackTag`, replace it with `direct`, or remove a balancer to make a file import. Whether a missing fallback target is accepted depends on the client/core combination; acceptance at load time does not prove that the intended fallback works. Providers should verify their outbound tags and actual failover behavior.

The temporarily hidden DoDo “Auto-select” button, a balancer inside Xray JSON, and a future provider-API Auto service are three different features. Hiding the button does not disable a balancer already present in a full profile. Provider-API Auto is not generally available.

See [formats and integration](integration.md), [protocols](protocols.md) and [compatibility](compatibility.md).

## Bundled `geoip.dat` and `geosite.dat`

Both files are bundled in the documented iOS and Android baseline. They do not have to be downloaded with the first key. Updates come from [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat).

Before installation, DoDo checks SHA-256, file format and required categories. A failed validation leaves the last working pair in place. Categories from another geodata collection may not exist here; arbitrary provider-hosted geofiles are not supported.

On iOS, open **Settings → Advanced → Geodata** to inspect versions or request a check. Automatic checks run when the system permits and the source is reachable. On Android, an active VPN keeps using its selected files; updated files apply on the next connection, with rollback to a working version available.

These files contain routing categories. They are not a map of DoDo users or a source of provider analytics.

## DNS, MTU and IP preference

Keep defaults unless investigating a specific network problem. Changes affect the next connection, not a running VPN, and do not rewrite the saved key.

| Setting | Meaning and limits |
| --- | --- |
| Tunnel MTU | Default/Auto, presets 1420 / 1380 / 1280, or an integer from 1280 to 1500 bytes. It controls packet size, not a speed cap. DoDo does not automatically search for the best MTU. |
| IP preference | “As configured”, “Prefer IPv4” or “Prefer IPv6” for a server addressed by a hostname. This is a preference, not a switch that disables IPv6 on the device. Complex profiles may retain the provider's behavior. |
| DNS from the profile | iOS can coordinate system tunnel DNS with Xray when the profile can be adapted safely. Ambiguous DNS graphs, FakeDNS and system-DNS dependencies are not forcibly rewritten. Android preserves the provider DNS too, but its tunnel implementation differs. |
| VPN resources | The iOS advanced screen shows extension memory, Go heap, connections and file descriptors when diagnostic logs and a fresh session provide data. Missing data is not reported as zero. |

For IPv6-only, DNS64 or 464XLAT networks, a fixed IPv4-only choice is not a universal fix. Test the actual network and provider profile; neither an IP preference nor a smaller MTU guarantees compatibility.

A manual Xray memory-limit slider, “No Limit Mode” and a general editor for every core option are not part of this documented baseline.

## Related pages

[Fallback addresses](fallback.md) · [Protected links](crypto.md) · [Provider console](provider-console.md) · [Diagnostics](diagnostics.md) · [Updates and feedback](updates-feedback.md)

Sources: [Routing](https://dodoproxy.app/docs/#routing), [Geodata](https://dodoproxy.app/docs/#geodata), [Network settings](https://dodoproxy.app/docs/#network), [Platform status](https://dodoproxy.app/docs/#status).
