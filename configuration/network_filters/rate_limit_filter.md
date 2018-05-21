# 速率限制

- 全局速率限制[架构概述](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/rate_limit_filter.html#config-network-filters-rate-limit-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/rate_limit/v2/rate_limit.proto.html#envoy-api-msg-config-filter-network-rate-limit-v2-ratelimit)

## 统计

所有配置的的速率限制过滤器都有以 *ratelimit.<stat_prefix>.* 开头的统计，以提供以下的统计报告：


| 名称       | 类型    | 描述                                                 |
| ---------- | ------- | ------------------------------------------------------------ |
| total      | Counter | 所有发给速率限制服务的请求数                    |
| error      | Counter | 所有联系速率限制服务的错误数              |
| over_limit | Counter | 所有由速率限制服务回复的超过速率限制的响应数       |
| ok         | Counter | 所有由速率限制服务回复的未超过速率限制的响应数      |
| cx_closed  | Counter | 所有因超过速率限制相应而被关闭的连接数 |
| active     | Gauge   | 所有发给速率限制服务的活跃请求数              |

## 运行时

网络级别速率限制过滤器支持以下运行时配置：

- ratelimit.tcp_filter_enabled

  将调用速率限制服务的连接百分比。缺省值为100。

- ratelimit.tcp_filter_enforcing

  将调用速率限制服务并强制执行决定的连接百分比。缺省值为100。这可以在完全执行结果之前，测试将会发生什么。