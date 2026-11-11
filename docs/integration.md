[DoDo APP](../README.md) · [Documentation](README.md)

[English](integration.md) · [Русский](integration.ru.md) · [فارسی](integration.fa.md) · [简体中文](integration.zh-CN.md)

# Provider integration

Give your users an HTTPS configuration link and an [Open in DoDo](deep-links.md) button. Ordinary import and VPN use do not require a DoDo account, API token or a particular panel. Remnawave, 3x-ui and custom services can deliver supported formats.

The details below describe the iOS 2.2.0 and Android APK 2.2.0 (2107) feature baseline. For rollout differences, see [platform compatibility](compatibility.md).

## 1. Return the right configuration

Match iOS requests with `User-Agent: DoDoApp`. Android uses the `DoDoApp/` prefix, for example `DoDoApp/2.2.0+2107 (Android 15; Demo Model)`. A rule that requires an exact `DoDoApp` match will miss Android.

Return HTTP 200 with a complete supported body. Use `application/json` for JSON. This non-working example shows a plain-text response:

```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
profile-title: Demo Provider
profile-update-interval: 1
support-url: https://example.invalid/support

vless://00000000-0000-4000-8000-000000000001@example.invalid:443?security=tls&type=ws&path=%2Fdemo#Demo
```

| Response body | How DoDo treats it |
| --- | --- |
| URI list | Independent servers, usually one per line |
| Base64 | A supported encoded list or JSON container; Base64 is not encryption |
| Complete Xray JSON | One logical profile with its DNS, routing, balancers and outbound dependencies |
| Array of complete JSON profiles | Each standalone profile becomes a choice |
| Collection of independent outbounds | Separate servers, without inventing a balancer |

Do not flatten a balanced or chained profile into separate server links. DoDo preserves compatible tags, dependencies, `observatory`, `burstObservatory`, `policy` and `stats`; local service inbounds and runtime ports may be adapted. Compatibility still depends on the bundled core.

An invalid or unsupported profile is rejected rather than silently simplified. A failed refresh keeps the previous server list. Matching servers retain their identity, order, pinning, selection and last ping values.

### Temporary Happ JSON compatibility

DoDo requests the ordinary URL first. If it receives usable complete JSON, no compatibility request is needed. Otherwise it may retry **the same URL** with `User-Agent: Happ/DoDoApp-compat` and an `X-DoDo-Client` header identifying the real client. Responses are not merged.

Prefer a DoDo response rule or an explicit full-format URL supplied by your panel. DoDo does not append `/json` to arbitrary URLs. A failed compatibility attempt keeps a usable primary response; an existing full profile is not downgraded to an incomplete URI list. This is response-format compatibility, **not** support for Happ deep links or private encryption.

## 2. Add card metadata

Send these as HTTP response headers. Header names are case-insensitive.

| Header | Value and effect |
| --- | --- |
| `profile-title` | Service name, plain text or `base64:` UTF-8. Fallback: `subscription-name`, then `Content-Disposition` filename / filename* |
| `x-dodo-provider-logo-url` | Public HTTPS image URL for the service logo, not the app icon |
| `support-url` | Support page or bot; adds the Support button when present |
| `profile-web-page-url` | Account, bot or renewal page; alias `homepage`. Payment takes place on your service |
| `subscription-userinfo` | `upload=…; download=…; total=…; expire=…`; traffic in bytes, expiry as Unix seconds |
| `profile-update-interval` | Integer **hours**, 1–168; default 1 hour. Background execution is scheduled by the OS |
| `announce` | Text or supported `base64:` text for a card announcement; not a push or story |
| `routing` / `autorouting` | Static or remotely updated [routing rules](routing.md) |
| `fallback-url` / `providerid` | A previously supplied [backup address](fallback.md) and its matching identity |
| `x-dodo-provider-id` | Public provider UUID for the separate [provider-console integration](provider-console.md) |

For ordinary metadata, DoDo reads real headers first, then supported body comments, decoded Base64 and JSON metadata; the first valid value wins. Static files can use supported comment fields such as `#profile-title: Demo Provider`.

**Exception:** `x-dodo-provider-id` must come from a successful HTTPS response on the exact verified host, not a body comment. Do not confuse it with the older `providerid` fallback label. Neither field proves payment or subscriber ownership.

Do not put a subscriber token in a logo, support or other public metadata URL. HTTP 403 alone does not define an expiry date.

## 3. Enforce device limits on your server

DoDo sends an installation ID and device details when requesting a configuration:

```http
User-Agent: DoDoApp
Accept: */*
x-hwid: 00000000-0000-4000-8000-000000000001
x-device-os: iOS
x-ver-os: 18.5.0
x-device-model: iPhone 14 Pro
```

Android uses the same technical header names with its own model, OS version, Android OS name and versioned User-Agent. HWID is not a serial number or IDFA. It remains stable across retries; an app-settings reset does not reset the provider's device quota. It is sent as a header, never appended to the URL.

| Provider response | Result |
| --- | --- |
| `x-hwid-max-devices-reached: true` | Device-limit error |
| `x-hwid-limit: true` | Legacy equivalent |
| `x-hwid-not-supported: true` or explicit `HWID_REQUIRED` | Clear loading failure, without generating a replacement ID |

The provider controls authorization and the allowed device count. HWID is not authentication. Cross-origin redirects remove sensitive headers on iOS and are rejected on Android; HTTPS-to-HTTP downgrade is not allowed.

## Limits and a release checklist

| Limit | iOS baseline | Android APK baseline |
| --- | --- | --- |
| Response | 8,000,000 bytes | 2 MiB |
| Total operation | 12,000,000 bytes | 4 MiB |
| Individual JSON profile | 2,000,000 bytes | Within the response limit |
| AGE1 encrypted body | 2 MiB | 2 MiB |
| Overall loading time | 25 seconds, bounded attempts | 25 seconds, bounded attempts |

TLS verification is not disabled to rescue a failed load. An incomplete response with a network error is not imported.

Before sending your button to users, check import, refresh, profile count and order, expiry/traffic, device-limit errors and one real connection on each target platform. Test the exact transport and authentication settings your panel issues. Import success or a TCP ping does not prove working internet access.

Next: [Protocols](protocols.md) · [Deep links](deep-links.md) · [Encrypted links](crypto.md) · [Troubleshooting](diagnostics.md)

Source: [DoDo integration reference](https://dodoproxy.app/docs/#quickstart), [response headers](https://dodoproxy.app/docs/#headers), [HWID](https://dodoproxy.app/docs/#device-hwid).
