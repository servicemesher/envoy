# 访问记录

[HTTP 连接管理](http_connection_management.md#arch-overview-http-conn-man)与 [tcp 代理](tcp_proxy.md#arch-overview-tcp-proxy)支持可扩展的访问记录，并拥有以下特性:

- 可按每个 HTTP 连接管理或 tcp 代理记录任何数量的访问记录。
- 异步 IO 架构。访问记录将永远不会阻塞主要的网络处理线程。
- 可定制化的访问记录格式,可使用预制定的字段,更可使用任意的 HTTP request 以及 response。
- 可定制化的访问记录过滤器,可允许不同类型的请求以及回复写入至不同的访问记录。

访问记录 [配置](../../configuration/access_log.md#config-access-log)。