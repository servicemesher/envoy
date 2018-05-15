# 速率限制

- Global rate limiting [architecture overview](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)
- [v1 API reference](../../api-v1/network_filters/rate_limit_filter.md#config-network-filters-rate-limit-v1)
- [v2 API reference](../../api-v2/config/filter/network/rate_limit/v2/rate_limit.proto.md#envoy-api-msg-config-filter-network-rate-limit-v2-ratelimit)

## 统计

Every configured rate limit filter has statistics rooted at *ratelimit.<stat_prefix>.* with the following statistics:

| Name       | Type    | Description                                                  |
| ---------- | ------- | ------------------------------------------------------------ |
| total      | Counter | Total requests to the rate limit service                     |
| error      | Counter | Total errors contacting the rate limit service               |
| over_limit | Counter | Total over limit responses from the rate limit service       |
| ok         | Counter | Total under limit responses from the rate limit service      |
| cx_closed  | Counter | Total connections closed due to an over limit response from the rate limit service |
| active     | Gauge   | Total active requests to the rate limit service              |

## 运行时

The network rate limit filter supports the following runtime settings:

- ratelimit.tcp_filter_enabled

  % of connections that will call the rate limit service. Defaults to 100.

- ratelimit.tcp_filter_enforcing

  % of connections that will call the rate limit service and enforce the decision. Defaults to 100. This can be used to test what would happen before fully enforcing the outcome.