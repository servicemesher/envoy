# HTTP 连接管理

HTTP 是现代面向服务价格如此关键的一个元素，以至于 Envoy 实现了大量特定于 HTTP 的功能。Envoy 有一个内建的网络层过滤器称为 [HTTP 连接管理器](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)。该管理器将原始字节翻译为 HTTP 层消息和事件。(即，收到的头，收到的体数据，收到的尾，等等)。它还处理对所有 HTTP 连接共同的功能和请求，如[访问日志](access_logging.md#arch-overview-access-logs)、[请求 ID 生成和追踪](tracing.md#arch-overview-tracing)、[请求/相应头操控](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers)、[路由表](http_routing.md#arch-overview-http-routing)管理以及[统计](../../configuration/http_conn_man/stats.md#config-http-conn-man-stats)。

HTTP 连接管理器[配置](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)。

## HTTP 协议

Envoy 的 HTTP 连接管理器内建支持 HTTP/1.1、WebSockets 和 HTTP/2。它不支持 SPDY。Envoy 的 HTTP 支持被设计为首先是一个 HTTP/2 多路复用代理。在内部，HTTP/2 这一名词用于描述系统元素。例如，一个 HTTP 请求和响应发生在一个*流*上。一个编解码器 API 被用于为流、请求、响应等等将不同的连线协议翻译为一个与协议不可知的形式。对于 HTTP/1.1，编解码器将协议的串行/流水线能力转换为对更高层看起来像 HTTP/2 的某种东西。这意味着大部分代码不需要理解一个流是否来源于一个 HTTP/1.1 或 HTTP/2 连接。

## HTTP 头净化

出于安全原因，HTTP 连接管理器执行不同的[头净化](../../configuration/http_conn_man/header_sanitizing.md#config-http-conn-man-header-sanitizing)操作。

## 路由表配置

每个 [HTTP 连接管理器过滤器](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)有一个相关的[路由表](http_routing.md#arch-overview-http-routing)。路由表可以两种方式中的一种指定：

- 静态地。
- 通过 [RDS API](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds) 动态地。
