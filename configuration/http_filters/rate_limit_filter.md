# 频率限制

- 全局频率限制[架构概览](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/rate_limit_filter#config-http-filters-rate-limit-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/rate_limit/v2/rate_limit.proto#envoy-api-msg-config-filter-http-rate-limit-v2-ratelimit)

如果一个请求的路由或者虚拟主机中设置了一个或多个符合过滤条件的[频率限制](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-rate-limits)，HTTP 频率限制过滤器会调用频率限制服务。[路由](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-rate-limits)能够可选地包含虚拟主机的频率限制配置。一个请求中可以被施加一个或者多个路由限制。每个配置都会生成一个发给频率限制服务的描述符。

调用频率限制服务之后，如果任何描述符得到了超限的返回值，那么这个服务就会生成一个 429 的状态码。

## 编写 Action

> **注意**
> 这一节是使用 v1 API 进行的，但其中的概念也是适用于 v2 API 的。未来会面向 v2 重写这部分内容。

每个路由或者虚拟主机上的的[频率限制 Action](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/rate_limits#config-http-conn-man-route-table-rate-limit-config) 都会生成一个描述符条目。描述符条目的向量组成一个描述符。Action 可以用任何顺序编写，可以创建更加复杂的频率限制描述符。描述符会按照配置中的 Action 顺序执行。

### 例一

例如，要生成下面的描述符：

    ("generic_key", "some_value")
    ("source_cluster", "from_cluster")

就需要编写这样的配置代码：

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

### 例二

如果一个 Action 没有加入描述符条目，那么这一配置就不会生成描述符。

例如下面的配置：

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

如果一个请求没有设置 [x-forwarded-for](../../configuration/http_conn_man/headers#config-http-conn-man-headers-x-forwarded-for)，不会生成描述符。

如果请求中设置了 [x-forwarded-for](../../configuration/http_conn_man/headers#config-http-conn-man-headers-x-forwarded-for)，会生成如下的描述符：

    ("generic_key", "some_value")
    ("remote_address", "<trusted address from x-forwarded-for>")
    ("source_cluster", "from_cluster")

## 统计

缓存过滤器输出会在 `cluster.<route target cluster>.ratelimit.` 命名空间输出统计数据。429 这一响应码也会出现在[动态 HTTP 统计数据](../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-dynamic-http)中。

|名称|类型|描述|
|---|---|---|
|ok|Counter|来自于频率限制服务的所有限制内响应数量|
|error|Counter|联系频率限制服务时的错误总数|
|over_limit|Counter|来自频率限制服务的所有超限响应数量|

## 运行时

HTTP 频率限制过滤器支持如下的运行时配置：

- *`ratelimit.http_filter_enabled`*

    调用频率限制服务的请求的百分比，缺省为 100。

- *`ratelimit.http_filter_enforcing`*

    调用频率限制服务并实施决策的请求的百分比。缺省为 100。这个选项可以用来在完全实施限制之前进行测试，从而了解频率限制实施产生的后果。

- *`ratelimit.<route_key>.http_filter_enabled`*

    在[频率限制配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/rate_limits#config-http-conn-man-route-table-rate-limit-config)中指定的 `route_key`，利用这个数值调用频率限制的请求的百分比。缺省值为 100。