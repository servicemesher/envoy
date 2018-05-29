# 路由匹配

> **注意**
>
> 本节为 v1 API编写，但概念也适用于 v2 API。 未来的版本中将以 v2 API 为目标重写。

当 Envoy 匹配路由时，它使用如下步骤：

1. HTTP 请求的 *host* 或者 *:authority* 头匹配到[虚拟主机](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/vhost.html#config-http-conn-man-route-table-vhost)。
2. 按顺序检查虚拟主机中的每个[路由条目](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route.html#config-http-conn-man-route-table-route) 。如果匹配，则使用该路由而不再进一步检查其他路由。
3. 独立地，按顺序检查虚拟主机中的每个[虚拟集群](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/vcluster.html#config-http-conn-man-route-table-vcluster) 。如果匹配，则使用该虚拟集群而不再进一步检查其他虚拟集群。