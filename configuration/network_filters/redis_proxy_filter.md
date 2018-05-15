# Redis 代理

- Redis [architecture overview](../../intro/arch_overview/redis.md#arch-overview-redis)
- [v1 API reference](../../api-v1/network_filters/redis_proxy_filter.md#config-network-filters-redis-proxy-v1)
- [v2 API reference](../../api-v2/config/filter/network/redis_proxy/v2/redis_proxy.proto.md#envoy-api-msg-config-filter-network-redis-proxy-v2-redisproxy)

## Statistics

Every configured Redis proxy filter has statistics rooted at *redis.<stat_prefix>.* with the following statistics:

| Name                            | Type    | Description                                  |
| ------------------------------- | ------- | -------------------------------------------- |
| downstream_cx_active            | Gauge   | Total active connections                     |
| downstream_cx_protocol_error    | Counter | Total protocol errors                        |
| downstream_cx_rx_bytes_buffered | Gauge   | Total received bytes currently buffered      |
| downstream_cx_rx_bytes_total    | Counter | Total bytes received                         |
| downstream_cx_total             | Counter | Total connections                            |
| downstream_cx_tx_bytes_buffered | Gauge   | Total sent bytes currently buffered          |
| downstream_cx_tx_bytes_total    | Counter | Total bytes sent                             |
| downstream_cx_drain_close       | Counter | Number of connections closed due to draining |
| downstream_rq_active            | Gauge   | Total active requests                        |
| downstream_rq_total             | Counter | Total requests                               |

## Splitter statistics

The Redis filter will gather statistics for the command splitter in the *redis.<stat_prefix>.splitter.* with the following statistics:

| Name                | Type    | Description                                                  |
| ------------------- | ------- | ------------------------------------------------------------ |
| invalid_request     | Counter | Number of requests with an incorrect number of arguments     |
| unsupported_command | Counter | Number of commands issued which are not recognized by the command splitter |

## Per command statistics

The Redis filter will gather statistics for commands in the *redis.<stat_prefix>.command.<command>.* namespace.

| Name  | Type    | Description        |
| ----- | ------- | ------------------ |
| total | Counter | Number of commands |

## Runtime

The Redis proxy filter supports the following runtime settings:

- redis.drain_close_enabled

  % of connections that will be drain closed if the server is draining and would otherwise attempt a drain close. Defaults to 100.