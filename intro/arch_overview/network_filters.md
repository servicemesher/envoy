# 网络级 (L3/L4) 过滤器

如[监听器](listeners.md#arch-overview-listeners)一节所述，网络级（L3/L4）过滤器构成Envoy连接处理的核心。过滤器API允许混合不同的过滤器组合，并匹配和附加到给定的监听器。有三种不同类型的网络级过滤器：

- **读**: 当Envoy从下游连接接收数据时，调用读过滤器。
- **写**: 当Envoy要发送数据到下游连接时，调用写过滤器。
- **读/写**: 当Envoy从下游连接接收数据和要发送数据到下游连接时，调用读/写过滤器。

网络级过滤器的API相对简单，因为最终过滤器只操作原始字节和少量连接事件（例如，TLS握手完成，连接在本地或远程断开等）。可停止链中的过滤器并继续执行后续的过滤器。这允许去运作更复杂的业务场景，例如调用[限速服务](global_rate_limiting.md#arch-overview-rate-limit)等。Envoy包含多个网络级的、过滤器，这些过滤器在此架构概述和[配置参考](../../configuration/network_filters/network_filters.md#config-network-filters)中都有说明。