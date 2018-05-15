# 路由匹配

> **Attention**
>
> This section is written for the v1 API but the concepts also apply to the v2 API. It will be rewritten to target the v2 API in a future release.

When Envoy matches a route, it uses the following procedure:

1. The HTTP request’s *host* or *:authority* header is matched to a [virtual host](../../api-v1/route_config/vhost.html#config-http-conn-man-route-table-vhost).
2. Each [route entry](../../api-v1/route_config/route.html#config-http-conn-man-route-table-route) in the virtual host is checked, *in order*. If there is a match, the route is used and no further route checks are made.
3. Independently, each [virtual cluster](../../api-v1/route_config/vcluster.html#config-http-conn-man-route-table-vcluster) in the virtual host is checked, *in order*. If there is a match, the virtual cluster is used and no further virtual cluster checks are made.