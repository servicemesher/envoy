# 我设置了健康检查，但是当有节点失败时，Envoy 又路由到那些节点，这是怎么回事？


这个功能因为负载均衡中[恐慌阈值](../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-panic-threshold)为人所知。在上游站点发生大量的健康检查失败的时候，它被用来防止级联故障。