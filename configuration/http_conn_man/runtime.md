# 运行时

The HTTP connection manager supports the following runtime settings:

- http_connection_manager.represent_ipv4_remote_address_as_ipv4_mapped_ipv6

  % of requests with a remote address that will have their IPv4 address mapped to IPv6. Defaults to 0. [use_remote_address](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.md#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-use-remote-address) must also be enabled. See[represent_ipv4_remote_address_as_ipv4_mapped_ipv6](../../api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto.md#envoy-api-field-config-filter-network-http-connection-manager-v2-httpconnectionmanager-represent-ipv4-remote-address-as-ipv4-mapped-ipv6) for more details.

- tracing.client_enabled

  % of requests that will be force traced if the [x-client-trace-id](headers.md#config-http-conn-man-headers-x-client-trace-id) header is set. Defaults to 100.

- tracing.global_enabled

  % of requests that will be traced after all other checks have been applied (force tracing, sampling, etc.). Defaults to 100.

- tracing.random_sampling

  % of requests that will be randomly traced. See [here](../../intro/arch_overview/tracing.md#arch-overview-tracing) for more information. This runtime control is specified in the range 0-10000 and defaults to 10000. Thus, trace sampling can be specified in 0.01% increments.