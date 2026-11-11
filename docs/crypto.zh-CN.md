[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](crypto.md) · [Русский](crypto.ru.md) · [فارسی](crypto.fa.md) · [简体中文](crypto.zh-CN.md)

# 加密链接：DoDo crypt1 与 AGE1

**crypt1 加密用户的 HTTPS 源地址，AGE1 加密服务器响应体。** 两者相互独立，不要求对方存在，也不需要付费方案或 `providerid`。

本文以 iOS 2.2.0 和 Android APK 2.2.0 (2107) 为基准。Android 请使用剪贴板或手动输入：部分 HUAWEI 设备对密集 crypt1 二维码的识别存在已知限制。不要据此假设已发布 macOS 版本具有相同支持，新 Mac 移植与 Windows 支持属于独立工作。详见[兼容性](compatibility.zh-CN.md)。

## 无需改动面板即可开始

1. 取出已经签发给某一用户的 HTTPS 配置地址。
2. 打开[浏览器生成器](https://dodoproxy.app/crypto/)，粘贴地址，保持自动响应策略。
3. 将生成的 `dodoapp://crypt1/<JWE>` 链接私下交给该用户。
4. 在 DoDo 中打开并确认导入，再检查配置数量、更新和实际 VPN 连接。

生成器在本地使用 WebCrypto，不会将输入 URL 发送至加密 API，也不会保存到 cookies 或浏览器存储。应用在本地解密封装，仅在用户确认后请求源服务器。**配置本身的下载和更新仍需要网络与服务商服务器。**

Remnawave 或自建面板可以保留原有格式：URI 列表、Base64、完整 Xray JSON 或配置数组。crypt1 只是包装地址，不会把负载均衡器拆成独立服务器，也不取代原有解析器。

## 在机器人中生成链接

下载公开的[服务商工具包](https://dodoproxy.app/crypto/DoDo-crypt1-provider-tools.zip)，其中包含 CLI、生成器模块、公钥清单和规范。需要 Node.js 24+；SDK 无需 npm 依赖，生成时不会联网下载密钥。

```sh
# Run inside the unpacked public tools directory.
# subscription-url.txt contains one user's HTTPS URL.
node cli/dodo-crypt1.mjs --name "Demo Provider" < subscription-url.txt

# Or provide the full payload:
node cli/dodo-crypt1.mjs --input-json < payload.json
```

不要将真实地址放进命令行参数、终端历史或机器人日志。生成的加密链接同样是访问凭据，不是公开宣传内容。

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

`createCrypt1Link` 返回 `Promise<string>`。使用清单中的 `activeKid`，不要永久固定某个密钥 ID。DoDo 轮换密钥时更新本地[公钥清单](https://dodoproxy.app/crypto/keys/public-keys.json)，无需每次用户请求都下载。服务商不需要 DoDo 私钥。

## Payload 字段

| 字段 | 必填 | 值 |
|---|---|---|
| `v` | 是 | 数字 `1`。 |
| `type` | 是 | 字符串 `subscription-url`。 |
| `url` | 是 | 原始完整 HTTPS 地址，最多 4096 UTF-8 字节。 |
| `response` | 是 | `auto` 或 `age1-required`，解码器没有隐式默认值。 |
| `name` | 否 | 去掉首尾空格后的非空显示名称，最多 120 UTF-8 字节，不含控制/格式字符，不代表服务商身份。 |

序列化后的整个 JSON payload（含 JSON 转义）最多 **8192 UTF-8 字节**。未知字段、重复 JSON 键名和错误类型都会被拒绝。

需要看起来属于公共网络的 DNS 主机名，端口可选且为 1–65535。URL 内嵌账号密码、fragment、空格/控制字符、反斜杠、IP 字面量、本地名称与已知本地 IP 别名不被接受。这只是语法检查，不能证明 DNS 实际解析结果属于公共网络。上面的 `example.com` 仅为示例，并非真实配置。

保持原始 URL 字符串不变。不要再次解码 `%2F` 或 `%252F`，不要修改 `+`、把令牌转移到另一主机，也不要通过规范化器重建地址。参见[深度链接编码](deep-links.zh-CN.md)。

## 独立实现所需的格式

```text
dodoapp://crypt1/<protected>.<encrypted_key>.<iv>.<ciphertext>.<tag>
```

使用 JWE Compact Serialization，必须恰好有五个非空部分。每部分采用规范的无填充 Base64URL（`A–Z`、`a–z`、`0–9`、`-`、`_`），不得包含 `=`、空白、换行或百分号编码。crypt1 链接自身不能有 query、fragment 或额外路径。整条链接最多 **16,384 UTF-8 字节**。

受保护头部只能包含以下四个必填字符串字段：

```json
{"alg":"RSA-OAEP-256","enc":"A256GCM","typ":"dodo-subscription+jwe","kid":"<activeKid from the public manifest>"}
```

| 参数 | 要求 |
|---|---|
| RSA | 3072 位，指数 65537。 |
| 密钥加密 | RSA-OAEP-256，SHA-256 与 MGF1 SHA-256，空 label。 |
| CEK | 每条链接使用新的 32 字节随机密钥。 |
| 内容加密 | AES-256-GCM，新的 12 字节 IV，16 字节认证标签。 |
| AAD | 第一个**已编码** JWE 部分的 ASCII 字节。 |
| 加密后的密钥 | 384 字节。 |

不支持 `zip`、`crit`、内嵌密钥或远程密钥 URL。未知 `kid` 会在本地失败，应用不会按 JWE 头部地址下载密钥。重复 JSON 键名（包括转义后的等价键名）无效。JVM 实现应显式指定 MGF1 SHA-256，仅靠 cipher 名称不能保证所有密码提供者行为一致。

同一 URL 再生成时会产生不同密文。DoDo 根据验证后的原始来源识别配置，不会为每个随机密文都创建新卡片。复制与分享使用原 crypt1 链接，而非解密后的 URL。较长链接可能无法装入二维码，请同时提供文本分享。

## AGE1：单独加密服务器响应

AGE1 使用 age file format v1 和 **X25519 recipients**，不是 crypt1 的 RSA 密钥。从公钥清单的 `ageRecipients` 获取 DoDo 接收者。

| crypt1 内的策略 | 接受的响应 |
|---|---|
| `auto` | 支持的明文响应或有效 AGE1。已经识别但损坏的 AGE 不会再按明文解析。 |
| `age1-required` | 每次加载都必须是有效 AGE1，包括备用地址，明文被拒绝。 |

设置 `age1-required` **不会启用服务器加密**。请先配置源服务，再签发要求 AGE1 的链接。DoDo 不发送用于开启 AGE 的特殊请求头；`response` 是本地接收策略，不是服务器命令。最明确的做法是提供始终返回 AGE1 的独立 HTTPS 端点。

使用官方 `age` CLI 和 `jq`：

```sh
jq -er '.ageRecipients[0].recipient' keys/public-keys.json > age-recipients.txt

age --encrypt --recipients-file age-recipients.txt \
  --output subscription.json.age subscription.json

# Alternative: ASCII armor instead of a binary response.
age --encrypt --armor --recipients-file age-recipients.txt \
  --output subscription.json.age.armor subscription.json
```

直接将文件作为 HTTP 响应体返回。二进制 age v1 通常用 `application/octet-stream`，ASCII armor 可用 `text/plain`。格式根据响应体开头识别，不只依赖 Content-Type。不要添加 JSON 包装、额外 Base64、BOM 或 HTML。

元数据仍放在加密响应体以外的真实 HTTP 头部：

```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Cache-Control: private, no-store
profile-title: Demo Provider
profile-update-interval: 2
providerid: demo-vpn
fallback-url: https://backup.example.com/sub/demo-token
```

支持二进制 `age-encryption.org/v1` 与 ASCII armor `-----BEGIN AGE ENCRYPTED FILE-----`。不支持口令/scrypt、SSH、插件，以及仅加密给其他应用接收者的文件。AGE 加密响应体上限为 **2 MiB**，同时受加载器的时间、大小及尝试次数限制。在本文移动端基准中，整个操作限时 25 秒；iOS 响应/操作大小上限为 8,000,000 / 12,000,000 字节，Android 为 2 / 4 MiB；iOS 单个 JSON 配置上限为 2,000,000 字节。

`age1-required` 下，主地址与[备用地址](fallback.zh-CN.md)都必须返回 AGE1。不能通过从失败响应中提取个别 URI 来绕过认证、完整性或格式错误。

## 安全与验收检查

crypt1 **不是 DRM，也不证明服务商身份**。有能力深入研究可信客户端的人仍可能恢复解密所需材料。原始地址与加密链接都应保密。HTTPS、订阅用户认证、撤销访问和设备限制必须保留在服务器端；不要把管理令牌或服务器密码放进封装。

- 测试完整配置与负载均衡器，保持原本语义。
- 错误接收者、损坏链接/响应体和未知密钥 ID 必须失败。
- 检查 `age1-required` 拒绝主地址与备用地址的明文响应。
- 测试重复导入、更新和分享；更新失败不能删除原有可用配置。
- 实际 VPN 流量应单独测试，不能用成功解密或导入代替。

DoDo crypt1/AGE1 **不能解密** Happ 或 INCY 的私有格式。接受兼容 HTTP/JSON 响应是另一项功能。

相关文档：[服务商集成](integration.zh-CN.md) · [深度链接](deep-links.zh-CN.md) · [备用地址](fallback.zh-CN.md) · [协议](protocols.zh-CN.md)

官方参考：[DoDo crypt1、生成器与 AGE1 设置](https://dodoproxy.app/docs/#crypt1)。标准：[JWE](https://www.rfc-editor.org/rfc/rfc7516)、[JWA](https://www.rfc-editor.org/rfc/rfc7518)、[age v1](https://c2sp.org/age)。
