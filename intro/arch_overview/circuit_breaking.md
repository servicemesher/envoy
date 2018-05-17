# 断路

断路（circuit breaking）是分布式系统的关键组成部分。尽快失败和向下游施加反向压力大体上会得到更好的效果。Envoy 网格的主要优点之一就是 Envoy 在网络级别强制实现断路限制，而不必独立配置和编写每个应用程序。Envoy 支持各种类型的完全分布式（非协调的）断路：

- **集群最大连接数**：Envoy 将为上游集群中的所有主机建立的最大连接数。实际上，这仅适用于 HTTP/1.1集群，因为 HTTP/2 使用到每个主机的单个连接。
- **集群最大挂起请求数**：在等待就绪连接池连接时将排队的最大请求数。实际上，这仅适用于 HTTP/1.1 集群，因为 HTTP/2 连接池不会排队请求。HTTP/2 请求会立即复用。如果该断路器溢出，则集群的[upstream_rq_pending_overflowcounter](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats) 计数器将增加。
- **集群最大请求数**：在任何给定时间内，集群中所有主机可以处理的最大请求数。实际上，这适用于仅 HTTP/2 集群，因为 HTTP/1.1 集群由最大连接断路器控制。如果该断路器溢出，集群的 [upstream_rq_pending_overflow](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats) 计数器将递增。
- **集群最大活动重试次数**：在任何给定时间内，集群中所有主机可以执行的最大重试次数。一般而言，我们建议积极进行断路重试，以便做零星故障重试而整体重试数量又不会爆炸式增加和导致大规模级联故障。如果该断路器溢出，集群的 [upstream_rq_retry_overflow](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats) 计数器将递增。

断路限制可以根据每个上游集群和优先级进行[配置](../../configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers)和跟踪。这使得分布式系统的不同组件可以独立调整并具有不同的限制。

请注意，在 HTTP 请求的情况下，断路会导致路由器设置 [x-envoy-overloaded](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded) 头。