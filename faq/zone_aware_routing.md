# 如何设置 zone 可感知路由？

在源服务（“cluster_a”）和目标服务（“cluster_b”）之间启用[区域感知路由](../intro/arch_overview/load_balancing.mdrch-overview-load-balancing-zone-aware-routing)需要执行几个步骤。

## 源服务上的 Envoy 配置

本节介绍与源服务一起运行的 Envoy 的具体配置。要求如下：

- Envoy 必须使用 [`--service-zone`](../operations/cli.html#cmdoption-service-zone) 选项启动，该选项为当前主机定义区域。

- 源和目的地集群的定义都必须具有 [sds](../https://www.envoyproxy.io/docs/envoy/latest/aip-v1/cluster_manager/cluster#config-cluster-manager-type) 类型。

- 必须将 [local_cluster_name](../https://www.envoyproxy.io/docs/envoy/latest/aip-v1/cluster_manager/cluster_manager#config-cluster-manager-local-cluster-name) 设置为源集群。

  以下配置中仅列出了集群管理器的主要部分。

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

## 目的地服务上的 Envoy 配置

没有必要与目的地服务并排运行 Envoy，但重要的是目的地集群中的每台主机都注册[源服务 Envoy 查询](../https://www.envoyproxy.io/docs/envoy/latest/aip-v1/cluster_manager/sds#config-cluster-manager-sds-api)的发现服务。[区域](https://www.envoyproxy.io/docs/envoy/latest/aip-v1/cluster_manager/sds#config-cluster-manager-sds-api-host)信息必须作为该响应的一部分提供。

下面的应答中只列出了与区域相关的数据。

```json
{
   "tags": {
       "az": "us-east-1d"
   }
}
```

## 基础设施搭建

上述配置对于区域感知路由是必需的，但是在[不执行](../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-zone-aware-routing-preconditions)区域感知路由时存在某些情况。

## 验证步骤

- 使用[每区域](../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-per-az-stats) Envoy 统计信息来监控跨区域流量。