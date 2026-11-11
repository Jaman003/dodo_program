# Ping, logs and analytics

[DoDo APP](../README.md) · [Documentation](README.md)

[English](diagnostics.md) · [Русский](diagnostics.ru.md) · [فارسی](diagnostics.fa.md) · [简体中文](diagnostics.zh-CN.md)

## What does ping measure?

These modes are documented for iOS 2.2.0 and Android APK 2.2.0 (2107). This does not imply identical behavior on every platform.

| Mode | Measurement | Limitation |
| --- | --- | --- |
| TCP, the default | Time to establish a TCP connection to the server, without division. | An open port does not prove authentication, internet access or VPN speed. |
| DoDo Ping | HTTP GET through a supported profile; displayed result divided by 3.3. | An estimate, not raw measured latency. |
| HTTP GET / HEAD | The selected method through the profile to an HTTPS test URL, without division. | Measures a request, not bandwidth. |
| ICMP | A host's ICMP response time. | ICMP can be blocked while VPN service works. |

Hysteria uses UDP, so TCP mode uses an HTTP check through its profile. Balancers and complex or ambiguous profiles also require a full-profile check, not a direct-port result presented as proof of the whole chain. For Android's full-profile checks, core startup and local authentication consume the overall timeout but are excluded from the displayed HTTP request time.

In iOS, open **Settings → For experienced users → Ping**. You can set an HTTPS test URL and a timeout of 1–10 seconds. Checks are concurrency-limited and do not reconnect the current VPN. A JSON balancer is tested through its profile and selection rules.

Raw measurements are stored separately. Changing the method starts a suitable check rather than labelling an old method's value as new. Updating a configuration keeps the last value for matching servers. `N/A` means the check failed or is unavailable; `…` means no result exists yet for the selected method.

A successful probe is not a complete VPN/TUN/DNS/routing test or a speed test. If a site fails while ping succeeds, report the protocol, error code and steps; do not disable TLS validation.

## Local logs and automatic reports

In iOS, **Settings → For experienced users → Logs** contains enable, live view, pause, copy and clear controls. Review logs before sharing: technical server addresses may remain in them.

Automatic technical reports are separate from local logs. Sending begins after user consent; the app offers controls to disable sending, revoke consent and delete sent data.

Reports contain bounded technical details such as the error code and stage, device model, OS version and app version. They exclude secret configuration URLs, subscription HWIDs, complete VPN configurations and traffic contents. Reports go to the DoDo developer, not to the provider's console.

| Platform | Coverage | Limits |
| --- | --- | --- |
| iOS | Handled import/VPN errors, bounded queue, delivery and deletion controls; limited Apple MetricKit crash/hang indications. | Not full crash files or call stacks. Not every extension crash, memory termination or hang is captured. Apple delivery can arrive later, including after another launch. |
| Android APK | Consented reports, protected queue, sending while the app is running; loading details distinguish network, response and import failures. | No permanent sender after the app closes; not guaranteed to capture every system crash or hang. |

Notification permission is not diagnostic consent. Device headers/HWID sent to a provider when loading a configuration are another data flow; they are not DoDo diagnostic reports. A provider can receive a log only if the user separately shares it.

## Usage analytics are another feature

DoDo owner analytics group suitable public `profile-title` and sanitized `support-url` values with model, platform and app/OS version. These names are labels, not verified ownership. macOS and Windows are not yet connected to this usage reporting.

On iOS 2.2.0 and Android APK 2.2.0, controls are in the privacy policy screen. Existing refusals are preserved. Analytics are separate from consent to technical error reports.

Sending is limited to **two attempts per 24 hours, at least 12 hours apart**. A statistics backend outage does not block launch, import or VPN. Counts are reporting installations and configuration counts, not online users or unique people.

Detailed records are retained up to 30 days. Monthly history began on 9 November 2026; closed, anonymized aggregate summaries have no automatic expiry and hide small groups. There is no reconstructed history before that date. Withdrawal deletes individual data and recalculates the open month; a person cannot be extracted from a closed anonymous summary.

Provider dashboards use a separate verified provider-ID scope. Matching a title or support contact cannot expose old owner reports to a provider. The backend/console are prepared, but the new iOS rollout is not verified and Android has not been ported. See [provider console status](provider-console.md).

See also: [Compatibility](compatibility.md) · [Routing](routing.md) · [Feedback](updates-feedback.md)

Source: [Ping](https://dodoproxy.app/docs/#ping) · [Diagnostics](https://dodoproxy.app/docs/#diagnostics) · [Analytics](https://dodoproxy.app/docs/#analytics)
