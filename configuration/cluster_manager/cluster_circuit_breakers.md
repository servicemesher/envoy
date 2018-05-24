# 断路

- 断路[架构概览](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/circuit_breaking.html)
- [v1 API 文档](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers)
- [v2 API 文档](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster/circuit_breaker.proto)

## 运行时

所有的断路设置都是运行时可配置的，是基于集群名称的优先级定义。 他们遵循以下命名方案 `circuit_breakers.<cluster_name>.<priority>.<setting>`。 `cluster_name` 是每个群集配置中的名称字, 设置在envoy的 [配置文件](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster.html#config-cluster-manager-cluster-name)中。 可用的运行时设置将覆盖envoy配置文件中的配置。