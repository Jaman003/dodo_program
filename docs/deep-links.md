[DoDo APP](../README.md) · [Docs](README.md)

[English](deep-links.md) · [Русский](deep-links.ru.md) · [فارسی](deep-links.fa.md) · [简体中文](deep-links.zh-CN.md)

# Deep links for bots and websites

Use `dodoapp://import?url=…` to open DoDo and offer to add a user's configuration. A deep link is not an account login and does not create VPN access: your service must already have issued a valid URL to that user.

The mobile examples below describe iOS 2.2.0 and Android APK 2.2.0 (2107). Do not assume every command is available on macOS or Windows; see [platform compatibility](compatibility.md).

## Recommended import link

```text
dodoapp://import?url=https%3A%2F%2Fexample.com%2Fsub%2Fdemo-token
```

The source URL in this example is `https://example.com/sub/demo-token`. It is a placeholder, not a working key.

| Form | Meaning |
|---|---|
| `dodoapp://import?url=…` | Recommended import command. |
| `dodoapp://import-sub?url=…` | Supported import alias. |
| `link` instead of `url` | Supported parameter alias; send exactly one non-empty target parameter. |
| `dodoapp://crypt1/<JWE>` | DoDo protected-link import; see [crypt1 and AGE1](crypto.md). |

Use a complete HTTPS URL for provider integrations. Do not send conflicting `url` and `link` parameters, two copies of the same parameter, or another app command as the import target.

## Encode each wrapper once

Treat the source URL as an opaque string. Encode it once when placing it inside the deep link's query parameter. Do not decode and rebuild paths, tokens, `%2F`, `%252F`, `+`, or the source query.

```js
const source = 'https://example.com/sub/demo-token?format=json&value=a%2Fb+test';
const deepLink = `dodoapp://import?url=${encodeURIComponent(source)}`;

// Only if your own HTTPS landing page accepts a deep link in its url parameter:
const landingLink = `https://example.com/connect?url=${encodeURIComponent(deepLink)}`;
```

The second line adds one encoding layer. The last line adds a separate layer for a separate outer URL. A landing page must remove only its own layer before opening the deep link; repeatedly decoding the whole string can change the user's key.

Avoid using `application/x-www-form-urlencoded` rules that turn an unescaped `+` into a space. `encodeURIComponent` encodes a literal plus as `%2B`.

## Older and unsupported forms

| Form | What to do |
|---|---|
| `dodoapp://add/<URL>` | Do not use. `add` is not a documented import command. |
| `dodoapp://import/<URL>` | Do not use the path-style form. Replace it with `import?url=` and encode the target once. |
| `dodo://import…` | Not supported for configuration import. The `dodo` scheme is used for routing commands below. |
| `happ://…`, including `happ://crypto…` | Not DoDo commands. Temporary HTTP/JSON format compatibility does not register another app's scheme or decrypt its private format. |

Do not generate two entries with the same `dodo` key in a bot's client map: one may overwrite the other. Keep a single recommended template.

## Routing is a separate command

```text
dodo://routing/add/<BASE64URL_UTF8_JSON>
dodo://routing/onadd/<BASE64URL_UTF8_JSON>
dodo://routing/off
```

The `dodoapp` scheme is also accepted for these routing commands. Encode a **DoDo rules profile**, not a complete Xray configuration. Base64URL without trailing `=` is convenient for the path; ordinary Base64 needs its own single percent-encoding layer.

Adding a routing profile takes effect on the next connection. `off` stops applying the imported profile without deleting it. Rule fields, remote updates, and platform limits are documented in [Routing and geodata](routing.md).

## VPN commands differ by platform

| Command | iOS | Android APK 2.2.0 |
|---|---|---|
| `dodoapp://toggle-vpn` | Toggles the current VPN state. Opening it again reverses the state. | Not available. |
| `dodoapp://vpn-permission` | Shows an explanation; it does not bypass or automatically display Apple's permission dialog. | Opens the Android VPN permission/explanation flow. |
| `dodoapp://vpn-disclosure` | Alias of the explanation command. | Android disclosure/permission flow. |

A toggle is not an idempotent “connect” command. Do not retry it automatically on page load or after a timeout.

## Before putting a button in a bot

1. Install DoDo, then open the recommended import link from the same browser or messenger your users use.
2. Test a source URL containing query parameters, `%` escapes and `+` without changing the source string.
3. Confirm the imported profile count, metadata, update and actual VPN connection; opening the app alone is not proof of a working tunnel.
4. Check cancellation and repeated opening. Do not log a real source URL or protected link.

DoDo must already be installed. An App Store or Google Play installation does not preserve this import action automatically: the user should open the original link again afterward. A Telegram account is not identified by a deep link alone. Clipboard import happens after a user action, not silent clipboard interception.

Long crypt1 links may be too dense for a QR code. Android camera recognition of these codes has a known limitation on some HUAWEI devices; offer text copy/manual import as well.

Related: [Provider integration](integration.md) · [crypt1 and AGE1](crypto.md) · [Fallback addresses](fallback.md) · [Diagnostics](diagnostics.md)

Canonical reference: [Opening DoDo from bots and websites](https://dodoproxy.app/docs/#deeplinks).
