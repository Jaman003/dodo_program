# Updates and feedback

[DoDo APP](../README.md) · [Documentation](README.md)

[English](updates-feedback.md) · [Русский](updates-feedback.ru.md) · [فارسی](updates-feedback.fa.md) · [简体中文](updates-feedback.zh-CN.md)

## Release history

The [official update history](https://dodoproxy.app/updates/) contains platform filters and search. A source change, test build, signed download and store release are different stages; availability on one platform does not establish availability on another.

| Platform | Version and date | Changes and status |
| --- | --- | --- |
| iOS / iPadOS | [2.2.2 (113) · 12 November 2026](ios-2.2.2.md) | Uploaded to App Store Connect for processing; not yet submitted to App Review or announced as available in the store. Full JSON env/version, HTTP GET/HEAD, Hysteria2 geodata, DNS/limited FakeDNS, FinalMask and fragmentation. |
| iOS / iPadOS | [2.2.1 · 9 November 2026](https://apps.apple.com/app/id6759006686) | Improved adding and updating keys; fixed importing some complete Xray configurations. App Store metadata checked on 11 November 2026. |
| iOS / iPadOS | [2.2.0 · 6 November 2026](https://dodoproxy.app/updates/#ios-2-2-0) | Core and connection stability improvements; key import/update, full configurations, protected links, DNS/network settings and QR/UI fixes. Requires iOS 15 or later. |
| Android | [APK 2.2.0 (2107) · 10 November 2026](https://dodoproxy.app/updates/#android-2-2-0-2107) | Compact circular controls, revised animations and cards, visible loading, QR/clipboard shortcuts, raw TCP and full-profile checks, crypt1/AGE1, fallback, network settings and consented diagnostics. |

The Android entry is a signed APK release on the website, not confirmation of a new Google Play rollout. [Download APK 2.2.0 (2107)](https://dodoproxy.app/downloads/DoDo-APP-Android-2.2.0-2107.apk). If DoDo was installed from Google Play, update through the store: signatures can differ. Do not uninstall the app or erase configurations to bypass an APK signature mismatch.

On some HUAWEI devices, dense QR codes containing protected links still may not scan. Use clipboard or manual input. Android provider stories and targeted push are not yet enabled.

The new native macOS work is separate; do not infer a Mac App Store release from iOS's version. A refreshed Windows release is being prepared. The [compatibility page](compatibility.md) distinguishes available, partial and planned functionality. Provider registration and its console remain early access; release 2.2.1 alone does not confirm the new provider-ID reporting rollout.

## Ideas, bugs and integration requests

Use the [DoDo feedback board](https://dodoproxy.app/feedback/). Search existing posts first, filter by platform or status, and support an existing request if it describes your issue.

New submissions enter moderation and do **not** immediately appear on the public board. Publication requires your permission and a moderator's decision. The team can reply, change status or hide a post. Statuses are:

- New.
- Reviewing.
- Planned.
- In progress.
- Done.
- Closed.

These are workflow labels, not promised delivery dates. Support counts relate to browser identities, not verified unique people.

For a useful bug report, include:

1. Device model, OS version and DoDo version.
2. Exact steps and whether the issue repeats.
3. Expected and actual behavior.
4. Error code and approximate time, if available.
5. Provider name or bare domain only, if relevant.

Never submit working keys, secret access URLs, passwords, HWIDs, private keys or complete configurations. Redact screenshots and logs before sharing. The initial feedback form has no file uploads; describe the issue using demonstration data.

A key's payment, expiry, device limit or server availability is normally a question for its provider's support. Import failures, crashes and app-interface problems belong on DoDo's feedback board. Ping alone does not prove that a provider's entire connection works; see [diagnostics](diagnostics.md).

## Stay in touch

[DoDo community — @dododeveloper](https://t.me/dododeveloper) · [News — @dodoproxy](https://t.me/dodoproxy)

Source: [Update history](https://dodoproxy.app/updates/) · [Feedback board](https://dodoproxy.app/feedback/) · [App Store](https://apps.apple.com/app/id6759006686)
