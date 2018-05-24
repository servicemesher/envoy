# TCP 代理

Envoy 本质上是一个 L3/L4 服务器，实现 L3/L4 的代理功能是一件轻而易举的事情。TCP 代理过滤器可以在下游的客户端与上游的集群间实现最基本的 1:1 网络连接代理功能。

这可以用来做一个安全通道的替代品，或者与其他的过滤器聚合如 [MongoDB 过滤器](mongo.md#arch-overview-mongo)或[速率限制过滤器](../../configuration/network_filters/rate_limit_filter.md#config-network-filters-rate-limit)。

TCP 代理过滤器会被上游集群全局资源管理器中使用的[连接限制](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers#config-cluster-manager-cluster-circuit-breakers-max-connections)约束。

TCP 代理过滤器需要和上游集群的资源管理器协商是否能在不逾越集群最大连接数的前提下，建立一个新连接，如果答案为否则 TCP 代理将不可以建立新连接。

TCP 代理过滤器[参考配置](../../configuration/network_filters/tcp_proxy_filter.md#config-network-filters-tcp-proxy)。