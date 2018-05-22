# Jaeger 原生追踪


Jaeger 追踪沙箱展示了 Envoy 的 [请求追踪](../../intro/arch_overview/tracing.md#arch-overview-tracing) 能力，它使用 [Jaeger](http://jaegertracing.io/) 作为追踪的提供者，并且使用 Jaeger 的原生 [C++ 客户端](https://github.com/jaegertracing/jaeger-client-cpp) 作为插件。使用 Jaeger 及其原生客户端替代 Envoy 内置的 Zipkin 客户端有以下优势：


- Jaeger 可使追踪传播（trace propagation）与其他服务一起工作而无需进行配置 [更改](https://github.com/jaegertracing/jaeger-client-go#zipkin-http-b3-compatible-header-propagation)。
- 可以使用各种不同的 [采样策略](https://www.jaegertracing.io/docs/sampling/#client-sampling-configuration)，包括可以从 Jaeger 后端集中控制的概率采样策略和远程采样策略。
- Spans 以更高效的二进制编码发送到采集器。


此沙箱与上述前端代理体系结构非常相似，但有一点不同：在返回响应之前，service1 会对 service2 进行一次 API 调用。这三个容器将被部署在名为 `envoymesh` 的虚拟网络中。（注意：沙箱只能在 x86-64 上运行）。


所有传入的请求都通过前端 envoy 进行路由，该 envoy 充当位于 `envoymesh` 网络边缘的反向代理。端口 `80` 被 docker compose 映射到端口 `8000`（参见[/examples/jaeger-native-tracing/docker-compose.yml](https://github.com/envoyproxy/envoy/blob/master//examples/jaeger-native-tracing/docker-compose.yml)）。请注意，所有的 envoy 都被配置为收集请求跟踪信息（例如，[/examples/jaeger-native-tracing/front-envoy-jaeger.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/jaeger-native-tracing/front-envoy-jaeger.yaml) 中配置的 http_connection_manager/config/tracing），并将 Jaeger 追踪器生成的 span 传播到 Jaeger 集群中（跟踪驱动在 [/examples/jaeger-native-tracing/front-envoy-jaeger.yaml](https://github.com/envoyproxy/envoy/blob/master//examples/jaeger-native-tracing/front-envoy-jaeger.yaml) 中配置）。


在将请求路由到相应的服务 envoy 或应用之前，Envoy 将负责为追踪生成恰当的 span（父/子上下文 span）。在高层次上，每个 span 将记录上游 API 调用的延迟以及将该 span 与其他相关 span 进行关联所需的信息（例如 trace ID）。


Envoy 追踪最重要的好处之一是它将负责将追踪行为传播到 Jaeger 服务集群中。然而，为了充分利用追踪机制，在对其余服务进行请求时，应用必须传播 Envoy 生成的追踪 header。在我们提供的沙箱中，一个简单的 flask 应用（请参阅追踪函数 [/examples/front-proxy/service.py](https://github.com/envoyproxy/envoy/blob/master//examples/front-proxy/service.py)）将作为 service1，在对外请求 service2 时传播追踪 header。


## 运行沙箱


以下文档按照上图描述对一个 envoy 集群的配置过程进行了演练。

**步骤1：建立沙箱**

要构建这个沙箱示例并启动示例应用程序，请运行以下命令：

```bash
$ pwd
envoy/examples/jaeger-native-tracing
$ docker-compose up --build -d
$ docker-compose ps
        Name                       Command               State      Ports
-------------------------------------------------------------------------------------------------------------
jaegertracing_service1_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
jaegertracing_service2_1      /bin/sh -c /usr/local/bin/ ... Up       80/tcp
jaegertracing_front-envoy_1   /bin/sh -c /usr/local/bin/ ... Up       0.0.0.0:8000->80/tcp, 0.0.0.0:8001->8001/tcp
```


**步骤2：生成一些负载**

您现在可以通过前端 envoy（front-envoy）向 service1 发送请求，如下所示：

```bash
$ curl -v $(docker-machine ip default):8000/trace/1
*   Trying 192.168.99.100...
* Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
> GET /trace/1 HTTP/1.1
> Host: 192.168.99.100:8000
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 89
< x-envoy-upstream-service-time: 1
< server: envoy
< date: Fri, 26 Aug 2016 19:39:19 GMT
< x-envoy-protocol-version: HTTP/1.1
<
Hello from behind Envoy (service 1)! hostname: f26027f1ce28 resolvedhostname: 172.19.0.6
* Connection #0 to host 192.168.99.100 left intact
```


**步骤3：在 Jaeger UI 中查看追踪**

在浏览器中打开 <http://localhost:16686>。您应该可以看到 Jaeger 仪表盘。设置服务为 “front-proxy” 并点击 ‘Find Traces’。您应该可以看到从 front-proxy 发起的追踪信息。请单击一个追踪来查看从 front-proxy 到 service1 再到 service2 的请求路径，以及每一跳产生的延迟。