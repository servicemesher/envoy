# 路由

HTTP 转发是用路由过滤器实现的。在 Envoy 环境中，几乎所有 HTTP 代理场景下都会使用到这一过滤器。该过滤器的主要职能就是执行[路由表](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route_config#config-http-conn-man-route-table)中的指令。在重定向和转发这两个主要任务之外，路由过滤器还需要处理重试、统计之类的任务。

[v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/router_filter#config-http-filters-router-v1)

[v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/router/v2/router.proto#envoy-api-msg-config-filter-http-router-v2-router)

## HTTP 头

不管是外发/请求，还是接收/响应的过程中，HTTP 头都是由路由过滤器消费和设置的，下面会介绍这些内容：

- [x-envoy-expected-rq-timeout-ms](#x-envoy-expected-rq-timeout-ms)
- [x-envoy-max-retries](#x-envoy-max-retries)
- [x-envoy-retry-on](#x-envoy-retry-on)
- [x-envoy-retry-grpc-on](#x-envoy-retry-grpc-on)
- [x-envoy-upstream-alt-stat-name](#x-envoy-upstream-alt-stat-name)
- [x-envoy-upstream-canary](#x-envoy-upstream-canary)
- [x-envoy-upstream-rq-timeout-alt-response](#x-envoy-upstream-rq-timeout-alt-response)
- [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms)
- [x-envoy-upstream-rq-per-try-timeout-ms](#x-envoy-upstream-rq-per-try-timeout-ms)
- [x-envoy-upstream-service-time](#x-envoy-upstream-service-time)
- [x-envoy-original-path](#x-envoy-original-path)
- [x-envoy-immediate-health-check-fail](#x-envoy-immediate-health-check-fail)
- [x-envoy-overloaded](#x-envoy-overloaded)
- [x-envoy-decorator-operation](#x-envoy-decorator-operation)

### x-envoy-expected-rq-timeout-ms

以毫秒为单位的时间，路由器要求请求在这一时长内完成。Envoy 把这个信息写入 HTTP Header，这样被代理的主机收到这一请求之后，就可以根据这一节的内容来判断是否超时，也就是快速失败。这一信息会用在内部请求中，按照 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms) 头或[路由超时](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-timeout)的顺序执行。

### x-envoy-max-retries

如果使用了[重试策略](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)，且没有指定重试次数，Envoy 缺省会重试一次。重试次数可以显式的在[路由重试配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)或者 `x-envoy-max-retries` 头中进行设置。如果[重试策略](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)没有配置，并且 [x-envoy-retry-on](#envoy-retry-on) 或者 [x-envoy-retry-grpc-on](#x-envoy-retry-grpc-on) 都没有开启，Envoy 就不会重试失败的请求。

Envoy 的重试功能，有一些需要注意的：

- 路由超时（通过 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms) 或者[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-timeout)进行设置）是*包含所有重试的*。如果设置请求超时为 3 秒钟，并且第一次请求消耗了 2.7 秒，那么重试（包含补偿）需要在 0.3 秒之内完成。这一设计的目的是避免重试和超时的同步激增。
- Envoy 使用了一种随机的递增算法，以 25 毫秒为基础单位进行补偿。首次重试会随机的选择 0 到 24 毫秒范围内的延时，第二次会在 0 到 74 毫秒之间，第三次就会发生 0 到 175 毫秒之间的延时。
- 如果最大重试次数同时在 HTTP 头和路由配置中都有设置，那么请求的最大重试次数会使用两个配置之中的最大值作为有效值。

### x-envoy-retry-on

如果外发请求中设置了这个 Header，Envoy 就会试着重试失败的请求（重试次数缺省为 1，可以使用 [x-envoy-max-retries](#x-envoy-max-retries) 或者[路由重试配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)进行控制）。`x-envoy-retry-on` 的值用于表达重试策略。可以使用 `,` 作为分隔符，同时支持多个条件，其中包括：

- *5xx*

    如果上游服务器响应了 5xx 的状态码，或者完全不响应（断开、复位或者读超时）。（包含 `connect-failure` 以及 `refused-stream`）。
  - *注意*：如果一个请求超出了 [x-envoy-expected-rq-timeout-ms](#x-envoy-expected-rq-timeout-ms)（会出现 504 错误码），Envoy 是不会重试的。如果在某个尝试消耗太多时间的时候继续重试，可以使用 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms)，这是一个请求的外缘时间限制，包含了相关的重试时间。
- *gateway-error*

    这个策略和 *5xx* 类似，但是只会在收到 502、503 或者 504 时进行重试。
- *connect-failure*

    Envoy 会在连接上游服务器失败的情况下进行重试（连接超时之类）（包含在 *5xx* 内）。
  - *注意*：连接失败/超时是 TCP 层而非请求层的问题。并不包含使用 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms) 或者[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-timeout)进行设置的请求超时。
- *retriable-4xx*

    如果上游服务器响应了一个可以重试的 4xx 状态码，Envoy 就会进行重试。目前这一类别只包含一个 409 状态。
  - *注意*：这种策略要当心。有时 409 是说明有乐观锁需要更新。这种情况下调用者不应该重试，而是应该读取然后重试其他的写入，否则自动重试可能会导致 409 的持续出现。
- *refused-stream*

    如果上游服务器使用 `REFUSED_STREAM` 错误码复位了数据流，Envoy 就会进行重试。这种复位类型的意义就是说明这种请求是可以安全的进行重试的（包含在 *5xx* 范围之内）

重试次数可以使用 [x-envoy-max-retries](#x-envoy-max-retries) 或者[重试策略](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)进行控制。

注意重试策略还可以在[路由层](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)使用。

缺省情况下，除非按照上述方案进行配置，否则 Envoy 不会进行重试操作。

### x-envoy-retry-grpc-on

为外发请求设置这个 Header，Envoy 在请求失败时就会重试（重试次数缺省为 1，可以使用 [x-envoy-max-retries](#x-envoy-max-retries) 或者[路由重试配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)进行控制）。gRPC 重试目前仅支持状态码包含在响应头中的情况。在尾部包含 gRPC 状态码的情况不会触发重试逻辑。可以使用 `,` 分隔的列表来指定多个策略。支持策略包括：

- *cancelled*

    如果 gRPC 响应头中的状态码是 `cancelled`（1），Envoy 会尝试进行重试。
- *deadline-exceeded*

    如果 gRPC 响应头中的状态码是 `deadline-exceeded`（4），Envoy 会尝试进行重试。
- *resource-exhausted*

    如果 gRPC 响应头中的状态码是 `resource-exhausted`（8），Envoy 会尝试进行重试。

在使用 x-envoy-retry-grpc-on header 的同时，可以使用 [x-envoy-max-retries](#x-envoy-max-retries) header 进行重试次数的限制。

注意重试策略还可以在[路由层](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-retry)使用。

缺省情况下，除非按照上述方案进行配置，否则 Envoy 不会进行重试操作。

### x-envoy-upstream-alt-stat-name

在外发请求中设置这些 Header，让 Envoy 将上游的响应码和计时统计发送到另外的统计树中，这对于 Envoy 以外的应用级别分类统计很有帮助。这方面的细节，在[输出树文档](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-alt-tree)中有进一步的解释。

这个概念比较容易和定义集群时指定的 [alt_stat_name](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#envoy-api-field-cluster-alt-stat-name) 混淆，它指定的是在统计树中根目录下集群的备用名称。

### x-envoy-upstream-canary

如果一个上游主机设置了这个 Header，路由会生成金丝雀服务专用的统计。[输出树文档](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-dynamic-http)中有更详细的说明。

### x-envoy-upstream-rq-timeout-alt-response

在外发请求中设置这个 Header，在请求超时的情况下 Envoy 设置一个 204 的返回码（而不是 504）。这个 Header 的值会被忽略，也就是说只要 Header 存在就可以了。可以参看 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms)。

### x-envoy-upstream-rq-timeout-ms

在外发请求中设置这个 Header，Envoy 会覆盖[路由配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-timeout)，超时时间使用毫秒为单位。参看 [x-envoy-upstream-rq-per-try-timeout-ms](#x-envoy-upstream-rq-per-try-timeout-ms)。

### x-envoy-upstream-rq-per-try-timeout-ms

在外发请求中设置这个 Header，Envoy 会为路由的请求设置一个每次尝试的超时，这个超时必须不大于全局的路由超时（参看 [x-envoy-upstream-rq-timeout-ms](#x-envoy-upstream-rq-timeout-ms)），否则就会被忽略。这使得调用者可以在进行重试的时候，设置一个更加严格的超时时间，从而控制整体的超时情况。

### x-envoy-upstream-service-time

上游主机处理请求所消耗的时间，单位是毫秒。如果客户端希望知道服务处理时间和网络延迟，这个信息就非常有用了。这个 Header 是在响应中设置的。

### x-envoy-original-path

如果路由使用了 [prefix_rewrite](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-prefix-rewrite)，Envoy 就会把原有路径的 Header 保存到这里，以便记录和除错。

### x-envoy-immediate-health-check-fail

如果上游主机返回了这个 Header（可以使任何值），Envoy 会立刻假设这个主机的[主动健康检查](../../intro/arch_overview/health_checking.md#arch-overview-health-checking)已经失败（如果集群已经[配置](../../configuration/cluster_manager/cluster_hc#config-cluster-manager-cluster-hc)了主动健康检查）。这个功能可以通过标准数据面的处理，让上游主机进入故障状态，而无需等待下一次的健康检查。健康检查会让主机重新进入健康状态。[健康检查概述](../../intro/arch_overview/health_checking.md#arch-overview-health-checking)中讲述了更多这方面内容。

### x-envoy-overloaded

如果上游设置了这个 Header，Envoy 就不会进行重试了。目前这个 Header 的值是无意义的，只要 Header 本身存在即可。另外 Envoy 如果遇到[管理模式](#config-http-filters-router-runtime-maintenance-mode)或者上游[熔断](../../intro/arch_overview/circuit_breaking#arch-overview-circuit-break)的情况，就会在下游响应中设置这个 Header。

### x-envoy-decorator-operation

如果入站请求中有这个 Header，他的值会覆盖任何本地由跟踪系统定义的这一值。类似的如果这个 Header 存在于一个外发响应中，他的值也会覆盖客户端的定义。

## 统计

路由器会在集群命名空间中（依赖集群在所选路由的定义）输出很多统计数据。参考[集群统计](../configuration/cluster_manager/cluster_stats#config-cluster-manager-cluster-stats)一节有更多这方面的信息。

路由过滤器在 `http.<stat_prefix>.` 命名空间输出统计信息。[前缀](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-stat-prefix)定义来自所属的 HTTP 连接管理器。

|名称|类型|描述|
|---|---|---|
|no_route|Counter|所有没有路由，返回 404 的请求总数|
|no_cluster|Counter|目标集群不存在并返回 404 的请求总数|
|rq_redirect|Counter|收到重定向响应的请求总数|
|rq_direct_response|Counter|直接响应的请求总数|
|rq_total|Counter|被路由的请求总数|

虚拟机群统计会被输出到 `vhost.<virtual host name>.vcluster.<virtual cluster name>.` 命名空间，包括了下列内容：

|名称|类型|描述|
|---|---|---|
|upstream_rq_<*xx>|Counter|HTTP 响应码（也就是 2xx、3xx 等）的聚合|
|upstream_rq_<*>|Counter|指定的 HTTP 响应码（也就是 201、302 等）|
|upstream_rq_time|Histogram|请求时间的毫秒数|

## 运行时

路由过滤器支持下列运行时设置：

- *`upstream.base_retry_backoff_ms`*
    基础的重试时间计数，[HTTP 路由](../intro/arch_overview/http_routing#arch-overview-http-routing-retry)一节有更多讲述。缺省为 25 毫秒。
- *`upstream.maintenance_mode.<cluster name>`*
    会立即得到 503 响应的请求的百分比。他会覆盖所有定义了集群名称的路由行为。可以用于负载清洗、故障注入等。缺省是禁用的。
- *`upstream.use_retry`*
    要进行重试的请求的百分比。这一配置会在任何重试配置之前检查，可以在需要的时候完全禁止任何重试。