# gRPC-Web

- gRPC [架构概述](../../intro/arch_overview/grpc.md#arch-overview-grpc)
- [v1 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/grpc_web_filter#config-http-filters-grpc-web-v1)
- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-field-config-filter-network-http-connection-manager-v2-httpfilter-name)

这个过滤器通过以下步骤可以将 gRPC-Web 客户端桥接到兼容的 gRPC 服务
<https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md> 。