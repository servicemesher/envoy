# HTTP路由

Envoy 包括一个 HTTP [路由器过滤器](../../configuration/http_filters/router_filter.md#config-http-filters-router)，可以安装它来执行高级路由任务。这对于处理边缘流量（传统的反向代理请求处理）以及构建服务间的 Envoy 网格（通常是通过主机/授权 HTTP 头的路由到达特定的上游服务集群）非常有用。Envoy 也可以被配置为转发代理。在转发代理配置中，网格客户端可以通过适当地配置其 http 代理来成为 Envoy。路由在一个高层级接受一个传入的 HTTP 请求，将其与上游集群相匹配，在上游集群中获得一个[连接池](../../intro/arch_overview/connection_pooling.md#arch-overview-conn-pool)，并转发请求。路由过滤器支持以下特性：

- 将域/授权映射到一组路由规则的虚拟主机。
- 前缀和精确路径匹配规则（包括[大小写敏感](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routematch-case-sensitive)和大小写不敏感）。目前还不支持正则/slug 匹配，这主要是因为难以用编程方式确定路由规则是否相互冲突。因此我们并不建议在反向代理级别使用正则/slug 路由，但是我们将来可能会根据用户需求量增加对这个特性的支持。
- 在虚拟主机级别上的 [TLS 重定向](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-virtualhost-require-tls)。
- 在路由级别的[路径/主机](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-redirectaction-host-redirect)重定向。
- 在路由级别的[直接（非代理）HTTP 响应](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/http/http_routing#arch-overview-http-routing-direct-response)。
- [显式主机重写](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/aws_request_signing/v3/aws_request_signing.proto#envoy-v3-api-field-extensions-filters-http-aws-request-signing-v3-awsrequestsigning-host-rewrite)。
- [自动主机重写](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routeaction-auto-host-rewrite)，基于所选的上游主机的 DNS 名称。
- [前缀重写](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-redirectaction-prefix-rewrite)。
- [使用正则表达式和匹配组进行路径重写](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routeaction-regex-rewrite)。
- 通过 HTTP 头部或路由配置指定[请求重试](#重试语义)。
- 通过 [HTTP头部](../../configuration/http_filters/router_filter.md#config-http-filters-router-headers) 或[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routeaction-timeout)指定请求超时。
- [请求对冲](#请求对冲)重试以响应请求(每次尝试)超时。
- 通过[运行时](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routematch-runtime-fraction)从一个上游集群转移到另一个集群（参见[流量转移/分割](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting)）。
- 使用基于[权重/百分比的路由](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-weighted-clusters)的流量跨越多个上游集群（参见[流量转移/分割](../../configuration/http_conn_man/traffic_splitting.md#config-http-conn-man-route-table-traffic-splitting-split)）。
- 任意头部的[路由规则](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-headermatcher)匹配。
- 虚拟集群规范。虚拟集群是在虚拟主机级别上指定的，由 Envoy  使用，在标准集群级别上生成额外的统计信息。虚拟集群可以使用正则表达式匹配。
- 基于[优先级](#优先级路由)的路由。
- 基于[哈希](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routeaction-hash-policy)策略的路由。
- [绝对路径](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-http-protocol-options)支持非 tls 转发代理。

## 路由作用域

作用域路由使 Envoy 可以在域名的搜索空间和路由规则上设置约束。路由作用域用一个键与路由表相关联。对于每个请求，HTTP 连接管理器都会动态计算出一个作用域键用以选择路由表。可以根据 [v3 API参考](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/on_demand/v3/on_demand.proto#envoy-v3-api-msg-extensions-filters-http-on-demand-v3-ondemand) 配置按需加载与路由作用域相关的 RouteConfiguration，并且按需提交 protobuf 设置为true。

作用域 RDS(SRDS) API 包含了一组[作用域](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/scoped_route.proto#envoy-v3-api-msg-config-route-v3-scopedrouteconfiguration)资源，每个资源定义了独立的路由配置，以及一个 [ScopeKeyBuilder](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-msg-extensions-filters-network-http-connection-manager-v3-scopedroutes-scopekeybuilder) 定义了 Envoy 用于查找域每个请求相对应的作用域键构造算法。

例如，对于以下作用域内的路由配置，Envoy 将查看“addr”头部值，首先将头部值根据“;”拆分，并使用第一个‘x-foo-key’键的值做为作用域键。如果“addr”头部值为“foo=1;x-foo-key=127.0.0.1;x-bar-key=1.1.1.1”，那么将会计算出“127.0.0.1”做为作用域键用于查找相应的路由配置。

```yaml
fragments:
  - header_value_extractor:
      name: Addr
      element_separator: ;
      element:
        key: x-foo-key
        separator: =
```

为了使键能够与 [ScopedRouteConfiguration](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/scoped_route.proto#envoy-v3-api-msg-config-route-v3-scopedrouteconfiguration) 匹配，计算出的键片段数必须与 [ScopedRouteConfiguration](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/scoped_route.proto#envoy-v3-api-msg-config-route-v3-scopedrouteconfiguration) 中的相匹配。然后按顺序匹配片段。构建键中缺失的片段（视为NULL）会使请求无法匹配到任何作用域，即无法为请求找到任何的路由入口。

## 路由表

HTTP 连接管理器的[配置](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/http_conn_man#config-http-conn-man)拥有所有已配置的 HTTP 过滤器使用的[路由表](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route.proto#envoy-v3-api-msg-config-route-v3-routeconfiguration)。尽管路由器过滤器是路由表的主要使用者，但是其他过滤器也有访问权限，以防它们想根据请求的最终目的地做出决策。例如，内置的限速过滤器查询路由表，以确定是否应该根据路由调用全局限速服务。连接管理器确保对特定请求的所有调用都是稳定的，即使决策涉及到随机性（例如，在运行时配置路由规则的情况下）。

## 重试语义

Envoy 允许在[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-routeaction-retry-policy)和通过[请求头](../../configuration/http_filters/router_filter.md#config-http-filters-router-headers)的特定请求中配置重试。以下配置是可能的：

- **最大数量的重试**：Envoy 将持续进行多次尝试。每次重试的间隔通过使用一个指数回溯算法（默认）决定，或者基于上游服务的头部（如果存在）。此外，*所有重试都包含在总体请求超时中*。这避免了由于大量重试而导致的长时间请求。
- **重试条件**：Envoy 可以根据应用程序的要求对不同类型的条件进行重试。例如，网络故障，所有 5xx 响应代码、幂等 4xx 响应码等等。
- **重试预算**：Envoy 可以通过重试预算来限制活跃请求的比例以防止流量的大幅度增长。
- **主机选择重试插件**：可以为 Envoy 的主机选择逻辑配置额外的逻辑，在重试选择主机时生效。指定重试主机谓词可以在选择某些主机时（例如在选择已尝试过的主机时）重新尝试选择主机，当选择优先级进行重试时可以配置[重试优先级](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-field-config-route-v3-retrypolicy-retry-priority)以调整优先级负载。

需要注意 Envoy 在 [x-envoy-overloaded](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-overloaded) 存在时才重试请求。建议配置[重试预算（首选）](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster/circuit_breaker.proto#envoy-api-field-cluster-circuitbreakers-thresholds-retry-budget)或者设置[最大活跃请求断路器](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/circuit_breaking#arch-overview-circuit-break)为设当的值以避免发生重试风暴。

## 请求对冲

Envoy 可以通过指定对冲策略启用请求对冲。这意味着 Envoy 将争用多个同时发生的上游请求，并将与第一个与可接受的响应头部相关联的响应返回到下游。重试策略用于确定是否应返回响应或是否应等待更多的响应。

当前，对冲只能在请求超时时响应执行。这意味着，将在不取消初始超时请求的情况下发出重试请求，并且将等待延迟响应。根据重试策略第一个“好”的响应将在下游返回。

该实现确保相同的上游请求不会重试两次。如果请求超时，导致响应5xx并创建两个可重试事件时，则可能发生这种情况。

## 优先级路由

Envoy 支持[路由](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-route)级别的优先路由。当前的优先级实现为每个优先级使用不同的[连接池](../../intro/arch_overview/connection_pooling.md#arch-overview-conn-pool)和[断路设置](../../configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers)。这意味着，即使对于 HTTP/2 请求，也将使用两个物理连接到上游主机。在未来，Envoy 可能会通过单一连接支持真正的 HTTP/2 优先级。

当前支持的优先级是*默认的*和*高的*。

## 直接响应

Envoy 支持发送“直接”回应。这些是预先配置的 HTTP 响应，不需要代理到上游服务器。

有两种方法可以在一条路由中指定直接响应：

- 设置 [direct_response](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto#envoy-api-field-route-route-direct-response) 字段。这适用于所有 HTTP 响应状态。
- 设置 [redirect](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto.md#envoy-api-field-route-route-redirect) 字段。这只适用于重定向响应状态，但它简化了*位置*头的设置。

直接响应有一个 HTTP 状态码和一个可选体。路由配置可以内联地指定响应体，或者指定包含体的文件的路径名。如果路线配置指定了一个文件路径名，Envoy  将在配置负载上读取文件并缓存内容。

> **注意**
>  
> 如果指定了响应体，那么它的大小必须不超过 4KB ，不管它是内联还是在文件中。Envoy 目前将所有正文保存在内存中，所以 4KB 的限制本意上是为了防止代理的内存占用太大。

如果路由或相关虚拟主机上设置了 **response_headers_to_add** ，Envoy 将在直接 HTTP 响应中包含指定的标头。
