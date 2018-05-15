# 健康检查

- Health check filter [architecture overview](../../intro/arch_overview/health_checking.md#arch-overview-health-checking-filter)
- [v1 API reference](../../api-v1/http_filters/health_check_filter.md#config-http-filters-health-check-v1)
- [v2 API reference](../../api-v2/config/filter/http/health_check/v2/health_check.proto.md#envoy-api-msg-config-filter-http-health-check-v2-healthcheck)

Note

Note that the filter will automatically fail health checks and set the [x-envoy-immediate-health-check-fail](router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) header if the [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) admin endpoint has been called. (The [/healthcheck/ok](../../operations/admin.md#operations-admin-interface-healthcheck-ok)admin endpoint reverses this behavior).