# 断路

- Circuit Breaking [architecture overview](../../intro/arch_overview/circuit_breaking.md#arch-overview-circuit-break).
- [v1 API documentation](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-v1).
- [v2 API documentation](../../api-v2/api/v2/cluster/circuit_breaker.proto.md#envoy-api-msg-cluster-circuitbreakers).

## Runtime

All circuit breaking settings are runtime configurable for all defined priorities based on cluster name. They follow the following naming scheme `circuit_breakers.<cluster_name>.<priority>.<setting>`. `cluster_name` is the name field in each cluster’s configuration, which is set in the envoy [config file](../../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-name). Available runtime settings will override settings set in the envoy config file.