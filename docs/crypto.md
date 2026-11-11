[DoDo APP](../README.md) · [Docs](README.md)

[English](crypto.md) · [Русский](crypto.ru.md) · [فارسی](crypto.fa.md) · [简体中文](crypto.zh-CN.md)

# Protected links: DoDo crypt1 and AGE1

**crypt1 encrypts a user's HTTPS source URL. AGE1 encrypts the response body.** They are independent: neither one requires the other, a paid plan, or a `providerid`.

This guide describes support in iOS 2.2.0 and Android APK 2.2.0 (2107). Use clipboard or manual import on Android: dense crypt1 QR codes have a known scanning limitation on some HUAWEI devices. Do not assume support in the published macOS version; the new Mac port and Windows support are separate work. See [compatibility](compatibility.md).

## Start without changing your panel

1. Take the existing HTTPS source URL issued to one user.
2. Open the [browser generator](https://dodoproxy.app/crypto/), paste it, and keep the response policy set to automatic.
3. Give the resulting `dodoapp://crypt1/<JWE>` link only to that user.
4. Open it in DoDo, confirm the import, then check profile count, updates and an actual VPN connection.

The generator uses WebCrypto locally: the entered URL is not sent to an encryption API or stored in cookies/browser storage. The app decrypts the envelope locally and contacts the source only after confirmation. **Loading and updating the configuration still require the provider's server and a network connection.**

Remnawave and custom panels can keep their normal source format: URI list, Base64, complete Xray JSON or an array of profiles. crypt1 wraps the URL; it does not flatten a balancer into individual servers or replace the configuration parser.

## Issue links from a bot

Download the public [provider tools](https://dodoproxy.app/crypto/DoDo-crypt1-provider-tools.zip). The package provides a CLI, generator module, public manifest and specification. Node.js 24+ is required; the SDK needs no npm packages and does not fetch keys during generation.

```sh
# Run inside the unpacked public tools directory.
# subscription-url.txt contains one user's HTTPS URL.
node cli/dodo-crypt1.mjs --name "Demo Provider" < subscription-url.txt

# Or provide the full payload:
node cli/dodo-crypt1.mjs --input-json < payload.json
```

Do not put a real source URL in shell arguments, command history or bot logs. The resulting protected link is also a credential, not public marketing content.

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

`createCrypt1Link` returns `Promise<string>`. Use the manifest's `activeKid`; do not pin one key ID forever. Update your local copy of the [public manifest](https://dodoproxy.app/crypto/keys/public-keys.json) when DoDo rotates keys, not on every user request. Providers need no DoDo private keys.

## Payload contract

| Field | Required | Value |
|---|---|---|
| `v` | Yes | Number `1`. |
| `type` | Yes | String `subscription-url`. |
| `url` | Yes | Original full HTTPS URL, at most 4096 UTF-8 bytes. |
| `response` | Yes | `auto` or `age1-required`; no implicit decoder default. |
| `name` | No | Non-empty trimmed display name, at most 120 UTF-8 bytes, no control/format characters. Not a provider identity. |

The complete serialized payload must fit within **8192 UTF-8 bytes**, including JSON escaping. Unknown fields, duplicate JSON names and wrong types are rejected.

Use a public DNS hostname and, if needed, a port from 1–65535. URL credentials, fragments, spaces/control characters, backslashes, IP literals and local names/known local-IP aliases are not accepted. This syntax check does not prove that a DNS answer is public. The `example.com` address above is illustrative, not a working subscription.

Keep the original URL string intact. Do not decode `%2F` or `%252F`, change `+`, move tokens to another host or reconstruct the URL through a normalizer. See [deep-link encoding](deep-links.md).

## Wire format for independent implementations

```text
dodoapp://crypt1/<protected>.<encrypted_key>.<iv>.<ciphertext>.<tag>
```

Use JWE Compact Serialization with exactly five non-empty segments. Each is canonical unpadded Base64URL (`A–Z`, `a–z`, `0–9`, `-`, `_`), without `=`, whitespace, line breaks or percent-encoding. The crypt1 link itself has no query, fragment or extra path segment. Maximum link size: **16,384 UTF-8 bytes**.

The protected header has only four required string fields:

```json
{"alg":"RSA-OAEP-256","enc":"A256GCM","typ":"dodo-subscription+jwe","kid":"<activeKid from the public manifest>"}
```

| Parameter | Requirement |
|---|---|
| RSA | 3072 bits, exponent 65537. |
| Key encryption | RSA-OAEP-256; SHA-256 and MGF1 SHA-256; empty label. |
| CEK | Fresh random 32 bytes for every link. |
| Content encryption | AES-256-GCM; fresh 12-byte IV; 16-byte authentication tag. |
| AAD | ASCII bytes of the first **encoded** JWE segment. |
| Encrypted key | 384 bytes. |

`zip`, `crit`, embedded keys and remote key URLs are not supported. Unknown `kid` values fail locally; the app does not download keys from a JWE header. Duplicate JSON names, including escaped equivalents, are invalid. On JVM, explicitly select MGF1 SHA-256; the cipher name alone does not guarantee it across providers.

Generating the same URL again produces a different ciphertext. DoDo matches the validated underlying source rather than creating a new card for every random ciphertext. Sharing/copying a protected card uses the original crypt1 link, not the decrypted URL. Very long links may not fit a QR code: offer text sharing too.

## AGE1: encrypt the server response separately

AGE1 uses age file format v1 with **X25519 recipients**, not the RSA key used by crypt1. Obtain a DoDo recipient from `ageRecipients` in the public manifest.

| Policy inside crypt1 | Accepted response |
|---|---|
| `auto` | A supported unencrypted response or valid AGE1. Recognized but damaged AGE is not retried as plaintext. |
| `age1-required` | Valid AGE1 on every load, including fallback. Plaintext is rejected. |

Setting `age1-required` does **not** turn on encryption at the provider. Configure the source first, then issue links requiring AGE1. DoDo does not send a special request header to enable AGE; `response` is a local acceptance policy, not a server command. A dedicated HTTPS endpoint that always returns AGE1 is the clearest setup.

Using the official `age` CLI and `jq`:

```sh
jq -er '.ageRecipients[0].recipient' keys/public-keys.json > age-recipients.txt

age --encrypt --recipients-file age-recipients.txt \
  --output subscription.json.age subscription.json

# Alternative: ASCII armor instead of a binary response.
age --encrypt --armor --recipients-file age-recipients.txt \
  --output subscription.json.age.armor subscription.json
```

Return the file directly as the HTTP body. Binary age v1 is normally `application/octet-stream`; ASCII armor can use `text/plain`. Detection uses the body prefix, not Content-Type alone. Do not add a JSON wrapper, an extra Base64 layer, BOM or HTML.

Metadata remains in real HTTP headers outside the encrypted body:

```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Cache-Control: private, no-store
profile-title: Demo Provider
profile-update-interval: 2
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/demo-token
```

Supported AGE forms are binary `age-encryption.org/v1` and ASCII armor `-----BEGIN AGE ENCRYPTED FILE-----`. Passphrase/scrypt, SSH, plugins and files encrypted only for another app's recipient are not supported. The encrypted AGE body has a **2 MiB** limit; ordinary loader time/size/attempt budgets also apply. In the documented mobile baseline, the operation budget is 25 seconds; iOS response/operation limits are 8,000,000 / 12,000,000 bytes, Android limits are 2 / 4 MiB. An individual iOS JSON profile is limited to 2,000,000 bytes.

For `age1-required`, both primary and [fallback](fallback.md) must return AGE1. Authentication, corruption or format errors must not be worked around by extracting individual URI strings from a failed body.

## Security and acceptance checklist

crypt1 is **not DRM or proof of provider identity**. A sufficiently motivated person can inspect the trusted client and recover its decryption material. Treat both the original URL and protected link as secrets. Keep HTTPS, subscriber authorization, revocation and device limits on your server; do not place administrative tokens or server passwords inside the envelope.

- Test full profiles and balancers without changing their semantics.
- Test the wrong recipient, a damaged link/body and an unknown key ID: these must fail.
- Ensure plaintext is rejected under `age1-required`, including at fallback.
- Check repeat import, refresh and sharing; a failed update must not erase the existing working configuration.
- Verify actual VPN traffic separately from successful decryption and import.

DoDo crypt1/AGE1 does **not** decrypt private Happ or INCY formats. Supporting a compatible HTTP/JSON response is a different feature.

Related: [Provider integration](integration.md) · [Deep links](deep-links.md) · [Fallback addresses](fallback.md) · [Protocols](protocols.md)

Canonical reference: [DoDo crypt1, generator and AGE1 setup](https://dodoproxy.app/docs/#crypt1). Standards: [JWE](https://www.rfc-editor.org/rfc/rfc7516), [JWA](https://www.rfc-editor.org/rfc/rfc7518), [age v1](https://c2sp.org/age).
