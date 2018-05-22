# 统计

Envoy 的主要目标之一是使网络可以理解。Envoy 根据不同的配置收集了大量的统计数据。一般来说，统计数据分为以下三类：


- **Downstream**：下游统计涉及传入的连接/请求。 它们由侦听器、HTTP 连接管理器、TCP 代理过滤器等生成。
- **Upstream**：上游统计涉及传出连接/请求。它们由连接池、路由器过滤器、TCP 代理过滤器等生成
- **Server**：服务器统计信息描述了 Envoy 服务器实例的工作方式。服务器正常运行时间或分配内存量等统计信息在此处分类。


单个代理场景通常涉及下游和上游统计信息。 这两种类型可用于获取该特定网络跃点的详细图。来自整个网格的统计数据给出了每一跳和整体网络健康状况的非常详细的图。 发布的统计数据在操作指南中详细记录。

在 v1 API中，Envoy 仅支持 statsd 作为统计输出格式。支持 TCP 和 UDP statsd。 截至 v2 API，Envoy 有能力支持自定义、可插拔接收器。Envoy 中包含了[一些标准接收器实现](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-statssink)。 有些接收器也支持使用标签/维度发布统计信息。

在 Envoy 和整个文档中，统计信息由规范的字符串表示来标识。 这些字符串的动态部分被剥离成为标签。 用户可以通过[Tag Specifier配置](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-tagspecifier)配置此行为。

特使发出三种类型的值作为统计数据：


-  **Counter（计数器）**：无符号整数只会增加而不会减少。 例如，总请求。  
-  **Gauge（量表）**：无符号整数，既有增加也有减少。 例如，当前有效的请求。  
-  **Histogram（直方图）**：作为值流的一部分的无符号整数，然后由收集器进行汇总以最终生成汇总百分点值。 例如，上游请求时间。

在内部，计数器和计量器被分批并定期冲洗以提高性能。直方图会在收到时写入。 注意：以前称为定时器的内容已成为直方图，因为这两种表示法之间的唯一区别就是单位。

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v1_overview#config-overview-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#envoy-api-field-config-bootstrap-v2-bootstrap-stats-sinks)

