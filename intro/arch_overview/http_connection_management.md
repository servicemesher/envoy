# HTTP 连接管理

HTTP is such a critical component of modern service oriented architectures that Envoy implements a large amount of HTTP specific functionality. Envoy has a built in network level filter called the[HTTP connection manager](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man). This filter translates raw bytes into HTTP level messages and events (e.g., headers received, body data received, trailers received, etc.). It also handles functionality common to all HTTP connections and requests such as [access logging](access_logging.md#arch-overview-access-logs), [request ID generation and tracing](tracing.md#arch-overview-tracing), [request/response header manipulation](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers), [route table](http_routing.md#arch-overview-http-routing) management, and [statistics](../../configuration/http_conn_man/stats.md#config-http-conn-man-stats).

HTTP connection manager [configuration](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man).

## HTTP 协议

Envoy’s HTTP connection manager has native support for HTTP/1.1, WebSockets, and HTTP/2. It does not support SPDY. Envoy’s HTTP support was designed to first and foremost be an HTTP/2 multiplexing proxy. Internally, HTTP/2 terminology is used to describe system components. For example, an HTTP request and response take place on a *stream*. A codec API is used to translate from different wire protocols into a protocol agnostic form for streams, requests, responses, etc. In the case of HTTP/1.1, the codec translates the serial/pipelining capabilities of the protocol into something that looks like HTTP/2 to higher layers. This means that the majority of the code does not need to understand whether a stream originated on an HTTP/1.1 or HTTP/2 connection.

## HTTP header sanitizing

The HTTP connection manager performs various [header sanitizing](../../configuration/http_conn_man/header_sanitizing.md#config-http-conn-man-header-sanitizing) actions for security reasons.路由表

## 路由表配置

Each [HTTP connection manager filter](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man) has an associated [route table](http_routing.md#arch-overview-http-routing). The route table can be specified in one of two ways:

- Statically.
- Dynamically via the [RDS API](../../configuration/http_conn_man/rds.md#config-http-conn-man-rds).