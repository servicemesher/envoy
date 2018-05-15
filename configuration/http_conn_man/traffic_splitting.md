# 流量转换切分

>  **Attention**
>
> This section is written for the v1 API but the concepts also apply to the v2 API. It will be rewritten to target the v2 API in a future release.

- [Traffic shifting between two upstreams](#traffic-shifting-between-two-upstreams)
- [Traffic splitting across multiple upstreams](#traffic-splitting-across-multiple-upstreams)

Envoy’s router can split traffic to a route in a virtual host across two or more upstream clusters. There are two common use cases.

1. Version upgrades: traffic to a route is shifted gradually from one cluster to another. The [traffic shifting](#config-http-conn-man-route-table-traffic-splitting-shift) section describes this scenario in more detail.
2. A/B testing or multivariate testing: `two or more versions` of the same service are tested simultaneously. The traffic to the route has to be *split* between clusters running different versions of the same service. The [traffic splitting](#config-http-conn-man-route-table-traffic-splitting-split) section describes this scenario in more detail.

## Traffic shifting between two upstreams

The [runtime](../../api-v1/route_config/route.mdg-http-conn-man-route-table-route-runtime) object in the route configuration determines the probability of selecting a particular route (and hence its cluster). By using the runtime configuration, traffic to a particular route in a virtual host can be gradually shifted from one cluster to another. Consider the following example configuration, where two versions `helloworld_v1` and `helloworld_v2` of a service named `helloworld`are declared in the envoy configuration file.

```
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

Envoy matches routes with a [first match](route_matching.html#config-http-conn-man-route-table-route-matching) policy. If the route has a runtime object, the request will be additionally matched based on the runtime [value](../../api-v1/route_config/route.html#config-http-conn-man-route-table-route-runtime-default) (or the default, if no value is specified). Thus, by placing routes back-to-back in the above example and specifying a runtime object in the first route, traffic shifting can be accomplished by changing the runtime value. The following are the approximate sequence of actions required to accomplish the task.

1. In the beginning, set `routing.traffic_shift.helloworld` to `100`, so that all requests to the `helloworld` virtual host would match with the v1 route and be served by the `helloworld_v1`cluster.
2. To start shifting traffic to `helloworld_v2` cluster, set `routing.traffic_shift.helloworld` to values `0 < x < 100`. For instance at `90`, 1 out of every 10 requests to the `helloworld` virtual host will not match the v1 route and will fall through to the v2 route.
3. Gradually decrease the value set in `routing.traffic_shift.helloworld` so that a larger percentage of requests match the v2 route.
4. When `routing.traffic_shift.helloworld` is set to `0`, no requests to the `helloworld` virtual host will match to the v1 route. All traffic would now fall through to the v2 route and be served by the `helloworld_v2` cluster.

## Traffic splitting across multiple upstreams

Consider the `helloworld` example again, now with three versions (v1, v2 and v3) instead of two. To split traffic evenly across the three versions (i.e., `33%, 33%, 34%`), the `weighted_clusters` option can be used to specify the weight for each upstream cluster.

Unlike the previous example, a **single** [route](../../api-v1/route_config/route.html#config-http-conn-man-route-table-route) entry is sufficient. The [weighted_clusters](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-weighted-clusters) configuration block in a route can be used to specify multiple upstream clusters along with weights that indicate the **percentage** of traffic to be sent to each upstream cluster.

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

By default, the weights must sum to exactly 100. In the V2 API, the [total weight](../../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-weightedcluster-total-weight) defaults to 100, but can be modified to allow finer granularity.

The weights assigned to each cluster can be dynamically adjusted using the following runtime variables: `routing.traffic_split.helloworld.helloworld_v1`,`routing.traffic_split.helloworld.helloworld_v2` and `routing.traffic_split.helloworld.helloworld_v3`.