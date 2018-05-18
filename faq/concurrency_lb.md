# 为什么 Round Robin 负载均衡看起来不起作用？

Envoy 使用隔离方式的[线程模型](../intro/arch_overview/threading_model.mdrch-overview-threading)。这意味着工作线程以及相应的负载均衡器彼此不能互相协助。当使用例如 [round robin](../intro/arch_overview/load_balancing.html#arch-overview-load-balancing-types-round-robin) 的负载均衡策略时，该策略也许不能非常好地操作多个工作线程。此时我们可以使用 [`--concurrency`](../operations/cli.md#cmdoption-concurrency)选项来调整需要运行的工作线程数。

隔离方式的运行模型也引致了如下情形，我们时常需要为每一个上游建立多个 HTTP/2 连接；且工作线程间无法共享[连接池](../intro/arch_overview/connection_pooling.md#arch-overview-conn-pool)。