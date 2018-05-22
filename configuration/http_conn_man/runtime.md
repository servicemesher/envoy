# 运行时

HTTP 连接管理支持以下运行时设定:

- http_connection_manager.represent_ipv4_remote_address_as_ipv4_mapped_ipv6

  将其 IPv4 地址映射到 IPv6 的远程地址的请求的百分比。默认为0。
  需要将 [use_remote_address](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-use-remote-address) 开启。
  详情可参看 [represent_ipv4_remote_address_as_ipv4_mapped_ipv6](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.html#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-represent-ipv4-remote-address-as-ipv4-mapped-ipv6)。

- tracing.client_enabled

  如设置 [x-client-trace-id](headers.md#config-http-conn-man-headers-x-client-trace-id) 头部，将被强行追踪的请求的百分比。默认为100。

- tracing.global_enabled

  在应用所有其他检查后追踪的请求的百分比（强制追踪，采样等）。默认为100。

- tracing.random_sampling

  将被随机跟踪的请求的百分比。 请查阅[此处](../../intro/arch_overview/tracing.md#arch-overview-tracing)以获得更多信息。该运行时间控制在0-10000范围内指定，默认值为10000。 因此，可以按0.01％的增量指定跟踪采样。