[DoDo App](../README.md) · [Documentation](README.md)

English · [Русский](compatibility.ru.md) · [فارسی](compatibility.fa.md) · [简体中文](compatibility.zh-CN.md)

# Platform compatibility and feature status

DoDo versions do not gain features at the same time. A successful import, a successful VPN connection and availability in an app store are separate checks.

The feature baseline below is **iOS 2.2.0** and **Android APK 2.2.0 (2107)**. The iOS App Store listing checked on **11 November 2026** shows **2.2.1**, released on 9 November; it requires iOS 15 or later. Its release notes describe key loading/updating and some full-JSON import fixes. That does not confirm publication of every newer provider integration.

## Platforms

| Platform | Release and documentation scope |
| --- | --- |
| iPhone / iPad | App Store release; iOS 15+. Device-specific features can require a newer OS or compatible hardware. See the store for the current release. |
| Android | The documented downloadable APK is 2.2.0 (2107). Google Play availability is a separate release step. If installed from Google Play, update through the store: signing keys can differ. Do not delete your keys or application data to force an APK update. |
| macOS | The published Mac build and the newer native Mac build have different feature sets. The native redesign and core update are prepared separately; do not assume an unreleased feature is in the Mac App Store build. |
| Windows | In preparation. There is no public release to promise feature parity for yet. |

Download links and release announcements are on [the website](https://dodoproxy.app/). Use the installed version as the starting point when reporting a problem.

## Import, routing and daily use

“Limited” means the documented formats and restrictions apply, not support for every possible configuration.

| Feature | iOS baseline | Android APK baseline | macOS | Windows |
| --- | --- | --- | --- | --- |
| URI / Base64 / full Xray JSON | Available, with profile validation | Available; full linked profile is retained | Verify the installed Mac build and profile | In preparation |
| Full-JSON routing and balancers | Limited to supported profiles; not flattened into separate outbounds | Limited to recognized primary protocols and dependencies | New native implementation is a separate release | In preparation |
| DoDo crypt1 / AGE1 for DoDo | Available | Available by clipboard/manual entry; dense crypt1 QR scanning remains unreliable on some HUAWEI devices | Port to the new Mac version is not confirmed as published | Planned |
| `fallback-url` | Available after a successful response supplies the backup | Available; backup must be learned in advance | Not available in the documented published Mac baseline | In preparation |
| Key metadata and `profile-update-interval` | Available | Available | Check the installed release | In preparation |
| Routing / autorouting / bundled geodata | Available with format and refresh limits | Available with different refresh intervals | Do not assume mobile parity | In preparation |
| TCP, DoDo Ping, HTTP GET, HTTP HEAD, ICMP | Available | Available, with profile-specific fallback checks | Check the installed release | In preparation |
| MTU and IPv4 / IPv6 preference | Available | Available | Check the installed release | In preparation |
| Live logs, QR sharing and confirmed reset | Available | Available; crypt1 camera limitation above applies | Check the installed release and UI | In preparation |
| Consented technical error reports | Available, with coverage limits | Available, with coverage limits | Mobile delivery is not implied | In preparation |

See [integration](integration.md), [protocols](protocols.md), [routing](routing.md), [protected links](crypto.md), [fallback addresses](fallback.md) and [diagnostics](diagnostics.md) before enabling a feature for all customers.

## How to interpret ping

| Method | What the result means |
| --- | --- |
| TCP | Measured connection setup time, without division by 3.3. An open port does not prove that VPN authentication or internet access works. |
| DoDo Ping | An estimate derived from an HTTP GET through the supported profile, divided by 3.3. It is not the raw HTTP time. |
| HTTP GET / HEAD | Measured request time through the profile, without division. It is not a speed test. |
| ICMP | Echo response time; a provider may block ICMP while the VPN itself works. |

Hysteria uses UDP, so an ordinary TCP check is not suitable. DoDo uses a profile-based HTTP check where supported. In Android TCP mode, balancers and complex or ambiguous JSON profiles are also checked through the complete profile rather than an arbitrary outbound. Checks are bounded and do not reconnect the active VPN.

## Protocol and OS limits

- AmneziaWG is retained on Android and is **not supported on iOS**. It is not converted to ordinary WireGuard; do not assume it is present in the new native Mac build.
- Ordinary WireGuard support is limited to the documented URI/profile forms. “WireGuard supported” does not mean every peer layout or Amnezia extension is accepted.
- Full JSON support does not mean arbitrary Xray server configurations can run on a phone. Client inbounds and startup settings may need adaptation; protocol, transport, core version and geodata categories still matter.
- On iPhone, Live Activity / Dynamic Island needs iOS 16.1+, a compatible device, permission and the DoDo setting enabled. iOS 15 support does not include these system features.
- Camera permission is required for QR scanning. Manual input and clipboard remain useful alternatives, especially for long encrypted links.
- Happ crypto/deep links, Lite Mode, a general Premium API, provider-forced theme/button styling and a “No Limit Mode” are not supported by this documented integration.

## Provider features: what is ready and what is not

| Capability | Status and boundary |
| --- | --- |
| Ordinary key delivery | No DoDo account, private panel API token or provider module is required. HTTPS plus a supported response format is enough. |
| Google registration and exact-host DNS TXT | Free early access. Provider approval remains separate from domain control. |
| Self-hosted provider console | Early access. A limited client of the DoDo API, not a copy of the central admin or access to other providers. |
| Public provider brand and stories | Integration is in early access and requires an approved provider, a valid exact-host DNS confirmation, `x-dodo-provider-id` in a successful response, and a compatible newer iOS client. Do not assume that a published app version already includes every change. |
| Analytics by public title/support | Documented for iOS and Android. These are installation/use estimates, not a live user or subscriber count. |
| Provider-ID-scoped device reports | Separate from title/support grouping. iOS integration is being verified; Android provider-ID statistics have **not** been ported. Do not promise complete cross-platform provider reports yet. |
| Proof of a particular subscriber | An optional separate signed pilot. A public provider ID or DNS TXT does not prove that the user has paid or has an active account. |
| Provider-targeted push | Not available for general use. Public content binding does not create a private push audience. |
| Provider-API Auto-select | In development. Never put the panel's administrative API token into a user's key. |

The DoDo Auto-select button is temporarily hidden on iOS. This does not disable a provider's balancer inside a full Xray profile. See [provider console and registration](provider-console.md) for the distinction between public content and subscriber authorization.

## Before rolling out an integration

1. Identify the platform and installed build, not just the name “DoDo”.
2. Test import, update and a real connection with your exact profile, DNS and transport.
3. Test the backup address after the app has successfully learned it.
4. Keep a manual/clipboard path for protected links and an older-client path where necessary.
5. Check [release history and feedback](updates-feedback.md) before treating an early-access feature as generally available.

Import failure must not silently become a successful partial list. A failed key update keeps the previous usable servers. This protects data, but cannot repair an unreachable provider, invalid credentials or a broken server-side route.

Sources: [Platform status](https://dodoproxy.app/docs/#status), [Protocols](https://dodoproxy.app/docs/#protocols), [Ping](https://dodoproxy.app/docs/#ping), [Provider registration](https://dodoproxy.app/docs/#provider-registration), [Analytics](https://dodoproxy.app/docs/#analytics), [App Store](https://apps.apple.com/app/id6759006686).
