# iOS 2.2.2：配置兼容性与服务器检查

[DoDo APP](../README.zh-CN.md) · [文档](README.zh-CN.md) · [版本历史](updates-feedback.zh-CN.md)

[English](ios-2.2.2.md) · [Русский](ios-2.2.2.ru.md) · [فارسی](ios-2.2.2.fa.md) · [简体中文](ios-2.2.2.zh-CN.md)

**截至 2026 年 11 月 12 日：iOS 2.2.2（113）**已上传至 App Store Connect 以供处理，尚未提交 App Review，也尚未宣布在 App Store 上线。构建处理、Apple 审核和正式发布是不同阶段。

以下改动仅适用于这一 iOS 构建。最低要求为 iOS 15，核心仍为 Xray 26.10.31；它们不代表 Android APK 2.2.0（2107）、macOS 或 Windows 已更新。

## 完整 JSON：`env` 与 `version`

根级 `env` 和 `version` 会随完整配置保留。仅包含 `outbounds` 及这些字段的文档不再被拆成独立链接；不包含完整配置要求的普通独立出口集合仍作为集合导入。

固定版本的核心在启动前检查 `version.min` / `version.max`，不会忽略不兼容要求。支持的 `env` 值仅应用于相应连接或辅助核心，并在确认清理完成后恢复，避免一个服务的设置影响下一会话。资源路径、应用设置、Go/TUN、代理及 TLS 信任参数受到保护。不支持的有效要求会被明确拒绝，错误信息不会包含 ENV 值。

## 经配置执行 HTTP GET / HEAD

连接与 HTTP 检查采用共同的配置准备流程，保留所选代理或负载均衡器、出口依赖、DNS、路由、传输参数及对应地理数据。检查在隔离的辅助 Xray 实例中执行，不会重新连接当前 VPN。HTTP 失败不会被直连成功或 TCP 端口开放替代。

修复了 Hysteria2 HTTP 检查的地理数据准备。由于 Hysteria2 使用 UDP，TCP 模式仍会自动对这些配置执行 HTTP 检查。普通 TCP 显示原始测量时间，不作除法；只有估算模式 DoDo Ping 将 HTTP GET 结果除以 3.3。这不是速度测试，也不是对活动 VPN 隧道的完整验证。

## DNS、分流 DoH 与有限 FakeDNS

改进了 Xray DNS 与系统隧道的协调、分流 DoH 及本地 DNS 别名。别名链有长度限制并检测循环，不会仅为启动成功而强制改写含义不明确的配置。

支持服务商明确配置、使用标准地址池的兼容 FakeDNS 配置；不表示任意地址池、混合 DNS 或 Lua 均受支持。FakeDNS 配置无法进行单独的 HTTP 检查，该检查显示 `N/A` 本身不代表 VPN 无法工作。

## FinalMask 与分片参数

兼容 URI 可携带 `fm`、`fragment` 以及 `fragmentPackets` / `fragmentLength` / `fragmentInterval`。实际成功的 HTTP 响应可提供：

```http
fragmentation-enable: true
fragmentation-type: xray
fragmentation-packets: tlshello
fragmentation-length: 50-100
fragmentation-delay: 10-20
fragmentation-maxsplit: 100-200
```

`fragmentation-interval` 是延迟参数的另一种 HTTP 响应头名称，不代表会重命名完整 JSON 内的字段。响应头方式要求 `fragmentation-enable: true` 且类型为 `xray`。

优先级为：**完整 JSON → 显式 URI 掩码 → 响应头**。不会自动叠加两个掩码，也不会静默修补服务商的完整 JSON；不支持的有效模式会被拒绝。自动 TCP 分片不会应用于 UDP-only / Hysteria 配置。界面没有用于编辑全部掩码参数的独立编辑器。

## 保持不变的部分

无需仅因客户端升级而更换正常密钥、调整配置顺序或修改服务器设置。TLS 证书验证不会降低，也未加入新 SDK。导入成功或某一 URL 检查成功不保证所有节点和网络均可用；服务商仍需验证自身 DNS、标签及传输参数。

[兼容性](compatibility.zh-CN.md) · [集成](integration.zh-CN.md) · [路由](routing.zh-CN.md) · [诊断](diagnostics.zh-CN.md)

公开说明：[iOS 2.2.2 准备情况](https://dodoproxy.app/docs/#ios-next-update)。App Store 上线时间将另行公布。
