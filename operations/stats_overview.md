# 统计概览

Envoy outputs numerous statistics which depend on how the server is configured. They can be seen locally via the [`GET /stats`](admin.md#get--stats) command and are typically sent to a [statsd cluster](../intro/arch_overview/statistics.md#arch-overview-statistics). The statistics that are output are documented in the relevant sections of the [configuration guide](../configuration/configuration.md#config). Some of the more important statistics that will almost always be used can be found in the following sections:

- [HTTP connection manager](../configuration/http_conn_man/stats.md#config-http-conn-man-stats)
- [Upstream cluster](../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats)