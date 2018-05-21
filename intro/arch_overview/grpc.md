# gRPC

[gRpc](http://www.grpc.io) 是来自 Google 的 RPC 框架。它使用协议缓冲区作为底层 序列化 /IDL(接口描述语言的缩写) 格式。在传输层，它使用 HTTP/2 进行请求/响应复用。Envoy 在传输层和应用层都提供对 gRPC 的一流支持：

- gRPC 使用 HTTP/2 trailers 特性（可以在 HTTP 请求和响应报文后追加 HTTP Header)来传送请求状态。Envoy 是能够正确支持 HTTP/2 trailers 的少数几个 HTTP 代理之一，因此也是可以传输 gRPC 请求和响应的代理之一。

- 某些语言的 gRPC 运行时相对不成熟。Envoy 支持 gRPC [桥接过滤器](../../configuration/http_filters/grpc_http1_bridge_filter.md#config-http-filters-grpc-bridge)，允许 gRPC 请求通过 HTTP/1.1 发送给 Envoy。然后，Envoy 将请求转换为 HTTP/2 以传输到目标服务器。该响应被转换回 HTTP/1.1。

- 安装后，除了标准的全局 HTTP 统计数据之外，桥接过滤器还会根据每个 RPC 统计数据进行收集。

- gRPC-Web 由一个指定的[过滤器](../../configuration/http_filters/grpc_web_filter.md#config-http-filters-grpc-web)支持，该过滤器允许 gRPC-Web 客户端通过 HTTP/1.1 向 Envoy 发送请求并代理到 gRPC 服务器。目前相关团队正在积极开发中，预计它将成为 gRPC [桥接过滤器](../../configuration/http_filters/grpc_http1_bridge_filter.md#config-http-filters-grpc-bridge)的后续产品。

- gRPC-JSON 转码器由一个指定的[过滤器](../../configuration/http_filters/grpc_json_transcoder_filter.md#config-http-filters-grpc-json-transcoder)支持，该过滤器允许 RESTful JSON API 客户端通过 HTTP 向 Envoy 发送请求并获取代理到 gRPC 服务。

## gRPC 服务

除了在数据层面上代理 gRPC 外，Envoy 在控制层面也使用了 gRPC，它从中[获取管理服务器的配置](../../configuration/overview/v2_overview.md#config-overview-v2)以及过滤器中的配置，例如用于[速率限制](../../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit))或授权检查。我们称之为 *gRPC 服务*。

当指定 gRPC 服务时，必须指定使用 [Envoy gRPC](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/grpc_service.proto.html#envoy-api-field-core-grpcservice-envoy-grpc) 客户端或 [Google C ++ gRPC 客户端](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/grpc_service.proto.html#envoy-api-field-core-grpcservice-google-grpc)。我们在下面的这个选择中讨论权衡。

Envoy gRPC 客户端是使用 Envoy 的 HTTP/2 上行连接管理的 gRPC 的最小自定义实现。服务被指定为常规 Envoy [集群](cluster_manager.md#arch-overview-cluster-manager)，定期处理[超时、重试](http_connection_management.md#arch-overview-http-conn-man)、[终端发现](dynamic_configuration.md#arch-overview-dynamic-config-sds)、[负载平衡、故障转移](load_balancing.md#arch-overview-load-balancing)、负载报告、[断路](circuit_breaking.md#arch-overview-circuit-break)、[健康检查](health_checking.md#arch-overview-health-checking)、[异常检测](connection_pooling.md#arch-overview-conn-pool)。它们与 Envoy 的数据层面共享相同的[连接池](connection_pooling.md#arch-overview-conn-pool)机制。同样，集群[统计信息](statistics.md#arch-overview-statistics)可用于 gRPC 服务。由于客户端是简化版的 gRPC 实现，因此不包括诸如 [OAuth2](https://oauth.net/2/) 或 [gRPC-LB](https://grpc.io/blog/loadbalancing) 之类的高级 gRPC 功能后备。

Google C++ gRPC 客户端基于 Google 在 <https://github.com/grpc/grpc> 上提供的 gRPC 参考实现。它提供了 Envoy gRPC 客户端中缺少的高级 gRPC 功能。Google C++ gRPC 客户端独立于 Envoy 的集群管理，执行自己的负载平衡、重试、超时、端点管理等。Google C++ gRPC 客户端还支持[自定义身份验证插件](https://grpc.io/docs/guides/auth.md#extending-grpc-to-support-other-authentication-mechanisms)。

在大多数情况下，当你不需要 Google C++ gRPC 客户端的高级功能时，建议使用 Envoy gRPC 客户端。这使得配置和监控更加简单。如果 Envoy gRPC 客户端中缺少你所需要的功能，则应该使用 Google C++ gRPC 客户端。