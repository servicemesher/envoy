# 术语

在深入主架构文档之前的一些定义。部分定义在行业中略有争议，下面将展开在Envoy文档和代码库中如何使用它们。

**Host/主机**: 能够进行网络通信的实体（如手机，服务器等上的应用程序）。在此文档中，主机是逻辑网络应用程序。一块物理硬件上可能运行有多个主机，只要他们可以独立寻址。

**Downstream/下游**: 下游主机连接到Envoy, 发送请求并接收响应。

**Upstream/上游**: 上游主机接收来自Envoy的连接和请求，并返回响应。

**Listener/监听器**: 监听器是命名网地址(例如, 端口, unix domain socket, 等)，可以被下游客户端连接。Envoy暴露一个或者多个监听器给下游主机连接。

**Cluster/集群**: 群集是指Envoy连接到的逻辑上相同的一组上游主机。Envoy通过[服务发现来](service_discovery.md#arch-overview-service-discovery)发现集群的成员。可以可选择的通过[主动健康检查](health_checking.md#arch-overview-health-checking)来确定集群成员的健康状态。Envoy通过[负载均衡策略](load_balancing.md#arch-overview-load-balancing)决定将请求路由到哪个集群成员。

**Mesh/网格**: 一组主机，协调好以提供一致的网络拓扑。在本文档中，“Envoy mesh”是一组Envoy代理，它们构成了分布式系统的消息传递基础，这个分布式系统由很多不同服务和应用程序平台组成。

**Runtime configuration/运行时配置**: 外置实时配置系统，和Envoy一起部署。可以更改配置设置，影响操作，而无需重启Envoy或更改主要配置。