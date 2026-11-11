[DoDo APP](../README.md) · [Docs](README.md)

[English](fallback.md) · [Русский](fallback.ru.md) · [فارسی](fallback.fa.md) · [简体中文](fallback.zh-CN.md)

# A backup source with `fallback-url`

If a primary configuration source is temporarily unreachable, DoDo can refresh it using a previously saved backup URL. This works with ordinary HTTPS links, supported JSON/full Xray profiles, URI lists and Base64. It is **not limited to crypt1 links**.

This guide describes iOS 2.2.0 and Android APK 2.2.0 (2107). The published Mac version does not provide this mechanism; do not assume Windows support. See [platform compatibility](compatibility.md).

No DoDo registration, paid access or provider API is required. Remnawave and custom panels follow the same HTTP contract.

## Response headers

Return these as real HTTP response headers, not JSON fields or comments inside the configuration:

| Header | Value | Rule |
|---|---|---|
| `fallback-url` | Complete HTTPS URL of this user's backup source. | Includes the provider-defined path, token and query. It is not just a domain. |
| `providerid` | Optional stable service ID, such as `demo-vpn`. | If saved with the backup, the backup must return exactly the same value. |

DoDo learns a backup only from a usable, successful response that actually came from the primary HTTPS origin. A redirect or error page from another host is not a trusted way to register a backup. The backup itself may be on a different valid HTTPS domain.

Header names are case-insensitive. The `providerid` **value is case-sensitive**, 1–128 ASCII characters: first a letter or digit, then letters, digits, `.`, `-`, `_` or `:`. Do not use an administrative API token as the ID.

`providerid` is not a premium entitlement, Happ license, HWID, provider-console account ID or proof of ownership. It is separate from `x-dodo-provider-id` used for [provider binding and console features](provider-console.md). A matching fallback ID does not grant stories, notifications or access to another provider's users.

## Example: primary and backup

Primary URL, shown only as a non-working example:

```text
https://primary.example.com/sub/demo-token?format=json
```

Its body contains the user's valid configuration, with headers such as:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/demo-token?format=json
```

The exact backup URL must serve the same user's configuration. When the primary supplied `providerid`, the backup response includes it too:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
providerid: demo-vpn
```

Without `providerid`, a backup can still work by the exact saved HTTPS URL. If an expected ID was saved but the backup omits or changes it, DoDo rejects the update and keeps the previous servers.

DoDo does **not** copy a token/path onto an arbitrary domain, find a URL by provider ID, guess `/json`, or follow a chain of mirrors. Supply the complete working address yourself.

## Set up Remnawave or a custom panel

1. Configure the backup service, DNS and a valid HTTPS certificate. A DNS record alone does not create a backup endpoint.
2. Make the backup return this user's configuration, not a landing page. Keep authorization, expiry, traffic limits and device limits consistent.
3. Add `fallback-url` and optionally `providerid` in the panel's subscription **Response Headers / Additional Response Headers** section; names vary by panel version.
4. Apply the headers to the intended formats and User-Agent rules, including full JSON. Do not replace unrelated response-format rules.

In Remnawave versions that support subscription variables in headers, a template may look like:

```http
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/{{SHORT_UUID}}
```

Check that your version expands `{{SHORT_UUID}}`. The actual HTTP response must contain one user's complete URL, not an unexpanded template or one personal URL reused for everyone. A custom backend can construct that exact URL directly.

An alternate domain on the same server may help with a domain-specific block, but not if the server or IP itself is unavailable. That case needs independently reachable backup infrastructure.

## When a backup is attempted

| Primary failure | Behaviour |
|---|---|
| Network failure, timeout, HTTP 408 or 5xx | A saved backup may be tried within the shared time/attempt budget. |
| TLS certificate error | No certificate-check bypass or fallback workaround. |
| Authorization refusal, HWID restriction, explicit disabled/expired access | These restrictions are not bypassed. |
| HTTP 404, 410 or 429 | Not bypassed through the backup. |

There is one backup, no mirror chain and no unlimited retries. The documented mobile operation budget is 25 seconds. Requests use the same stored HWID; retrying is not a way to reset device limits.

On success, the same card is refreshed: the original source identity and card order remain. If loading or validation fails, DoDo preserves the existing servers and settings rather than replacing them with an incomplete result.

## The backup must be learned first

The primary must have been successfully added or refreshed in a supporting version. If it is unavailable on the very first import, DoDo cannot read a header from the unavailable response. `providerid` alone cannot find the backup.

For [crypt1 and AGE1](crypto.md), the same rules apply. With `response: "age1-required"`, both primary and backup must return valid AGE1; the backup cannot downgrade the policy to plaintext. Keep metadata headers outside an encrypted body.

## Provider test checklist

- First complete a normal refresh and verify the exact `fallback-url` in your server response.
- Confirm the backup serves the same user and, if set, returns the same case-sensitive `providerid`.
- Simulate an eligible failure of the primary only; disabling all networking does not test fallback.
- Check a wrong/missing ID and invalid backup body: the previous working configuration must remain.
- Verify the real VPN connection separately from a successful configuration refresh.
- Do not include real tokens in public screenshots, logs or support posts.

Related: [Integration](integration.md) · [Deep links](deep-links.md) · [crypt1 and AGE1](crypto.md) · [Diagnostics](diagnostics.md)

Canonical reference: [Backup subscription: fallback-url](https://dodoproxy.app/docs/#subscription-fallback).
