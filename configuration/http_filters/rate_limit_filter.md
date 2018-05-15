#速率限制 

- Global rate limiting [architecture overview](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)
- [v1 API reference](../../api-v1/http_filters/rate_limit_filter.md#config-http-filters-rate-limit-v1)
- [v2 API reference](../../api-v2/config/filter/http/rate_limit/v2/rate_limit.proto.md#envoy-api-msg-config-filter-http-rate-limit-v2-ratelimit)

The HTTP rate limit filter will call the rate limit service when the request’s route or virtual host has one or more [rate limit configurations](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-rate-limits) that match the filter stage setting. The [route](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-include-vh) can optionally include the virtual host rate limit configurations. More than one configuration can apply to a request. Each configuration results in a descriptor being sent to the rate limit service.

If the rate limit service is called, and the response for any of the descriptors is over limit, a 429 response is returned.

## Composing Actions

Attention

This section is written for the v1 API but the concepts also apply to the v2 API. It will be rewritten to target the v2 API in a future release.

Each [rate limit action](../../api-v1/route_config/rate_limits.md#config-http-conn-man-route-table-rate-limit-config) on the route or virtual host populates a descriptor entry. A vector of descriptor entries compose a descriptor. To create more complex rate limit descriptors, actions can be composed in any order. The descriptor will be populated in the order the actions are specified in the configuration.

### Example 1

For example, to generate the following descriptor:

```
("generic_key", "some_value")
("source_cluster", "from_cluster")
```

The configuration would be:

```json
{
  "actions" : [
  {
    "type" : "generic_key",
    "descriptor_value" : "some_value"
  },
  {
    "type" : "source_cluster"
  }
  ]
}
```

### Example 2

If an action doesn’t append a descriptor entry, no descriptor is generated for the configuration.

For the following configuration:

```json
{
  "actions" : [
  {
    "type" : "generic_key",
    "descriptor_value" : "some_value"
  },
  {
    "type" : "remote_address"
  },
  {
    "type" : "souce_cluster"
  }
  ]
}
```

If a request did not set [x-forwarded-for](../http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for), no descriptor is generated.

If a request sets [x-forwarded-for](../http_conn_man/headers.md#config-http-conn-man-headers-x-forwarded-for), the the following descriptor is generated:

```
("generic_key", "some_value")
("remote_address", "<trusted address from x-forwarded-for>")
("source_cluster", "from_cluster")
```

## Statistics

The buffer filter outputs statistics in the *cluster.<route target cluster>.ratelimit.* namespace. 429 responses are emitted to the normal cluster [dynamic HTTP statistics](../cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-dynamic-http).

| Name       | Type    | Description                                             |
| ---------- | ------- | ------------------------------------------------------- |
| ok         | Counter | Total under limit responses from the rate limit service |
| error      | Counter | Total errors contacting the rate limit service          |
| over_limit | Counter | total over limit responses from the rate limit service  |

## Runtime

The HTTP rate limit filter supports the following runtime settings:

- ratelimit.http_filter_enabled

  % of requests that will call the rate limit service. Defaults to 100.

- ratelimit.http_filter_enforcing

  % of requests that will call the rate limit service and enforce the decision. Defaults to 100. This can be used to test what would happen before fully enforcing the outcome.

- ratelimit.<route_key>.http_filter_enabled

  % of requests that will call the rate limit service for a given *route_key* specified in the [rate limit configuration](../../api-v1/route_config/rate_limits.md#config-http-conn-man-route-table-rate-limit-config). Defaults to 100.