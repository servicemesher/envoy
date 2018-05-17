# 统计概览

Envoy 基于服务器的配置输出数字统计信息。统计信息在本地可以通过 [`GET /stats`](admin.md#get--stats) 命令查看，通常情况下，统计信息会发送到  [statsd cluster](../intro/arch_overview/statistics.md#arch-overview-statistics)。输出的统计信息记录在[配置指南](../configuration/configuration.md#config)的相关部分中。一些更重要的统计信息总是会被使用到，这些信息可以查阅以下章节：

- [HTTP 链接管理器](../configuration/http_conn_man/stats.md#config-http-conn-man-stats)
- [Upstream 集群](../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats)

