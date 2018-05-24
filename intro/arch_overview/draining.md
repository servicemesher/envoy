# 排除

排除是 Envoy 相应各种事件的要求试图优雅地关闭连接的过程。排除发生在以下时机：

- 服务器通过 [healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) 管理端点手动设置健康状况检查失败。有关更多信息，请参阅 [健康检查过滤器](health_checking.md#arch-overview-health-checking-filter) 架构概述。
- 服务器 [热重启](hot_restart.md#arch-overview-hot-restart)。
- 通过 [LDS](dynamic_configuration.md#arch-overview-dynamic-config-lds) 修改或者移除单个监听器。

每个 [已配置监听器](listeners.md#arch-overview-listeners) 有 [drain_type](https://www.envoyproxy.io/docs/envoy/latest/api-v1/listeners/listeners#config-listeners-drain-type) 设置，用来控制排除何时发生。目前支持的值有：

- default

  对于所有上述三种情况（管理排除，热重启和 LDS 更新/删除），Envoy 将排除监听器。这是默认设置。

- modify_only

  Envoy 只会响应上述第二和第三种情况（热重启和 LDS 更新/删除）而排除监听器。如果 Envoy 同时托管 ingress 和 egress 监听器，此设置很有用。需要在 egress 监听器上设置 *modify_only*，以便在尝试执行受控关闭，依赖  ingress 监听器排除来执行全服务器排除时，它们只在修改期间排除。

请注意，虽然排除是每监听器的概念，但它必须在网络过滤器级别被支持。目前支持优雅排除的过滤器只有 [HTTP 连接管理器](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man) 、[Redis ](../../configuration/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy)和 [Mongo](../../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy)。