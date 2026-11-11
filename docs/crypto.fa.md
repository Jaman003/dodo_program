[DoDo APP](../README.fa.md) · [مستندات](README.fa.md)

[English](crypto.md) · [Русский](crypto.ru.md) · [فارسی](crypto.fa.md) · [简体中文](crypto.zh-CN.md)

<div dir="rtl">

# پیوند محافظت‌شده: DoDo crypt1 و AGE1

**crypt1 نشانی HTTPS کاربر را رمزگذاری می‌کند. AGE1 بدنهٔ پاسخ سرور را رمزگذاری می‌کند.** این دو مستقل‌اند؛ هیچ‌کدام به دیگری، طرح پولی یا `providerid` نیاز ندارد.

این راهنما پشتیبانی iOS 2.2.0 و Android APK 2.2.0 (2107) را توصیف می‌کند. در Android از کلیپ‌بورد یا ورود دستی استفاده کنید؛ اسکن QR متراکم crypt1 در بعضی HUAWEIها محدودیت شناخته‌شده دارد. این قابلیت را برای نسخهٔ منتشرشدهٔ macOS فرض نکنید؛ انتقال جدید Mac و پشتیبانی Windows کار جداگانه است. [سازگاری](compatibility.fa.md) را ببینید.

## شروع بدون تغییر پنل

1. نشانی HTTPS موجودِ پیکربندی یک کاربر را بردارید.
2. [سازندهٔ مرورگر](https://dodoproxy.app/crypto/) را باز کنید، نشانی را وارد کنید و سیاست پاسخ را روی خودکار نگه دارید.
3. پیوند `dodoapp://crypt1/<JWE>` را فقط به همان کاربر بدهید.
4. آن را در DoDo باز و وارد کردن را تأیید کنید؛ سپس تعداد پروفایل‌ها، به‌روزرسانی و اتصال واقعی VPN را بررسی کنید.

سازنده از WebCrypto به‌صورت محلی استفاده می‌کند؛ URL به API رمزگذاری فرستاده و در cookies یا فضای ذخیره‌سازی مرورگر ثبت نمی‌شود. برنامه پوشش را محلی رمزگشایی می‌کند و تنها پس از تأیید به منبع وصل می‌شود. **دریافت و به‌روزرسانی خود پیکربندی همچنان به شبکه و سرور ارائه‌دهنده نیاز دارد.**

Remnawave یا پنل اختصاصی می‌تواند قالب معمول خود را نگه دارد: فهرست URI، Base64، پیکربندی کامل Xray JSON یا آرایهٔ پروفایل‌ها. crypt1 تنها URL را می‌پوشاند؛ متعادل‌کننده را به سرورهای جدا تبدیل نمی‌کند و جای تجزیه‌گر را نمی‌گیرد.

## ساخت پیوند در ربات

[ابزار عمومی ارائه‌دهنده](https://dodoproxy.app/crypto/DoDo-crypt1-provider-tools.zip) شامل CLI، ماژول سازنده، manifest عمومی و مشخصات را دریافت کنید. Node.js 24+ لازم است؛ SDK بستهٔ npm لازم ندارد و هنگام ساخت، کلیدها را از شبکه دریافت نمی‌کند.

```sh
# Run inside the unpacked public tools directory.
# subscription-url.txt contains one user's HTTPS URL.
node cli/dodo-crypt1.mjs --name "Demo Provider" < subscription-url.txt

# Or provide the full payload:
node cli/dodo-crypt1.mjs --input-json < payload.json
```

نشانی واقعی را در آرگومان فرمان، تاریخچهٔ ترمینال یا لاگ ربات قرار ندهید. پیوند خروجی نیز مجوز دسترسی است، نه محتوای تبلیغاتی عمومی.

```js
import { readFile } from 'node:fs/promises';
import { createCrypt1Link } from './generator/crypt1.mjs';

const publicManifest = JSON.parse(
  await readFile(new URL('./keys/public-keys.json', import.meta.url), 'utf8')
);

const link = await createCrypt1Link({
  v: 1,
  type: 'subscription-url',
  url: 'https://example.com/sub/demo-token?format=json',
  response: 'auto',
  name: 'Demo Provider',
}, publicManifest);

// Send link privately to its owner; do not add it to shared logs.
```

خروجی `createCrypt1Link` از نوع `Promise<string>` است. از `activeKid` در manifest استفاده کنید؛ یک شناسه را برای همیشه ثابت نکنید. نسخهٔ محلی [manifest عمومی](https://dodoproxy.app/crypto/keys/public-keys.json) را هنگام تغییر کلیدهای DoDo به‌روز کنید، نه برای هر درخواست کاربر. ارائه‌دهنده به کلید خصوصی DoDo نیاز ندارد.

## قرارداد payload

| فیلد | الزامی | مقدار |
|---|---|---|
| `v` | بله | عدد `1`. |
| `type` | بله | رشتهٔ `subscription-url`. |
| `url` | بله | نشانی کامل و اصلی HTTPS؛ حداکثر 4096 بایت UTF-8. |
| `response` | بله | `auto` یا `age1-required`؛ رمزگشا مقدار پیش‌فرض ضمنی ندارد. |
| `name` | خیر | نام نمایشی غیرخالی، بدون فاصلهٔ ابتدا/انتها؛ حداکثر 120 بایت UTF-8؛ بدون نویسهٔ کنترلی/قالب‌بندی. هویت ارائه‌دهنده نیست. |

کل JSON سریال‌شده، با احتساب escapeها، باید در **8192 بایت UTF-8** بگنجد. فیلد ناشناخته، نام تکراری JSON و نوع اشتباه رد می‌شود.

نام DNS عمومی‌نما و در صورت نیاز پورت 1–65535 لازم است. اطلاعات ورود داخل URL، fragment، فاصله/نویسهٔ کنترلی، backslash، IP صریح، نام محلی و نام مستعار شناخته‌شدهٔ IP محلی پذیرفته نمی‌شود. این فقط بررسی ساختار است و عمومی بودن پاسخ DNS را ثابت نمی‌کند. `example.com` در مثال، منبع واقعی نیست.

رشتهٔ اصلی URL را حفظ کنید. `%2F` یا `%252F` را دوباره باز نکنید، `+` را تغییر ندهید، توکن را به میزبان دیگری نبرید و URL را با normalizer بازسازی نکنید. [کدگذاری پیوند عمیق](deep-links.fa.md) را ببینید.

## قالب برای پیاده‌سازی مستقل

```text
dodoapp://crypt1/<protected>.<encrypted_key>.<iv>.<ciphertext>.<tag>
```

از JWE Compact Serialization با دقیقاً پنج بخش غیرخالی استفاده کنید. هر بخش Base64URL استاندارد و بدون padding است (`A–Z`، `a–z`، `0–9`، `-`، `_`)؛ `=`، فاصله، خط جدید و percent-encoding ممنوع است. خود پیوند crypt1 نباید query، fragment یا بخش مسیر اضافی داشته باشد. حداکثر اندازهٔ پیوند **16,384 بایت UTF-8** است.

هدر محافظت‌شده فقط چهار فیلد رشته‌ای الزامی دارد:

```json
{"alg":"RSA-OAEP-256","enc":"A256GCM","typ":"dodo-subscription+jwe","kid":"<activeKid from the public manifest>"}
```

| پارامتر | الزام |
|---|---|
| RSA | 3072 بیت، exponent برابر 65537. |
| رمزگذاری کلید | RSA-OAEP-256 با SHA-256 و MGF1 SHA-256؛ label خالی. |
| CEK | 32 بایت تصادفی تازه برای هر پیوند. |
| رمزگذاری محتوا | AES-256-GCM؛ IV تصادفی 12 بایتی؛ authentication tag به اندازهٔ 16 بایت. |
| AAD | بایت‌های ASCII اولین بخش **کدگذاری‌شدهٔ** JWE. |
| کلید رمز‌شده | 384 بایت. |

`zip`، `crit`، کلید داخل هدر و URL دریافت کلید پشتیبانی نمی‌شوند. `kid` ناشناخته محلی رد می‌شود؛ برنامه از نشانی داخل هدر کلید دانلود نمی‌کند. نام تکراری JSON، حتی با escape معادل، نامعتبر است. در JVM، MGF1 SHA-256 را صریح تعیین کنید؛ نام cipher به‌تنهایی آن را در همهٔ پیاده‌سازی‌ها تضمین نمی‌کند.

ساخت دوباره برای همان URL، ciphertext متفاوت می‌دهد. DoDo منبع اصلیِ تأییدشده را تطبیق می‌دهد، نه اینکه برای هر ciphertext تصادفی کارت تازه بسازد. کپی و اشتراک‌گذاری از پیوند اصلی crypt1 استفاده می‌کند، نه URL بازشده. پیوند بلند ممکن است در QR جا نشود؛ ارسال متنی را نیز فراهم کنید.

## AGE1: رمزگذاری جداگانهٔ پاسخ سرور

AGE1 از age file format v1 و **X25519 recipients** استفاده می‌کند، نه کلید RSAِ crypt1. گیرندهٔ DoDo را از `ageRecipients` در manifest عمومی بردارید.

| سیاست داخل crypt1 | پاسخ پذیرفته‌شده |
|---|---|
| `auto` | پاسخ بازِ پشتیبانی‌شده یا AGE1 معتبر. AGE شناسایی‌شده ولی خراب به‌عنوان متن باز دوباره تجزیه نمی‌شود. |
| `age1-required` | AGE1 معتبر در هر بار دریافت، از جمله پشتیبان. پاسخ باز رد می‌شود. |

انتخاب `age1-required` رمزگذاری سرور را **فعال نمی‌کند**. ابتدا منبع را تنظیم کنید، سپس پیوند اجباری AGE1 بدهید. DoDo هدر مخصوصی برای فعال کردن AGE نمی‌فرستد؛ `response` سیاست پذیرش محلی است، نه فرمان سرور. روشن‌ترین روش، endpoint جداگانهٔ HTTPS است که همیشه AGE1 برمی‌گرداند.

نمونه با CLI رسمی `age` و `jq`:

```sh
jq -er '.ageRecipients[0].recipient' keys/public-keys.json > age-recipients.txt

age --encrypt --recipients-file age-recipients.txt \
  --output subscription.json.age subscription.json

# Alternative: ASCII armor instead of a binary response.
age --encrypt --armor --recipients-file age-recipients.txt \
  --output subscription.json.age.armor subscription.json
```

فایل را مستقیماً در بدنهٔ HTTP برگردانید. برای age باینری معمولاً `application/octet-stream` و برای ASCII armor نوع `text/plain` مناسب است. تشخیص از ابتدای بدنه انجام می‌شود، نه تنها Content-Type. پوشش JSON، یک لایهٔ Base64 اضافه، BOM یا HTML قرار ندهید.

فراداده بیرون بدنهٔ رمز‌شده، در هدرهای واقعی HTTP می‌ماند:

```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Cache-Control: private, no-store
profile-title: Demo Provider
profile-update-interval: 2
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/demo-token
```

قالب باینری `age-encryption.org/v1` و ASCII armor با `-----BEGIN AGE ENCRYPTED FILE-----` پشتیبانی می‌شوند. حالت passphrase/scrypt، SSH، افزونه و فایل تنها برای گیرندهٔ برنامهٔ دیگر پشتیبانی نمی‌شود. بدنهٔ رمز‌شدهٔ AGE حداکثر **2 MiB** است؛ سقف زمان، اندازه و دفعات بارگذار نیز اعمال می‌شود. در نسخه‌های موبایلِ مستندشده، زمان کل عملیات 25 ثانیه است؛ سقف پاسخ/عملیات در iOS برابر 8,000,000 / 12,000,000 بایت و در Android برابر 2 / 4 MiB است. سقف هر پروفایل JSON در iOS نیز 2,000,000 بایت است.

در `age1-required`، منبع اصلی و [پشتیبان](fallback.fa.md) هر دو باید AGE1 بدهند. خطای صحت، کلید یا قالب نباید با استخراج URI از بدنهٔ نامعتبر دور زده شود.

## امنیت و چک‌لیست

crypt1 **DRM یا اثبات هویت ارائه‌دهنده نیست**. فردی با بررسی هدفمند کلاینت مورد اعتماد می‌تواند دادهٔ لازم برای رمزگشایی را بازیابی کند. نشانی اصلی و پیوند محافظت‌شده را محرمانه نگه دارید. HTTPS، احراز هویت مشترک، لغو دسترسی و محدودیت دستگاه باید در سرور باقی بماند؛ توکن مدیریتی یا گذرواژهٔ سرور را در پوشش قرار ندهید.

- پروفایل کامل و متعادل‌کننده را بدون تغییر معنای آن آزمایش کنید.
- گیرندهٔ اشتباه، لینک/بدنهٔ خراب و شناسهٔ کلید ناشناخته باید رد شوند.
- در `age1-required`، پاسخ بازِ منبع اصلی و پشتیبان باید رد شود.
- وارد کردن دوباره، تازه‌سازی و اشتراک‌گذاری را بررسی کنید؛ خطای به‌روزرسانی نباید پیکربندی کارکردهٔ قبلی را پاک کند.
- ترافیک واقعی VPN را جدا از موفقیت رمزگشایی و وارد کردن آزمایش کنید.

DoDo crypt1/AGE1 قالب‌های اختصاصی Happ یا INCY را **رمزگشایی نمی‌کند**. پشتیبانی پاسخ سازگار HTTP/JSON قابلیت دیگری است.

مرتبط: [اتصال ارائه‌دهنده](integration.fa.md) · [پیوند عمیق](deep-links.fa.md) · [نشانی پشتیبان](fallback.fa.md) · [پروتکل‌ها](protocols.fa.md)

مرجع اصلی: [DoDo crypt1، سازنده و راه‌اندازی AGE1](https://dodoproxy.app/docs/#crypt1). استانداردها: [JWE](https://www.rfc-editor.org/rfc/rfc7516)، [JWA](https://www.rfc-editor.org/rfc/rfc7518)، [age v1](https://c2sp.org/age).

</div>
