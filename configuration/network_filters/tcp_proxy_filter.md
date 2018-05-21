# TCP 代理

- TCP 代理 [架构概述](../../intro/arch_overview/tcp_proxy.md#arch-overview-tcp-proxy)
- [v1 接口文档](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/tcp_proxy_filter#config-network-filters-tcp-proxy-v1)
- [v2 接口文档](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto#envoy-api-msg-config-filter-network-tcp-proxy-v2-tcpproxy)

## 统计

在适当的情况下，TCP 代理会发出自己下游以及[上游集群的统计信息](../cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats)。下游的统计信息都有以 *tcp.<stat_prefix>* 开头的统计，并提供如下的统计报告：

| 名称                                          | 类型    | 描述                                                  |
| --------------------------------------------- | ------- | ------------------------------------------------------------ |
| downstream_cx_total                           | Counter | 过滤器处理的连接总数            |
| downstream_cx_no_route                        | Counter | 没有找到匹配路由或没有找到路由集群的连接总数 |
| downstream_cx_tx_bytes_total                  | Counter | 写入下游连接的总字节数             |
| downstream_cx_tx_bytes_buffered               | Gauge   | 当前缓存到下游连接的总字节数  |
| downstream_cx_rx_bytes_total                  | Counter | 从下游连接读取的总字节数              |
| downstream_cx_rx_bytes_buffered               | Gauge   | 当前从下游连接缓存的总字节数 |
| downstream_flow_control_paused_reading_total  | Counter | 流量控制从下游暂停读取的总次数 |
| downstream_flow_control_resumed_reading_total | Counter | 流量控制从下游恢复读取的总次数 |
| idle_timeout                                  | Counter | 由于空闲连接超时而关闭的连接总数       |
| upstream_flush_total                          | Counter | 在下游连接关闭后继续刷新上游数据的连接总数 |
| upstream_flush_active                         | Gauge   | 在下游连接关闭后，当前继续刷新上游数据的连接总数 |