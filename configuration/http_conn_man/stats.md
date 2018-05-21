# 统计

每个连接管理器都有一个以 *http.<stat_prefix>.* 为根的统计树，其统计信息如下：

| 名称                                          | 类型      | 描述                                                  |
| --------------------------------------------- | --------- | ------------------------------------------------------------ |
| downstream_cx_total                           | Counter   | 连接总数                                          |
| downstream_cx_ssl_total                       | Counter   | TLS 连接总数 |
| downstream_cx_http1_total                     | Counter   | HTTP/1.1 连接总数                                   |
| downstream_cx_websocket_total                 | Counter   | WebSocket 连接总数                                  |
| downstream_cx_http2_total                     | Counter   | HTTP/2 连接总数                                     |
| downstream_cx_destroy                         | Counter   | 被破坏的连接总数                                  |
| downstream_cx_destroy_remote                  | Counter   | 由于远程关闭，而导致被破坏的连接总数              |
| downstream_cx_destroy_local                   | Counter   | 由于本地关闭，而导致被破坏的连接总数               |
| downstream_cx_destroy_active_rq               | Counter   | 由于超过一个活跃请求，而导致被破坏的连接总数           |
| downstream_cx_destroy_local_active_rq         | Counter   | 由于超过一个活跃请求，而导致被本地破坏的连接总数   |
| downstream_cx_destroy_remote_active_rq        | Counter   | 由于超过一个活跃请求，而导致被远程破坏的连接总数  |
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
| downstream_cx_drain_close                     | Counter   | 由于删除，而导致被关闭的连接总数                     |
| downstream_cx_idle_timeout                    | Counter   | 由于空闲超时，而导致被关闭的连接总数                 |
| downstream_flow_control_paused_reading_total  | Counter   | 由于流量控制，而导致被禁止的总读取次数 |
| downstream_flow_control_resumed_reading_total | Counter   | 由于流量控制，而导致在连接上启用的总读取次数 |
| downstream_rq_total                           | Counter   | 请求总数                                               |
| downstream_rq_http1_total                     | Counter   | HTTP/1.1 总请求总数                                      |
| downstream_rq_http2_total                     | Counter   | HTTP/2 请求总数                                        |
| downstream_rq_active                          | Gauge     | 活跃请求总数                                        |
| downstream_rq_response_before_rq_complete     | Counter   | 在请求完成之前发送的总响应数         |
| downstream_rq_rx_reset                        | Counter   | 收到的请求重置总数                                |
| downstream_rq_tx_reset                        | Counter   | 发出的请求重置总数                                    |
| downstream_rq_non_relative_path               | Counter   | 带有非相对 HTTP 路径的请求总数                |
| downstream_rq_too_large                       | Counter   | 由于缓存过大正文，而导致 413 响应的请求总数 |
| downstream_rq_1xx                             | Counter   | 1xx 响应总数                                              |
| downstream_rq_2xx                             | Counter   | 2xx 响应总数                                              |
| downstream_rq_3xx                             | Counter   | 3xx 响应总数                                              |
| downstream_rq_4xx                             | Counter   | 4xx 响应总数                                              |
| downstream_rq_5xx                             | Counter   | 5xx 响应总数                                              |
| downstream_rq_ws_on_non_ws_route              | Counter   | 由于非 WebSocket 路由而被拒绝的 WebSocket 升级请求总数  |
| downstream_rq_time                            | Histogram | 请求时间(毫秒)                                    |
| rs_too_large                                  | Counter   | 由于缓存过大正文，而导致的错误响应总数  |

## 以 user agent 维度进行统计

以 user agent 维度进行的统计信息都以 *http.<stat_prefix>.user_agent.<user_agent>.* 开头。
目前 Envoy 匹配 iOS (*ios*) 以及 Android (*android*) 的 user agent ，并产生以下统计信息：

| 名称                                   | 类型     | 描述                                                         |
| -------------------------------------- | ------- | ------------------------------------------------------------ |
| downstream_cx_total                    | Counter | 连接总数                                                      |
| downstream_cx_destroy_remote_active_rq | Counter | 由于超过一个活跃请求，而导致被远程破坏的连接总数                     |
| downstream_rq_total                    | Counter | 请求总数                                                      |

## 以监听器维度进行统计

以监听器维度进行的统计信息都以 *listener.<address>.http.<stat_prefix>.* 开头，并有以下统计信息：

| 名称              | 类型     | 描述                |
| ----------------- | ------- | ------------------- |
| downstream_rq_1xx | Counter | 1xx 响应总数         |
| downstream_rq_2xx | Counter | 2xx 响应总数         |
| downstream_rq_3xx | Counter | 3xx 响应总数         |
| downstream_rq_4xx | Counter | 4xx 响应总数         |
| downstream_rq_5xx | Counter | 5xx 响应总数         |

## 以编解码器维度进行统计

每一个编解码器都有进行按编解码器维度进行统计的能力。目前只有 http2 有编解码器统计信息。

### Http2 编解码器统计

所有的 http2 统计信息都以 *http2.* 开头

| 名称                   | 类型     | 描述                                                         |
| ---------------------- | ------- | ------------------------------------------------------------ |
| header_overflow        | Counter | 由于头部大于 Envoy :: Http :: Http2 :: ConnectionImpl :: StreamImpl :: MAX_HEADER_SIZE（63k）而重置的连接总数 |
| headers_cb_no_stream   | Counter | 在没有关联流的情况下进行头部回调的错误总数。 由于尚未确诊的 bug，这会导致一些意外事件的发送 |
| rx_messaging_error     | Counter | 因为违背 HTTP/2 spec 中的 [section 8](https://tools.ietf.org/html/rfc7540#section-8) 而导致的无效接受帧总数。 这将导致 *tx_reset* |
| rx_reset               | Counter | Envoy 收到的重置流帧的总数        |
| too_many_header_frames | Counter | 由于接收太多头帧而导致 HTTP2 连接重置的总次数。 Envoy 支持代理最多一个 100-Continue 头部帧 ,一个 non-100 响应代码头部帧和一个拥有尾部的帧 |
| trailers               | Counter | 由下游请求所看到的尾部总数 |
| tx_reset               | Counter | Envoy 发送的重置流帧总数    |

## 追踪统计

追踪统计信息是在做出追踪决定时发出的。所有追踪统计信息都以 *http.<stat_prefix>.tracing.* 开头，并带有以下统计信息：

| 名称            | 类型     | 描述                                                         |
| --------------- | ------- | ------------------------------------------------------------ |
| random_sampling | Counter | 通过随机抽样可追踪决策的总数                                          |
| service_forced  | Counter | 通过服务器运行时标识  *tracing.global_enabled* 的可追踪决策的总数 |
| client_enabled  | Counter | 通过请求头部 *x-envoy-force-trace* 设定的可追踪决策的总数 |
| not_traceable   | Counter | 通过 request id 的不可追踪的决策总数        |
| health_check    | Counter | 通过健康检查的不可追踪的决策总数      |