# 统计

每个连接管理器都有一个以 * http.<stat_prefix>.* 为根的统计树，其统计信息如下：

| 名称                                          | 类型      | 描述                                                  |
| --------------------------------------------- | --------- | ------------------------------------------------------------ |
| downstream_cx_total                           | Counter   | 连接总数                                          |
| downstream_cx_ssl_total                       | Counter   | TLS 连接总数 |
| downstream_cx_http1_total                     | Counter   | HTTP/1.1 连接总数                                   |
| downstream_cx_websocket_total                 | Counter   | WebSocket 连接总数                                  |
| downstream_cx_http2_total                     | Counter   | HTTP/2 连接总数                                     |
| downstream_cx_destroy                         | Counter   | 被破坏的连接总数                                  |
| downstream_cx_destroy_remote                  | Counter   | 因为远程关闭而被破坏的连接总数              |
| downstream_cx_destroy_local                   | Counter   | 因为本地关闭而被破坏的连接总数               |
| downstream_cx_destroy_active_rq               | Counter   | 因为超过一个活跃连接而被破坏的连接总数           |
| downstream_cx_destroy_local_active_rq         | Counter   | 因为超过一个活跃连接而被本地破坏的连接总数   |
| downstream_cx_destroy_remote_active_rq        | Counter   | 因为超过一个活跃连接而被远程破坏的连接总数  |
| downstream_cx_active                          | Gauge     | 活跃连接总数                                     |
| downstream_cx_ssl_active                      | Gauge     | 活跃 TLS 连接总数                                 |
| downstream_cx_http1_active                    | Gauge     | 活跃 HTTP/1.1 连接总数                             |
| downstream_cx_websocket_active                | Gauge     | 活跃 WebSocket 连接总数                            |
| downstream_cx_http2_active                    | Gauge     | 活跃 HTTP/2 连接总数                               |
| downstream_cx_protocol_error                  | Counter   | 协议错误总数                                        |
| downstream_cx_length_ms                       | Histogram | 连接长度毫秒                               |
| downstream_cx_rx_bytes_total                  | Counter   | 收到的总字节数                                         |
| downstream_cx_rx_bytes_buffered               | Gauge     | 当前收到并缓存的总字节数                     |
| downstream_cx_tx_bytes_total                  | Counter   | 发出的总字节数                                             |
| downstream_cx_tx_bytes_buffered               | Gauge     | 当前发出并缓存的总字节数                          |
| downstream_cx_drain_close                     | Counter   | Total connections closed due to draining                     |
| downstream_cx_idle_timeout                    | Counter   | Total connections closed due to idle timeout                 |
| downstream_flow_control_paused_reading_total  | Counter   | Total number of times reads were disabled due to flow control |
| downstream_flow_control_resumed_reading_total | Counter   | Total number of times reads were enabled on the connection due to flow control |
| downstream_rq_total                           | Counter   | Total requests                                               |
| downstream_rq_http1_total                     | Counter   | Total HTTP/1.1 requests                                      |
| downstream_rq_http2_total                     | Counter   | Total HTTP/2 requests                                        |
| downstream_rq_active                          | Gauge     | Total active requests                                        |
| downstream_rq_response_before_rq_complete     | Counter   | Total responses sent before the request was complete         |
| downstream_rq_rx_reset                        | Counter   | Total request resets received                                |
| downstream_rq_tx_reset                        | Counter   | Total request resets sent                                    |
| downstream_rq_non_relative_path               | Counter   | Total requests with a non-relative HTTP path                 |
| downstream_rq_too_large                       | Counter   | Total requests resulting in a 413 due to buffering an overly large body |
| downstream_rq_1xx                             | Counter   | Total 1xx responses                                          |
| downstream_rq_2xx                             | Counter   | Total 2xx responses                                          |
| downstream_rq_3xx                             | Counter   | Total 3xx responses                                          |
| downstream_rq_4xx                             | Counter   | Total 4xx responses                                          |
| downstream_rq_5xx                             | Counter   | Total 5xx responses                                          |
| downstream_rq_ws_on_non_ws_route              | Counter   | Total WebSocket upgrade requests rejected by non WebSocket routes |
| downstream_rq_time                            | Histogram | Request time milliseconds                                    |
| rs_too_large                                  | Counter   | Total response errors due to buffering an overly large body  |

## Per user agent statistics

Additional per user agent statistics are rooted at *http.<stat_prefix>.user_agent.<user_agent>.*Currently Envoy matches user agent for both iOS (*ios*) and Android (*android*) and produces the following statistics:

| Name                                   | Type    | Description                                                  |
| -------------------------------------- | ------- | ------------------------------------------------------------ |
| downstream_cx_total                    | Counter | Total connections                                            |
| downstream_cx_destroy_remote_active_rq | Counter | Total connections destroyed remotely with 1+ active requests |
| downstream_rq_total                    | Counter | Total requests                                               |

## Per listener statistics

Additional per listener statistics are rooted at *listener.<address>.http.<stat_prefix>.* with the following statistics:

| Name              | Type    | Description         |
| ----------------- | ------- | ------------------- |
| downstream_rq_1xx | Counter | Total 1xx responses |
| downstream_rq_2xx | Counter | Total 2xx responses |
| downstream_rq_3xx | Counter | Total 3xx responses |
| downstream_rq_4xx | Counter | Total 4xx responses |
| downstream_rq_5xx | Counter | Total 5xx responses |

## Per codec statistics

Each codec has the option of adding per-codec statistics. Currently only http2 has codec stats.

### Http2 codec statistics

All http2 statistics are rooted at *http2.*

| Name                   | Type    | Description                                                  |
| ---------------------- | ------- | ------------------------------------------------------------ |
| header_overflow        | Counter | Total number of connections reset due to the headers being larger than Envoy::Http::Http2::ConnectionImpl::StreamImpl::MAX_HEADER_SIZE (63k) |
| headers_cb_no_stream   | Counter | Total number of errors where a header callback is called without an associated stream. This tracks an unexpected occurrence due to an as yet undiagnosed bug |
| rx_messaging_error     | Counter | Total number of invalid received frames that violated [section 8](https://tools.ietf.org/html/rfc7540#section-8) of the HTTP/2 spec. This will result in a *tx_reset* |
| rx_reset               | Counter | Total number of reset stream frames received by Envoy        |
| too_many_header_frames | Counter | Total number of times an HTTP2 connection is reset due to receiving too many headers frames. Envoy currently supports proxying at most one header frame for 100-Continue one non-100 response code header frame and one frame with trailers |
| trailers               | Counter | Total number of trailers seen on requests coming from downstream |
| tx_reset               | Counter | Total number of reset stream frames transmitted by Envoy     |

## Tracing statistics

Tracing statistics are emitted when tracing decisions are made. All tracing statistics are rooted at *http.<stat_prefix>.tracing.* with the following statistics:

| Name            | Type    | Description                                                  |
| --------------- | ------- | ------------------------------------------------------------ |
| random_sampling | Counter | Total number of traceable decisions by random sampling       |
| service_forced  | Counter | Total number of traceable decisions by server runtime flag *tracing.global_enabled* |
| client_enabled  | Counter | Total number of traceable decisions by request header *x-envoy-force-trace* |
| not_traceable   | Counter | Total number of non-traceable decisions by request id        |
| health_check    | Counter | Total number of non-traceable decisions by health check      |