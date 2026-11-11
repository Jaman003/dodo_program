# Provider console

[DoDo APP](../README.md) · [Documentation](README.md)

[English](provider-console.md) · [Русский](provider-console.ru.md) · [فارسی](provider-console.fa.md) · [简体中文](provider-console.zh-CN.md)

## Start with your service

Provider registration is free and in early access. Full production rollout is still being prepared. Sign in with Google at [DoDo Partners](https://api.dodoproxy.app/), create your service and add its public support contact. A Remnawave account, administrative API token or VPN-node access is not required.

Public branding and stories require three separate checks:

1. The owner signs in with Google.
2. The exact host issuing configurations is verified through DNS TXT.
3. DoDo approves the service.

For `sub.example.com`, the TXT record has this structure. Use the unique value from your own account, not this placeholder:

```text
_dodo-verify.sub.example.com
dodo-verification=v1:CODE_FROM_YOUR_ACCOUNT
```

Verification covers only that host, not all subdomains or unrelated services on a shared CDN. It lasts 30 days. The console warns seven days before expiry; request a new TXT record and complete renewal manually. There is no automatic background renewal.

## Return your provider ID

Add this real HTTP header to a successful HTTPS configuration response from the verified host:

```http
x-dodo-provider-id: YOUR_PROVIDER_UUID_FROM_DODO
```

Do not put it in a JSON field or body comment. An error page, redirect or response from another host does not establish this link.

The UUID is public. It is not an API token, proof of payment, premium status or proof that a particular person owns a configuration. It is different from the short `providerid` used for [fallback addresses](fallback.md). Public branding and stories do not require a module, signer or private signing key. Closed subscriber content would need a separate ownership check.

## Host your own console

[Download Provider Console 0.1.1](https://dodoproxy.app/providers/DoDo-Provider-Console-0.1.1.tar.gz) · [Installation guide](https://dodoproxy.app/providers/DoDo-Provider-Console-0.1.1-README-RU.md)

The package runs on your Linux server using Docker Compose, behind HTTPS on an exact configured origin. Verify `SHA256SUMS` and follow the packaged instructions. It does not need root access to your VPN infrastructure, host networking, the Docker socket or a Remnawave module.

Approve the installation through Google using its one-time private setup code, then sign in separately as an owner or editor. Installation approval does not replace a person's permissions. Each installation has revocable, provider-scoped access to the central DoDo API; the package does not include the main administrator interface, platform master keys or push credentials. Content and published media remain on DoDo's side.

## Stories and analytics

| Feature | Scope and status |
| --- | --- |
| Stories | Draft, preview, publish for 12 or 24 hours, unpublish and archive. Requires active DNS verification, DoDo approval and a compatible new iOS build. Client availability is limited during early access; publishing does not guarantee display in every installed version. |
| Media | JPEG, PNG or WebP up to 10 MB; MP4 up to 16 MB and 30 seconds. Unpublish a story before replacing its media. |
| Provider-ID analytics | Backend and console 0.1.1 are prepared: aggregates by model, OS/app version and UTC day; 1/7/30-day and platform filters. A newly installed/published iOS client has not yet been verified for this scope. Android has not yet been ported. |
| Provider-specific push, closed stories, API auto-selection | Not ready for general use. Existing Xray JSON balancers are a separate feature. |
| Network map | Closed testing. Remnawave is connected manually with read-only access; custom panels need a separate connector. Uses aggregate data, not `geoip.dat` / `geosite.dat`, and does not expose raw IPs or exact device locations. A provider ID alone does not enable it. |

Stories are public: never include secret links, customer details or paid-subscriber-only material.

Existing analytics grouped by `profile-title` and `support-url` belong to the DoDo owner. Matching a name or support contact does not grant a provider access to those records. Provider-ID reports require the verified exact host and service approval.

Counts represent installations that sent reports, not people online or paying subscribers. A single installation may contain several providers. The console receives aggregates, not individual keys, URLs, HWIDs or a device directory. Reporting is limited to two attempts per 24 hours, at least 12 hours apart.

A console/API outage must not stop configuration import or VPN traffic. After a timed-out write, refresh the section and check its result before retrying. Early access is not a claim of an already completed real-provider pilot or unlimited capacity.

See also: [Integration](integration.md) · [Diagnostics and analytics](diagnostics.md) · [Compatibility](compatibility.md)

Source: [Provider registration](https://dodoproxy.app/docs/#provider-registration) · [Self-hosted console](https://dodoproxy.app/docs/#provider-console) · [Provider features](https://dodoproxy.app/providers/)
