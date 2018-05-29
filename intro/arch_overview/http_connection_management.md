# HTTP 连接管理

HTTP 是现代面向服务架构中非常关键的组件，因此 Envoy 实现了大量 HTTP 特有的功能点。Envoy 有一个内置的网络过滤器名为 [HTTP 连接管理器](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)。该管理器将原始字节翻译为 HTTP 消息和事件。(即，收到的标头、收到的正文数据、收到的标尾等等)。它还处理对所有 HTTP 连接以及请求的共性功能，如[访问日志](access_logging.md#arch-overview-access-logs)、[请求 ID 生成和追踪](tracing.md#arch-overview-tracing)、[请求/响应头操控](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers)、[路由表](http_routing.md#arch-overview-http-routing)管理以及[统计](../../configuration/http_conn_man/stats.md#config-http-conn-man-stats)。

HTTP 连接管理器[配置](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)。

## HTTP 协议

Envoy 的 HTTP 连接管理器内置支持 HTTP/1.1、WebSockets 和 HTTP/2，不支持 SPDY。Envoy 的 HTTP 支持首先被设计为一个 HTTP/2 多路复用代理。在内部，HTTP/2 这一名词用于描述系统元素。例如，HTTP 请求和响应发生在*流*上。编解码器 API 用于将不同的连线协议转换为针对流、请求、响应等不可知的形式。对于 HTTP/1.1，编解码器将协议的串行/流水线能力转换为更高层的 HTTP/2。这意味着大部分代码不需要了解流是源于 HTTP/1.1 还是 HTTP/2 连接。

## HTTP 头净化

出于安全原因，HTTP 连接管理器执行不同的[头净化](../../configuration/http_conn_man/header_sanitizing.md#config-http-conn-man-header-sanitizing)操作。

## 路由表配置

每个 [HTTP 连接管理器过滤器](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man)都有一个相关的[路由表](http_routing.md#arch-overview-http-routing)。路由表可以用以下任意一种方式指定：

- 静态设置。
- 通过 [RDS API](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds) 动态地设置。
