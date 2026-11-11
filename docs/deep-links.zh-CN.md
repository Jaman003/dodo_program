[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](deep-links.md) · [Русский](deep-links.ru.md) · [فارسی](deep-links.fa.md) · [简体中文](deep-links.zh-CN.md)

# 机器人与网站的深度链接

使用 `dodoapp://import?url=…` 打开 DoDo，并让用户选择添加配置。深度链接不等于登录，也不会创建 VPN 访问权限；服务商必须先向该用户签发有效地址。

下列移动端说明以 iOS 2.2.0 和 Android APK 2.2.0 (2107) 为基准。不要据此推断 macOS 或 Windows 支持所有命令，请查看[平台兼容性](compatibility.zh-CN.md)。

## 推荐的导入链接

```text
dodoapp://import?url=https%3A%2F%2Fexample.com%2Fsub%2Fdemo-token
```

此例的原始地址为 `https://example.com/sub/demo-token`，仅为示例，不是真实密钥。

| 格式 | 用途 |
|---|---|
| `dodoapp://import?url=…` | 推荐的导入命令。 |
| `dodoapp://import-sub?url=…` | 支持的导入别名。 |
| 用 `link` 替代 `url` | 支持的参数别名；只传入一个非空目标参数。 |
| `dodoapp://crypt1/<JWE>` | 导入 DoDo 加密链接，详见 [crypt1 与 AGE1](crypto.zh-CN.md)。 |

服务商集成时请使用完整 HTTPS 地址。不要同时传入相互冲突的 `url` 和 `link`、重复目标参数，也不要把其他应用命令作为导入目标。

## 每层包装只编码一次

将原始 URL 视为不可擅自修改的字符串。把它放进深度链接的 query 参数时，只进行一次编码。不要再次解码或重建路径、令牌、`%2F`、`%252F`、`+` 或原始 query。

```js
const source = 'https://example.com/sub/demo-token?format=json&value=a%2Fb+test';
const deepLink = `dodoapp://import?url=${encodeURIComponent(source)}`;

// Only when your own HTTPS landing page accepts a deep link in its url parameter:
const landingLink = `https://example.com/connect?url=${encodeURIComponent(deepLink)}`;
```

第二行增加深度链接所需的一层编码。最后一行则为外层网页地址增加独立的一层编码。中转页只应去掉属于自己的那一层，再打开深度链接；反复解码整个字符串可能改变用户的密钥。

避免使用会把未转义 `+` 变为空格的 `application/x-www-form-urlencoded` 处理方式。`encodeURIComponent` 会将加号本身编码为 `%2B`。

## 旧格式与不支持的形式

| 格式 | 处理方式 |
|---|---|
| `dodoapp://add/<URL>` | 不要使用，`add` 不是已公布的导入命令。 |
| `dodoapp://import/<URL>` | 不要将目标放在路径中。改用 `import?url=` 并对目标编码一次。 |
| `dodo://import…` | 不支持配置导入。`dodo` scheme 用于下文的路由命令。 |
| `happ://…`，包括 `happ://crypto…` | 不是 DoDo 命令。临时 HTTP/JSON 兼容不代表注册其他应用的 scheme，也不代表能解密其私有格式。 |

机器人客户端映射中只保留一个 `dodo` 项。重复字典键可能覆盖前一个模板。

## 路由使用独立命令

```text
dodo://routing/add/<BASE64URL_UTF8_JSON>
dodo://routing/onadd/<BASE64URL_UTF8_JSON>
dodo://routing/off
```

这些路由命令也接受 `dodoapp` scheme。内容必须是 **DoDo 规则配置**，而非完整 Xray 配置。路径适合使用无末尾 `=` 的 Base64URL；普通 Base64 需要额外进行一次百分号编码。

添加的规则在下次连接时生效。`off` 会停用已导入规则，但不会删除配置。字段、远程更新与平台限制见[路由与地理数据](routing.zh-CN.md)。

## VPN 命令的平台差异

| 命令 | iOS | Android APK 2.2.0 |
|---|---|---|
| `dodoapp://toggle-vpn` | 切换当前 VPN 状态，再次打开会反向切换。 | 不可用。 |
| `dodoapp://vpn-permission` | 显示说明，不会绕过或自动弹出 Apple 系统授权。 | 打开 Android VPN 说明与授权流程。 |
| `dodoapp://vpn-disclosure` | 授权说明命令的别名。 | Android 说明与授权流程。 |

切换命令并非幂等的“连接”命令，不要在网页加载或超时后自动重试。

## 发布机器人按钮前的检查

1. 安装 DoDo，从用户实际使用的浏览器或聊天软件打开推荐链接。
2. 使用包含 query、`%` 转义和 `+` 的地址测试，保持原始字符串不变。
3. 检查配置数量、元数据、更新和实际 VPN 连接；仅能打开应用不证明隧道正常。
4. 测试取消与重复打开，不要记录真实源地址或 crypt1 链接。

DoDo 必须已经安装。通过 App Store 或 Google Play 安装后不会自动保留这次导入动作，用户需要再次打开原链接。深度链接本身不能识别 Telegram 账号。剪贴板只在用户主动操作时读取，不会静默拦截。

较长 crypt1 链接可能生成过于密集的二维码。部分 HUAWEI 设备的 Android 扫码存在已知限制，请同时提供复制文本与手动输入。

相关文档：[服务商集成](integration.zh-CN.md) · [crypt1 与 AGE1](crypto.zh-CN.md) · [备用地址](fallback.zh-CN.md) · [诊断](diagnostics.zh-CN.md)

官方参考：[从机器人和网站打开 DoDo](https://dodoproxy.app/docs/#deeplinks)。
