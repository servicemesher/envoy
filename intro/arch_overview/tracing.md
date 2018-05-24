# 追踪

## 概览

在较大的面向服务的架构中，分布式跟踪系统让开发者能够得到可视化的调用流程展示。在了解序列化、并行情况以及延迟分析的时候，这个功能至关重要。Envoy 用三个功能来支撑系统范围内的跟踪：

- *生成 Request ID*：Envoy 会在需要的时候生成 UUID，并操作名为 [x-request-id] 的 HTTP Header。应用可以转发这个 Header 用于统一的记录和跟踪。
- *集成外部跟踪服务*：Envoy 支持可插接的外部跟踪可视化服务。目前支持有 [LightStep](http://lightstep.com/)、[Zipkin](http://zipkin.io/) 或者 Zipkin 兼容的后端（比如说 [Jaeger](https://github.com/jaegertracing/)）。另外要添加其他的跟踪服务也并非难事。
- *客户端跟踪 ID 连接*：[x-client-trace-id](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-client-trace-id) Header 可以用来把不信任的请求 ID 连接到受信的 [x-request-id](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-request-id) Header 上。

## 如何初始化追踪

处理请求的 HTTP 连接管理器必须设置[跟踪](https://www.envoyproxy.io/docs/envoy/latest/api-v1/network_filters/http_conn_man#config-http-conn-man-tracing)对象。有多种途径可以初始化跟踪：

- 外部客户端，使用 [x-client-trace-id](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-client-trace-id) Header。
- 内部服务，使用 [x-envoy-force-trace](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-envoy-force-trace) Header。
- 随机采样使用运行时设置：[random_sampling](../../configuration/http_conn_man/runtime.md#config-http-conn-man-runtime-random-sampling)

路由过滤器可以使用 [start_child_span](https://www.envoyproxy.io/docs/envoy/latest/api-v1/http_filters/router_filter#config-http-filters-router-start-child-span) 来为 egress 调用创建下级 Span。

## 跟踪上下文信息的传播

Envoy 提供了报告网格内服务间通信跟踪信息的能力。然而一个调用流中，会有多个代理服务器生成的跟踪信息碎片，跟踪服务必须在出入站的请求中传播上下文信息，才能拼出完整的跟踪信息。

不管使用的是哪个跟踪服务，都应该传播 [x-request-id](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-request-id)，这样在被调用服务中启动相关性的记录。

为了理解 Span（本地的作业单元）之间的父子关系，跟踪服务自身也需要更多的上下文信息。这种需要可以通过在跟踪服务自身中直接使用 LightStep（OpenTracing API）或者 Zipkin 跟踪器来完成，从而在入站请求中提取跟踪上下文，然后把上下文信息注入到后续的出站请求中。这种方法还可以让服务能够创建附加的 Span，用来描述服务内部完成的工作。这对端到端跟踪的检查是很有帮助的。

另外跟踪上下文也可以被服务手工进行传播：

- 如果使用了 LightStep 跟踪器，在发送 HTTP 请求到其他服务时，Envoy 依赖这个服务来传播 [x-ot-span-context](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-ot-span-context) Header。
- 如果使用的是 Zipkin，Envoy 要传播的是 B3 Header。（[x-b3-traceid](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-traceid), [x-b3-spanid](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-spanid), [x-b3-parentspanid](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-parentspanid), [x-b3-sampled](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled), 以及 [x-b3-flags](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-flags).  [x-b3-sampled](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-b3-sampled)）也可以由外部客户端提出，用来启用或者禁用某个服务的跟踪请求。

## 每条追踪中包含哪些数据

端到端的跟踪会包含亦或者多个 Span。一个 Span 就是一个逻辑上的工作单元，具有一个启动时间和时长，其中还会包含特定的 Metadata，Envoy 生成的 Span 包含如下信息：

- 使用 [`--service-cluster`](../../operations/cli.md#cmdoption-service-cluster) 设置的始发服务集群。
- 请求的启动时间和持续时间。
- 使用 [`--service-node`](../../operations/cli.md#cmdoption-service-node) 设置的始发服务主机。
- [x-envoy-downstream-service-cluster](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-downstream-service-cluster) Header 设置的下游集群。
- HTTP URL。
- HTTP 方法。
- HTTP 响应码。
- 跟踪系统的特定信息。

Span 中还要包括一个名字（或者操作名），这一信息通常是由服务所属主机定义的。也可以在路由上使用 [Decorator](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-decorator) 来进行自定义。还可以使用 [x-envoy-decorator-operation](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-decorator-operation) Header 来设置名称。

Envoy 自动把 Span 发送给跟踪信息采集器。跟踪服务会使用其中的上下文信息来对多个 Span 进行拼装。例如[x-request-id](../../configuration/http_conn_man/headers.md#config-http-conn-man-headers-x-request-id) (LightStep) 或者 Zipkin 中的 trace ID 配置。

要获取更多 Envoy 跟踪设置方面的信息，可以阅读如下链接：

- [v1 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v1/tracing#config-tracing-v1)
- [v2 API 参考](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/trace.proto#envoy-api-msg-config-trace-v2-tracing)