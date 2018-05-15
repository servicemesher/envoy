我设置了健康检查，但是当有节点失败时，Envoy 又路由到那些节点，这是怎么回事？
================================================================================================

This feature is known as the load balancer [panic threshold](../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-panic-threshold). It is used to prevent cascading failure when upstream hosts start failing health checks in large numbers.