# HTTP 路由

Envoy includes an HTTP [router filter](../../configuration/http_filters/router_filter.md#config-http-filters-router) which can be installed to perform advanced routing tasks. This is useful both for handling edge traffic (traditional reverse proxy request handling) as well as for building a service to service Envoy mesh (typically via routing on the host/authority HTTP header to reach a particular upstream service cluster). Envoy also has the ability to be configured as forward proxy. In the forward proxy configuration, mesh clients can participate by appropriately configuring their http proxy to be an Envoy. At a high level the router takes an incoming HTTP request, matches it to an upstream cluster, acquires a [connection pool](connection_pooling.md#arch-overview-conn-pool) to a host in the upstream cluster, and forwards the request. The router filter supports the following features:

- Virtual hosts that map domains/authorities to a set of routing rules.
- Prefix and exact path matching rules (both [case sensitive](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-case-sensitive) and case insensitive). Regex/slug matching is not currently supported, mainly because it makes it difficult/impossible to programmatically determine whether routing rules conflict with each other. For this reason we don’t recommend regex/slug routing at the reverse proxy level, however we may add support in the future depending on demand.
- [TLS redirection](../../api-v1/route_config/vhost.md#config-http-conn-man-route-table-vhost-require-ssl) at the virtual host level.
- [Path](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-path-redirect)/[host](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-host-redirect) redirection at the route level.
- [Direct (non-proxied) HTTP responses](#arch-overview-http-routing-direct-response) at the route level.
- [Explicit host rewriting](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-host-rewrite).
- [Automatic host rewriting](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-auto-host-rewrite) based on the DNS name of the selected upstream host.
- [Prefix rewriting](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-prefix-rewrite).
- [Websocket upgrades](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-use-websocket) at route level.
- [Request retries](#arch-overview-http-routing-retry) specified either via HTTP header or via route configuration.
- Request timeout specified either via [HTTP header](../../configuration/http_filters/router_filter.md#config-http-filters-router-headers) or via [route configuration](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-timeout).
- Traffic shifting from one upstream cluster to another via [runtime values](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-runtime) (see [traffic shifting/splitting](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting)).
- Traffic splitting across multiple upstream clusters using [weight/percentage-based routing](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-weighted-clusters) (see [traffic shifting/splitting](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting-split)).
- Arbitrary header matching [routing rules](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-headers).
- Virtual cluster specifications. A virtual cluster is specified at the virtual host level and is used by Envoy to generate additional statistics on top of the standard cluster level ones. Virtual clusters can use regex matching.
- [Priority](#arch-overview-http-routing-priority) based routing.
- [Hash policy](../../api-v1/route_config/route.md#config-http-conn-man-route-table-hash-policy) based routing.
- [Absolute urls](../../api-v1/network_filters/http_conn_man.md#config-http-conn-man-http1-settings) are supported for non-tls forward proxies.

## 路由表

The [configuration](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man) for the HTTP connection manager owns the [route table](../../api-v1/route_config/route_config.md#config-http-conn-man-route-table) that is used by all configured HTTP filters. Although the router filter is the primary consumer of the route table, other filters also have access in case they want to make decisions based on the ultimate destination of the request. For example, the built in rate limit filter consults the route table to determine whether the global rate limit service should be called based on the route. The connection manager makes sure that all calls to acquire a route are stable for a particular request, even if the decision involves randomness (e.g. in the case of a runtime configuration route rule).

## 重试语义

Envoy allows retries to be configured both in the [route configuration](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route-retry) as well as for specific requests via [request headers](../../configuration/http_filters/router_filter.md#config-http-filters-router-headers). The following configurations are possible:

- **Maximum number of retries**: Envoy will continue to retry any number of times. An exponential backoff algorithm is used between each retry. Additionally, *all retries are contained within the overall request timeout*. This avoids long request times due to a large number of retries.
- **Retry conditions**: Envoy can retry on different types of conditions depending on application requirements. For example, network failure, all 5xx response codes, idempotent 4xx response codes, etc.

Note that retries may be disabled depending on the contents of the [x-envoy-overloaded](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded).

## 优先级路由

Envoy supports priority routing at the [route](../../api-v1/route_config/route.md#config-http-conn-man-route-table-route) level. The current priority implementation uses different [connection pool](connection_pooling.md#arch-overview-conn-pool) and [circuit breaking](../../configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers) settings for each priority level. This means that even for HTTP/2 requests, two physical connections will be used to an upstream host. In the future Envoy will likely support true HTTP/2 priority over a single connection.

The currently supported priorities are *default* and *high*.

## 直接响应

Envoy supports the sending of “direct” responses. These are preconfigured HTTP responses that do not require proxying to an upstream server.

There are two ways to specify a direct response in a Route:

- Set the [direct_response](../../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-direct-response) field. This works for all HTTP response statuses.
- Set the [redirect](../../api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-redirect) field. This works for redirect response statuses only, but it simplifies the setting of the *Location* header.

A direct response has an HTTP status code and an optional body. The Route configuration can specify the response body inline or specify the pathname of a file containing the body. If the Route configuration specifies a file pathname, Envoy will read the file upon configuration load and cache the contents.

Attention

If a response body is specified, it must be no more than 4KB in size, regardless of whether it is provided inline or in a file. Envoy currently holds the entirety of the body in memory, so the 4KB limit is intended to keep the proxy’s memory footprint from growing too large.

If **response_headers_to_add** has been set for the Route or the enclosing Virtual Host, Envoy will include the specified headers in the direct HTTP response.