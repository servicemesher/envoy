# 如何设置 zone 可感知路由？

There are several steps required for enabling [zone aware routing](../intro/arch_overview/load_balancing.mdrch-overview-load-balancing-zone-aware-routing) between source service (“cluster_a”) and destination service (“cluster_b”).

## Envoy configuration on the source service

This section describes the specific configuration for the Envoy running side by side with the source service. These are the requirements:

- Envoy must be launched with [`--service-zone`](../operations/cli.html#cmdoption-service-zone) option which defines the zone for the current host.

- Both definitions of the source and the destination clusters must have [sds](../api-v1/cluster_manager/cluster.md#config-cluster-manager-type) type.

- [local_cluster_name](../api-v1/cluster_manager/cluster_manager.md#config-cluster-manager-local-cluster-name) must be set to the source cluster.

  Only essential parts are listed in the configuration below for the cluster manager.

```json
{
  "sds": "{...}",
  "local_cluster_name": "cluster_a",
  "clusters": [
    {
      "name": "cluster_a",
      "type": "sds",
    },
    {
      "name": "cluster_b",
      "type": "sds"
    }
  ]
}
```

## Envoy configuration on the destination service

It’s not necessary to run Envoy side by side with the destination service, but it’s important that each host in the destination cluster registers with the discovery service [queried by the source service Envoy](../api-v1/cluster_manager/sds.md#config-cluster-manager-sds-api). [Zone](../api-v1/cluster_manager/sds.md#config-cluster-manager-sds-api-host) information must be available as part of that response.

Only zone related data is listed in the response below.

```json
{
   "tags": {
       "az": "us-east-1d"
   }
}
```

## Infrastructure setup

The above configuration is necessary for zone aware routing, but there are certain conditions when zone aware routing is [not performed](../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-zone-aware-routing-preconditions).

## Verification steps

- Use [per zone](../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-per-az-stats) Envoy stats to monitor cross zone traffic.