# TCP 代理

- TCP proxy [architecture overview](../../intro/arch_overview/tcp_proxy.md#arch-overview-tcp-proxy)
- [v1 API reference](../../api-v1/network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy-v1)
- [v2 API reference](../../api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto.md#envoy-api-msg-config-filter-network-tcp-proxy-v2-tcpproxy)

## Statistics

The TCP proxy filter emits both its own downstream statistics as well as many of the [cluster upstream statistics](../cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats) where applicable. The downstream statistics are rooted at *tcp.<stat_prefix>.*with the following statistics:

| Name                                          | Type    | Description                                                  |
| --------------------------------------------- | ------- | ------------------------------------------------------------ |
| downstream_cx_total                           | Counter | Total number of connections handled by the filter            |
| downstream_cx_no_route                        | Counter | Number of connections for which no matching route was found or the cluster for the route was not found |
| downstream_cx_tx_bytes_total                  | Counter | Total bytes written to the downstream connection             |
| downstream_cx_tx_bytes_buffered               | Gauge   | Total bytes currently buffered to the downstream connection  |
| downstream_cx_rx_bytes_total                  | Counter | Total bytes read from the downstream connection              |
| downstream_cx_rx_bytes_buffered               | Gauge   | Total bytes currently buffered from the downstream connection |
| downstream_flow_control_paused_reading_total  | Counter | Total number of times flow control paused reading from downstream |
| downstream_flow_control_resumed_reading_total | Counter | Total number of times flow control resumed reading from downstream |
| idle_timeout                                  | Counter | Total number of connections closed due to idle timeout       |
| upstream_flush_total                          | Counter | Total number of connections that continued to flush upstream data after the downstream connection was closed |
| upstream_flush_active                         | Gauge   | Total connections currently continuing to flush upstream data after the downstream connection was closed |