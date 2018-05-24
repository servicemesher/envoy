# 健康检查

- 健康检查过滤器[架构概述](../../intro/arch_overview/health_checking.md#arch-overview-health-checking-filter)
- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/health_check_filter#config-http-filters-health-check-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/health_check/v2/health_check.proto#envoy-api-msg-config-filter-http-health-check-v2-healthcheck)

> **注意**
>
> 请注意如果调用了 [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) 管理端点，过滤器将会自动地在健康检查中失败并标识 [x-envoy-immediate-health-check-fail](router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) 标头。（[/healthcheck/ok](../../operations/admin.md#operations-admin-interface-healthcheck-ok) 管理端点可以反转此行为）。