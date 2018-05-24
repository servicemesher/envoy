# gRPC HTTP/1.1 桥接

- gRPC [架构简述](../../intro/arch_overview/grpc.md#arch-overview-grpc)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/grpc_http1_bridge_filter#config-http-filters-grpc-bridge-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-field-config-filter-network-http-connection-manager-v2-httpfilter-name)


这是一个简单的过滤器，对于不支持复杂 gRPC 服务响应尾部的 HTTP/1.1 客户端，该过滤器可以实现桥接功能。它通过以下的步骤工作:

- 当发送一个请求时，过滤器会检查该链接是否是 HTTP/1.1 以及该请求的内容类型是否是 *application/grpc* 。

- 如果符合上面的条件，当收到响应时，过滤器会缓存它并等待响应的尾部，然后检查 *grpc-status* 状态码。如果状态码不为零，过滤器把 HTTP 状态码转换为503。 同时复制 *grpc-status* 和 *grpc-message* 的尾部到响应的头部，以便客户端在需要的时候可以查看他们。

- 客户端应该发送可以转换为以下伪头部的 HTTP/1.1 请求:
  - *:method：* POST
  - *:path*: <gRPC-METHOD-NAME>
  - *content-type*: application/grpc
- 请求的主体部分应该是以下格式的序列化的 grpc 主体：
  - 一个字节的零字符(没有压缩)
  - 网络顺序的4字节的原型消息长度
  - 序列化后的原型消息

- 因为这个模式必须缓存响应以查找 *grpc-status* 尾部，因此它只对一元模式的 gRPC 接口有效。


这个过滤器同时收集所有传输的 gRPC 请求，即使这些请求是正常通过 HTTP/2 传输的 gRPC 请求。 

更多的信息：有线格式地址 [gRPC 通过 HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)。


## 统计

过滤器在 *cluster.<route target cluster>.grpc.* 命名空间中输出统计信息。


| 名称                                  | 类型    | 描述                                  |
| ------------------------------------ | ------- | ------------------------------------- |
| <grpc service>.<grpc method>.success | Counter | 完全成功的服务或方法调用                 |
| <grpc service>.<grpc method>.failure | Counter | 完全失败的服务或方法调用                 |
| <grpc service>.<grpc method>.total   | Counter | 完全服务或方法调用                      |