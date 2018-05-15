# 排除

Draining is the process by which Envoy attempts to gracefully shed connections in response to various events. Draining occurs at the following times:

- The server has been manually health check failed via the [healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) admin endpoint. See the [health check filter](health_checking.md#arch-overview-health-checking-filter) architecture overview for more information.
- The server is being [hot restarted](hot_restart.md#arch-overview-hot-restart).
- Individual listeners are being modified or removed via [LDS](dynamic_configuration.md#arch-overview-dynamic-config-lds).

Each [configured listener](listeners.md#arch-overview-listeners) has a [drain_type](../../api-v1/listeners/listeners.md#config-listeners-drain-type) setting which controls when draining takes place. The currently supported values are:

- default

  Envoy will drain listeners in response to all three cases above (admin drain, hot restart, and LDS update/remove). This is the default setting.

- modify_only

  Envoy will drain listeners only in response to the 2nd and 3rd cases above (hot restart and LDS update/remove). This setting is useful if Envoy is hosting both ingress and egress listeners. It may be desirable to set *modify_only* on egress listeners so they only drain during modifications while relying on ingress listener draining to perform full server draining when attempting to do a controlled shutdown.

Note that although draining is a per-listener concept, it must be supported at the network filter level. Currently the only filters that support graceful draining are [HTTP connection manager](../../configuration/http_conn_man/http_conn_man.md#config-http-conn-man), [Redis](../../configuration/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy), and [Mongo](../../configuration/network_filters/mongo_proxy_filter.md#config-network-filters-mongo-proxy).