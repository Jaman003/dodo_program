# 服务商控制台

[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md)

[English](provider-console.md) · [Русский](provider-console.ru.md) · [فارسی](provider-console.fa.md) · [简体中文](provider-console.zh-CN.md)

## 注册服务

服务商注册免费开放，目前处于抢先体验阶段，完整正式发布仍在准备中。在 [DoDo Partners](https://api.dodoproxy.app/) 使用 Google 登录，创建服务并填写公开的支持联系方式。无需提供 Remnawave 密码、管理 API 令牌或 VPN 节点访问权限。

公开品牌和动态需要三项独立验证：

1. 所有者通过 Google 登录。
2. 使用 DNS TXT 验证发放配置的准确主机名。
3. 获得 DoDo 对服务的批准。

对于 `sub.example.com`，TXT 记录结构如下。请使用自己控制台生成的唯一值，不要直接复制占位符：

```text
_dodo-verify.sub.example.com
dodo-verification=v1:CODE_FROM_YOUR_ACCOUNT
```

验证仅适用于该主机，不覆盖全部子域名，也不覆盖共享 CDN 上的其他服务。有效期为 30 天，到期前 7 天会显示提醒。请获取新的 TXT 记录并手动完成续期；不会在后台自动续期。

## 返回服务商 ID

在已验证主机发放配置的成功 HTTPS 响应中加入真正的 HTTP 响应头：

```http
x-dodo-provider-id: YOUR_PROVIDER_UUID_FROM_DODO
```

不要将其写成 JSON 字段或正文注释。错误页面、重定向及其他主机的响应不能建立此关联。

该 UUID 是公开标识，不是 API 令牌、付费功能状态、付款证明或某位用户拥有配置的证明。它与[备用地址](fallback.zh-CN.md)使用的短 `providerid` 不同。公开品牌和动态不需要模块、signer 或私有签名密钥。仅限订阅者访问的内容需要单独验证访问权。

## 在自己的服务器部署

[下载 Provider Console 0.1.1](https://dodoproxy.app/providers/DoDo-Provider-Console-0.1.1.tar.gz) · [安装说明](https://dodoproxy.app/providers/DoDo-Provider-Console-0.1.1-README-RU.md)

软件包通过 Docker Compose 在 Linux 上运行，并在配置好的准确来源地址后使用 HTTPS 反向代理。请验证 `SHA256SUMS` 并遵循包内说明。它不需要 VPN 基础设施的 root 权限、host network、Docker socket 或 Remnawave 模块。

使用一次性私密设置代码，通过 Google 批准该安装，然后单独以所有者或编辑者身份登录。批准安装不能代替人员权限。每个安装获得可撤销、仅限本服务商的中央 DoDo API 访问权。软件包不包含主管理员界面、平台主密钥或 push 凭证。数据和已发布媒体存储在 DoDo 端。

## 动态和统计

| 功能 | 范围与状态 |
| --- | --- |
| 动态 | 草稿、预览、发布 12 或 24 小时、撤下和归档。需要有效 DNS 验证、DoDo 批准以及兼容的新 iOS 构建。抢先体验期间仅面向兼容客户端，不保证在所有已安装版本中显示。 |
| 媒体 | JPEG、PNG、WebP 最大 10 MB；MP4 最大 16 MB、30 秒。更换媒体前请先撤下动态。 |
| provider ID 统计 | 后端和控制台 0.1.1 已准备：按型号、系统/应用版本、UTC 日期汇总，支持 1/7/30 天及平台筛选。此范围尚未通过新安装或新发布的 iOS 客户端验证，Android 端尚未移植。 |
| 服务商定向 push、私有动态、通过 API 自动选服 | 尚未准备好供普遍使用。Xray JSON 内部的负载均衡是独立功能。 |
| 网络地图 | 封闭测试。Remnawave 需手动以只读权限接入，自建面板需要独立连接器。使用聚合数据，而非 `geoip.dat` / `geosite.dat`；不展示原始 IP 或精确位置。仅提供 provider ID 不会启用此功能。 |

动态是公开内容，不能包含秘密链接、客户资料或仅供付费订阅者查看的内容。

现有按 `profile-title` 和 `support-url` 分组的统计属于 DoDo 所有者。名称或联系方式相同，不会赋予服务商访问这些记录的权限。provider-ID 报告需要已验证的准确主机和服务批准。

计数代表发送过报告的安装，不代表在线人数或付费订阅者。一个安装可以包含多个服务商。控制台仅接收汇总，不包含密钥、URL、HWID 或单设备清单。最多每 24 小时尝试发送两次，间隔至少 12 小时。

控制台或 API 故障不应影响配置导入和 VPN 流量。写入超时后，请先刷新页面确认结果，再决定是否重试。抢先体验不意味着已完成真实服务商试点，也不意味着容量无限。

另见：[集成](integration.zh-CN.md) · [诊断与统计](diagnostics.zh-CN.md) · [兼容性](compatibility.zh-CN.md)

来源：[服务商注册](https://dodoproxy.app/docs/#provider-registration) · [自托管控制台](https://dodoproxy.app/docs/#provider-console) · [服务商功能](https://dodoproxy.app/providers/)
