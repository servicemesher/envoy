# 回写

回写是一个简单的网络过滤器，主要用于演示网络级别过滤器 API。 安装此过滤器后，它会将所有接收到的数据回写（写入）回连接的下游客户端。

- [v1 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/echo_filter#config-network-filters-echo-v1)
- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-field-listener-filter-name)