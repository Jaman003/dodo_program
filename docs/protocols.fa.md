[DoDo APP](../README.fa.md) · [مستندات](README.fa.md)

[English](protocols.md) · [Русский](protocols.ru.md) · [فارسی](protocols.fa.md) · [简体中文](protocols.zh-CN.md)

<div dir="rtl">

# پروتکل‌ها، انتقال و TLS

نام پروتکل به‌تنهایی سازگاری را تضمین نمی‌کند. ترکیب پروتکل، انتقال، تنظیمات امنیتی و هستهٔ داخلی باید با سرور یکسان باشد. جدول زیر قابلیت‌های iOS 2.2.0 را نشان می‌دهد؛ تفاوت‌های Android در ادامه آمده است.

## پروتکل‌های اتصال

| پروتکل | قالب ورودی | وضعیت و محدودیت |
| --- | --- | --- |
| VLESS | `vless://`، Xray JSON | پشتیبانی می‌شود؛ شامل پروفایل‌های سازگار TLS / REALITY |
| VMess | `vmess://` با Base64 JSON، Xray JSON | پشتیبانی می‌شود؛ فیلدهای انتقال و امنیت بررسی می‌شوند |
| Trojan | `trojan://`، Xray JSON | پشتیبانی می‌شود؛ TLS برای URI پیش‌فرض است |
| Shadowsocks | `ss://`، Xray JSON | شکل‌های رایج SIP002 / Base64؛ رمزها به هسته وابسته‌اند؛ افزونهٔ خارجی در URI پشتیبانی نمی‌شود |
| Hysteria2 | `hy2://`، `hysteria2://`، Xray JSON سازگار | نسخهٔ ۲؛ Salamander و پارامترهای hopping در قالب پشتیبانی‌شده |
| SOCKS5 | `socks://`، `socks5://`، Xray JSON | با یا بدون نام و رمز؛ SOCKS به‌تنهایی انتقال را رمزنگاری نمی‌کند |
| WireGuard معمولی | `wg://`، `wireguard://` | محدود: پارامترهای URI یا پروفایل wg-quick به‌صورت Base64 داخل URI؛ نه فایل دلخواه چند-peer و نه افزونه‌های AmneziaWG |
| AmneziaWG، TUIC، SSR، Hysteria v1 | طرح‌های اختصاصی آن‌ها | در iOS پشتیبانی نمی‌شوند؛ AmneziaWG به WireGuard عادی تبدیل نمی‌شود |

JSON کامل سازگار می‌تواند خروجی کمکی HTTP، SOCKS یا WireGuard داشته باشد؛ این به معنی امکان واردکردن هر طرح URI به‌طور مستقل نیست.

**Android APK:** برخلاف iOS، AmneziaWG در Android باقی مانده است. واردکردن و اجرای Hysteria2 پیاده‌سازی شده؛ پارامترهای واقعی سرورتان را آزمایش کنید. JSON کامل به پروتکل اصلی شناخته‌شده و خروجی‌های وابستهٔ سازگار نیاز دارد. پیش از ارائهٔ یک پروفایل به همهٔ دستگاه‌ها [جدول پلتفرم‌ها](compatibility.fa.md) را ببینید.

## انتقال‌ها و فیلدهای URI

| فیلد | مقدار یا کاربرد |
| --- | --- |
| `type` / `network` | `tcp`، `raw`، `ws`، `websocket`، `grpc`، `xhttp`، `splithttp`، `httpupgrade`، `kcp`، `mkcp` |
| نام جایگزین | `websocket` → WS؛ `splithttp` → XHTTP |
| `security` | `none`، `tls`، `reality`، فقط در ترکیب سازگار با پروتکل و هسته |
| TLS / REALITY | `sni` / `peer`، `fp` / `fingerprint`، `alpn`؛ برای REALITY نیز `pbk`، `sid`، `spx` |
| WS | `host`، `path` |
| gRPC | `serviceName`، `authority` |
| XHTTP | `host`، `path`، `mode`، شیء JSON با نام `extra` |
| بررسی گواهی | `pcs` / `pinnedPeerCertSha256`، `vcn` / `verifyPeerCertByName` |

مقادیر واقعی تنظیم‌شده روی سرور را بفرستید. برای تنظیمات تودرتوی انتقال، زنجیرهٔ خروجی‌ها، متعادل‌کننده یا DNS ارائه‌دهنده، [Xray JSON کامل](integration.fa.md) بهتر از URI کوتاه است.

**Hysteria2 از QUIC بدون `type=quic` استفاده می‌کند.** آن را با `hy2://` یا `hysteria2://` وارد کنید. مقادیر انتقال URI یعنی `quic`، `h2` و `http` در واردکنندهٔ iOS توصیف‌شده پشتیبانی نمی‌شوند. این محدودیت واردکننده است، نه نبود پروتکل‌های مبتنی بر QUIC در هسته.

## امنیت و گزینه‌های پیشرفته

- pin نامعتبر گواهی، الزام ECH پشتیبانی‌نشده یا درخواست دورزدن اعتبارسنجی گواهی باعث رد آن URI می‌شود، نه کاهش امنیت TLS.
- JSON کامل سازگار می‌تواند fragmentation در TLS، از جمله `freedom.settings.fragment`، داشته باشد؛ ویرایشگر مستقل fragmentation وجود ندارد.
- مقادیر `flow`، mux، سوکت و انتقال باید با سرور و هسته سازگار باشند. پشتیبانی VLESS تضمین همهٔ ترکیب‌های Vision / REALITY / XHTTP نیست.
- واردکردن موفق، دسترس‌پذیری سرور را ثابت نمی‌کند. [آزمایش HTTP و لاگ تشخیصی](diagnostics.fa.md) به تفکیک خطای دریافت، احراز هویت و شبکه کمک می‌کنند.

DoDo یک کلاینت است، نه فروشندهٔ دسترسی VPN. کلید و مشخصات سرور را از سرویس یا مدیر خود بگیرید.

منابع: [پروتکل‌های DoDo](https://dodoproxy.app/docs/#protocols)، [انتقال و TLS](https://dodoproxy.app/docs/#transports). برای معنای تنظیمات هسته، [مستندات رسمی Xray](https://xtls.github.io/en/config/) را ببینید.

</div>
