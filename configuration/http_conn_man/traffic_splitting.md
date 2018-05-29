# 流量转移拆分

>  **注意**
>
>  本节为 v1 API编写，但概念也适用于 v2 API。 未来的版本中将以 v2 API 为目标重写。

- [在两个上游之间转移流量](#traffic-shifting-between-two-upstreams)
- [跨多个上游拆分流量](#traffic-splitting-across-multiple-upstreams)

Envoy 的路由器可以跨两个或更多上游集群将流量拆分到虚拟主机中的路由。有两个常见的用例。

1. 版本升级：路由的流量逐渐从一个集群优雅地转移到另一个集群。 [流量转移](#config-http-conn-man-route-table-traffic-splitting-shift)部分更详细地描述了这个场景。
2. A/B 测试或多变量测试：同时测试`两个或更多版本`的相同服务。流向路由的流量必须在运行同一服务的不同版本的集群之间进行拆分。 [流量拆分](#config-http-conn-man-route-table-traffic-splitting-split)部分更详细地描述了这种情况。

## 在两个上游之间转移流量

路由配置中的[运行时](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route/http-conn-man-route-table-route-runtime)对象判断选择特定路由（以及它的集群）的可能性（译者注：可以理解为百分比）。通过使用运行时配置，虚拟主机中到特定路由的流量可逐渐从一个集群转移到另一个集群。考虑以下示例配置，其中在 envoy 配置文件中声明了名为 `helloworld` 的服务的两个版本 `helloworld_v1` 和 `helloworld_v2`。

```json
{
  "route_config": {
    "virtual_hosts": [
      {
        "name": "helloworld",
        "domains": ["*"],
        "routes": [
          {
            "prefix": "/",
            "cluster": "helloworld_v1",
            "runtime": {
              "key": "routing.traffic_shift.helloworld",
              "default": 50
            }
          },
          {
            "prefix": "/",
            "cluster": "helloworld_v2",
          }
        ]
      }
    ]
  }
}
```

Envoy 使用 [first match](route_matching.html#config-http-conn-man-route-table-route-matching)  策略来匹配路由。如果路由具有运行时对象，则会根据运行时[值](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route.html#config-http-conn-man-route-table-route-runtime-default)另外匹配请求（如果未指定值，则为默认值）。因此，通过在上述示例中背靠背地放置路由并在第一个路由中指定运行时对象，可以通过更改运行时值来完成流量转移。以下是完成任务所需的大致操作顺序。

1. 在开始时，将 `routing.traffic_shift.helloworld` 设置为 `100`, 因此所有到 `helloworld`  虚拟主机的请求都将匹配 v1 路由并由 `helloworld_v1` 集群提供服务。
2. 为了开始将流量转移到 `helloworld_v2` 集群, 设置 `routing.traffic_shift.helloworld` 为 `0 < x < 100`. 例如设置为 `90` 时，有1个不会与 v1 路由匹配，然后会落入 v2 路由。
3. 逐渐减少 `routing.traffic_shift.helloworld` 中设置的值，以便大部分请求与 `v2` 路由匹配。
4. 当 `routing.traffic_shift.helloworld`  设置为 `0` 时, 到 `helloworld`  虚拟主机的请求都不会匹配 v1 路由。现在，所有流量都会流向 v2 路由，并由 helloworld_v2 集群提供服务。

## 跨多个上游拆分流量

再次考虑 `helloworld` 示例，现在有三个版本（v1、v2和v3）而不是两个。要在三个版本间平均分配流量（即`33％`、`33％`、`34％`），可以使用 `weighted_clusters`选项指定每个上游集群的权重。

与前面的例子不同，单个[路由](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route.html#config-http-conn-man-route-table-route)条目就足够了。路由中的 [weighted_clusters](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-weighted-clusters) 配置块可用于指定多个上游集群以及权重，权证则表示要发送到每个上游集群的流量百分比。

```json
{
  "route_config": {
    "virtual_hosts": [
      {
        "name": "helloworld",
        "domains": ["*"],
        "routes": [
          {
            "prefix": "/",
            "weighted_clusters": {
              "runtime_key_prefix" : "routing.traffic_split.helloworld",
              "clusters" : [
                { "name" : "helloworld_v1", "weight" : 33 },
                { "name" : "helloworld_v2", "weight" : 33 },
                { "name" : "helloworld_v3", "weight" : 34 }
              ]
            }
          }
        ]
      }
    ]
  }
}
```

默认情况下，权重的总和必须精确地为100。在 V2 API 中，总权重默认为100，但可以修改以允许更精细的粒度。

可以使用以下运行时变量动态调整分配给每个集群的权重：`routing.traffic_split.helloworld.helloworld_v1`、`routing.traffic_split.helloworld.helloworld_v2` 和 `routing.traffic_split.helloworld.helloworld_v3`。