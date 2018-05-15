# 访问记录

The [HTTP connection manager](http_connection_management.md#arch-overview-http-conn-man) and [tcp proxy](tcp_proxy.md#arch-overview-tcp-proxy) supports extensible access logging with the following features:

- Any number of access logs per connection manager or tcp proxy.
- Asynchronous IO flushing architecture. Access logging will never block the main network processing threads.
- Customizable access log formats using predefined fields as well as arbitrary HTTP request and response headers.
- Customizable access log filters that allow different types of requests and responses to be written to different access logs.

Access log [configuration](../../configuration/access_log.md#config-access-log).