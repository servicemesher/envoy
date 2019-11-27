# 故障注入

故障注入过滤器可以用来测试弹性的微服务架构中不同的错误形式，过滤器也可用用户自定义的错误代码来延时注入以及终止请求，从而提供了不同的失败场景下处理的能力，例如服务失败、服务过载、服务高延时、网络分区等。故障注入可以限定基于（目的地）上游集群中的一组特定的请求或者一组预定义的请求头。

故障可观察到的范围仅限于通过网络通信的应用程序。无法模拟本地主机上的CPU和磁盘故障。

目前，故障注入过滤器有如下局限性：

- 中止代码仅限于HTTP状态代码
- 延时仅限于固定时长

在未来的版本中将会包括对特殊路由限定故障的支持，基于分布注入*gRPC* 和 *HTTP/2* 的指定错误码和延时的持续时长

## 配置

注释

故障注入过滤器需要在其他过滤器之前注入，包括路由过滤器。

- [v1 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/fault_filter#config-http-filters-fault-injection-v1)
- [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/fault/v2/fault.proto#envoy-api-msg-config-filter-http-fault-v2-httpfault)

## 运行时

Http 错误注入器支持以下全局运行时配置：

- fault.http.abort.abort_percent

  如果请求头匹配，将按照百分比终止请求。默认添加 *abort_percent* 的配置。如果配置中不包含 *abort* 语句块，则 *abort_percent* 的默认值为0。

- fault.http.abort.http_status

  如果请求头匹配，HTTP 状态码将作为匹配时中止的状态码。默认添加 HTTP 状态码的配置。如果配置中不包含 *abort* 语句块，则 *http_status* 默认值为 0。

- fault.http.delay.fixed_delay_percent

  如果请求头匹配，则按照百分比延时请求。默认添加 *delay_percent* 的配置，否则为0。

- fault.http.delay.fixed_duration_ms

  延时持续毫秒数。如果不指定值，则 *fixed_duration_ms* 使用默认的配置。如果在运行时和配置中都缺少该字段，则不是用延时。

*备注*，如果存在指定的下游集群的故障过滤器运行时设置，将重写默认的运行时设置。以下是指定下游集群运行时的keys：

- fault.http.<downstream-cluster>.abort.abort_percent
- fault.http.<downstream-cluster>.abort.http_status
- fault.http.<downstream-cluster>.delay.fixed_delay_percent
- fault.http.<downstream-cluster>.delay.fixed_duration_ms

可以通过 [HTTP x-envoy-downstream-service-cluster](../http_conn_man/headers.md#config-http-conn-man-headers-downstream-service-cluster) 的 header 头获取下游集群的名称。如果在运行时中没有找到运行时配置，则使用全局的运行时配置作为当前配置。

## 统计

故障过滤器使用 *http.<stat_prefix>.fault.* 作为命名空间输出统计结果。这个统计前缀 [stat prefix](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-stat-prefix) 来源于 HTTP 连接管理器。

| 名称                                 | 类型    | 描述                                             |
| ------------------------------------ | ------- | ------------------------------------------------------- |
| delays_injected                      | Counter | 延迟的总请求数                        |
| aborts_injected                      | Counter | 中止的总请求数                |
| <downstream-cluster>.delays_injected | Counter | 指定的下游集群的延迟的总请求数   |
| <downstream-cluster>.aborts_injected | Counter | 指定的下游集群的中止的总请求数   |
