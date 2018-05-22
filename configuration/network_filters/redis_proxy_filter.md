# Redis 代理

- Redis [架构概述](../../intro/arch_overview/redis.md#arch-overview-redis)
- [v1 接口文档](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/redis_proxy_filter#config-network-filters-redis-proxy-v1)
- [v2 接口文档](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/redis_proxy/v2/redis_proxy.proto#envoy-api-msg-config-filter-network-redis-proxy-v2-redisproxy)

## 统计

每个已配置的 Redis 代理过滤器都有以 *redis.<stat_prefix>.* 开头的统计，并提供如下的统计报告：

| Name                            | Type    | Description                                  |
| ------------------------------- | ------- | -------------------------------------------- |
| downstream_cx_active            | Gauge   | 活跃的连接总数                     |
| downstream_cx_protocol_error    | Counter | 协议错误总次数                        |
| downstream_cx_rx_bytes_buffered | Gauge   | 当前接收并缓存的总字节数      |
| downstream_cx_rx_bytes_total    | Counter | 接收的总字节数                         |
| downstream_cx_total             | Counter | 连接总数                            |
| downstream_cx_tx_bytes_buffered | Gauge   | 当前发送并缓存的总字节数          |
| downstream_cx_tx_bytes_total    | Counter | 发送的总字节数                             |
| downstream_cx_drain_close       | Counter | 因连接耗尽而关闭的连接总数 |
| downstream_rq_active            | Gauge   | 活跃的请求总数                        |
| downstream_rq_total             | Counter | 请求总数                               |

## 分离器统计

Redis 过滤器将采集命令分离器的统计信息，并存放到以 *redis.<stat_prefix>.splitter* 开头的统计中，提供如下的统计报告：

| Name                | Type    | Description                                                  |
| ------------------- | ------- | ------------------------------------------------------------ |
| invalid_request     | Counter | 参数个数不正确的请求数     |
| unsupported_command | Counter | 命令分离器无法识别的命令数 |

## 每个命令的统计

Redis 过滤器将收集每个命令的统计信息，并存放到以 *redis.<stat_prefix>.command.<command>* 开头的命名空间中，提供如下的统计报告：

| Name  | Type    | Description        |
| ----- | ------- | ------------------ |
| total | Counter | 命令数 |

## 运行时

Redis 代理过滤器支持如下的运行时设置：

- redis.drain_close_enabled
  
  当服务器因为连接耗尽而尝试关闭连接的百分比。 默认值是 100。