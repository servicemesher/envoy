# 健康检查

Active health checking can be [configured](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc) on a per upstream cluster basis. As described in the [service discovery](service_discovery.md#arch-overview-service-discovery) section, active health checking and the SDS service discovery type go hand in hand. However, there are other scenarios where active health checking is desired even when using the other service discovery types. Envoy supports three different types of health checking along with various settings (check interval, failures required before marking a host unhealthy, successes required before marking a host healthy, etc.):

- **HTTP**: During HTTP health checking Envoy will send an HTTP request to the upstream host. It expects a 200 response if the host is healthy. The upstream host can return 503 if it wants to immediately notify downstream hosts to no longer forward traffic to it.
- **L3/L4**: During L3/L4 health checking, Envoy will send a configurable byte buffer to the upstream host. It expects the byte buffer to be echoed in the response if the host is to be considered healthy. Envoy also supports connect only L3/L4 health checking.
- **Redis**: Envoy will send a Redis PING command and expect a PONG response. The upstream Redis server can respond with anything other than PONG to cause an immediate active health check failure. Optionally, Envoy can perform EXISTS on a user-specified key. If the key does not exist it is considered a passing healthcheck. This allows the user to mark a Redis instance for maintenance by setting the specified key to any value and waiting for traffic to drain. See[redis_key](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-redis-key).

## 被动健康检查

Envoy also supports passive health checking via [outlier detection](outlier.md#arch-overview-outlier-detection).

## 链接池交互

See [here](connection_pooling.md#arch-overview-conn-pool-health-checking) for more information.

## HTTP 健康检查过滤器

When an Envoy mesh is deployed with active health checking between clusters, a large amount of health checking traffic can be generated. Envoy includes an HTTP health checking filter that can be installed in a configured HTTP listener. This filter is capable of a few different modes of operation:

- **No pass through**: In this mode, the health check request is never passed to the local service. Envoy will respond with a 200 or a 503 depending on the current draining state of the server.
- **No pass through, computed from upstream cluster health**: In this mode, the health checking filter will return a 200 or a 503 depending on whether at least a [specified percentage](../../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-field-config-filter-http-health-check-v2-healthcheck-cluster-min-healthy-percentages) of the servers are healthy in one or more upstream clusters. (If the Envoy server is in a draining state, though, it will respond with a 503 regardless of the upstream cluster health.)
- **Pass through**: In this mode, Envoy will pass every health check request to the local service. The service is expected to return a 200 or a 503 depending on its health state.
- **Pass through with caching**: In this mode, Envoy will pass health check requests to the local service, but then cache the result for some period of time. Subsequent health check requests will return the cached value up to the cache time. When the cache time is reached, the next health check request will be passed to the local service. This is the recommended mode of operation when operating a large mesh. Envoy uses persistent connections for health checking traffic and health check requests have very little cost to Envoy itself. Thus, this mode of operation yields an eventually consistent view of the health state of each upstream host without overwhelming the local service with a large number of health check requests.

Further reading:

- Health check filter [configuration](../../configuration/http_filters/health_check_filter.md#config-http-filters-health-check).
- [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) admin endpoint.
- [/healthcheck/ok](../../operations/admin.md#operations-admin-interface-healthcheck-ok) admin endpoint.

## 主动健康检查快速失败

When using active health checking along with passive health checking ([outlier detection](outlier.md#arch-overview-outlier-detection)), it is common to use a long health checking interval to avoid a large amount of active health checking traffic. In this case, it is still useful to be able to quickly drain an upstream host when using the [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) admin endpoint. To support this, the [router filter](../../configuration/http_filters/router_filter.md#config-http-filters-router) will respond to the [x-envoy-immediate-health-check-fail](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) header. If this header is set by an upstream host, Envoy will immediately mark the host as being failed for active health check. Note that this only occurs if the host’s cluster has active health checking [configured](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc). The [health checking filter](../../configuration/http_filters/health_check_filter.md#config-http-filters-health-check) will automatically set this header if Envoy has been marked as failed via the [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) admin endpoint.

## 健康检查身份

Just verifying that an upstream host responds to a particular health check URL does not necessarily mean that the upstream host is valid. For example, when using eventually consistent service discovery in a cloud auto scaling or container environment, it’s possible for a host to go away and then come back with the same IP address, but as a different host type. One solution to this problem is having a different HTTP health checking URL for every service type. The downside of that approach is that overall configuration becomes more complicated as every health check URL is fully custom.

The Envoy HTTP health checker supports the [service_name](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-service-name) option. If this option is set, the health checker additionally compares the value of the *x-envoy-upstream-healthchecked-cluster* response header to *service_name*. If the values do not match, the health check does not pass. The upstream health check filter appends *x-envoy-upstream-healthchecked-cluster* to the response headers. The appended value is determined by the [`--service-cluster`](../../operations/cli.md#cmdoption-service-cluster) command line option.