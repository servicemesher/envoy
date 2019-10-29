# Envoy 是什么？


Envoy 是专为大型现代 SOA（面向服务架构）架构设计的 L7 代理和通信总线。该项目源于以下理念：

> *网络对应用程序来说应该是透明的。当网络和应用程序出现问题时，应该很容易确定问题的根源。*


实际上，实现上述的目标是非常困难的。为了做到这一点，Envoy 提供了以下高级功能：

**进程外架构：**Envoy 是一个独立进程，设计为伴随每个应用程序服务运行。所有的 Envoy 形成一个透明的通信网格，每个应用程序发送消息到本地主机或从本地主机接收消息，但不知道网络拓扑。在服务间通信的场景下，进程外架构与传统的代码库方式相比，具有两大优点：


-  Envoy 可以使用任何应用程序语言。Envoy 部署可以在 Java、C++、Go、PHP、Python 等之间形成一个网格。面向服务架构使用多个应用程序框架和语言的趋势越来越普遍。Envoy 透明地弥合了它们之间的差异。
- 任何做过大型面向服务架构的人都知道，升级部署库可能会非常痛苦。Envoy可以透明地在整个基础架构上快速部署和升级。


**现代 C++11 代码库：**Envoy 是用 C++11 编写的。之所以选择（系统）原生代码是因为我们认为像 Envoy 这样的基础架构组件应该尽可能避让（资源争用）。由于在共享云环境中部署以及使用了非常有生产力但不是特别高效的语言（如 PHP、Python、Ruby、Scala 等），现代应用程序开发人员已经难以找出延迟的原因。原生代码通常提供了优秀的延迟属性，不会对已混乱的系统增加额外负担。与用 C 编写的其他原生代码代理的解决方案不同，C++11 具有出色的开发生产力和性能。


**L3/L4 filter 架构：**Envoy 的核心是一个 L3/L4 网络代理。可插入 [filter](arch_overview/network_filters.md#arch-overview-network-filters) 链机制允许开发人员编写 filter 来执行不同的 TCP 代理任务并将其插入到主体服务中。现在已有很多用来支持各种任务的 filter，如原始 [TCP 代理](arch_overview/tcp_proxy.md#arch-overview-tcp-proxy)、[HTTP 代理](arch_overview/http_connection_management.md#arch-overview-http-conn-man)、[TLS 客户端证书认证](arch_overview/ssl.md#arch-overview-ssl-auth-filter)等。


**HTTP L7 filter 架构：** HTTP 是现代应用程序体系结构的关键组件，Envoy [支持](arch_overview/http_filters.md#arch-overview-http-filters)额外的 HTTP L7 filter 层。可以将 HTTP filter 插入执行不同任务的 HTTP 连接管理子系统，例如[缓存](../configuration/http_filters/buffer_filter.md#config-http-filters-buffer)，[速率限制](arch_overview/global_rate_limiting.md#arch-overview-rate-limit)，[路由/转发](arch_overview/http_routing.md#arch-overview-http-routing)，嗅探 Amazon 的 [DynamoDB](arch_overview/dynamo.md#arch-overview-dynamo) 等等。


**顶级 HTTP/2 支持：** 当以 HTTP 模式运行时，Envoy 同时[支持](arch_overview/http_connection_management.md#arch-overview-http-protocols) HTTP/1.1 和 HTTP/2。Envoy 可以作为 HTTP/1.1 和 HTTP/2 之间的双向透明代理。这意味着它可以桥接 HTTP/1.1 和 HTTP/2 客户端以及目标服务器的任意组合。建议配置所有服务之间的 Envoy 使用  HTTP/2 来创建持久连接的网格，以便可以复用请求和响应。随着协议的逐步淘汰，Envoy 将不支持 SPDY。


**HTTP L7 路由：**当以 HTTP 模式运行时，Envoy 支持一种[路由](arch_overview/http_routing.md#arch-overview-http-routing)子系统，能够根据路径、权限、内容类型、[运行时](arch_overview/runtime.md#arch-overview-runtime)及参数值等对请求进行路由和重定向。这项功能在将 Envoy 用作前端/边缘代理时非常有用，同时，在构建服务网格时也会使用此功能。


**gRPC支持：**[gRPC](http://www.grpc.io/) 是一个来自 Google 的 RPC 框架，它使用 HTTP/2 作为底层多路复用传输协议。Envoy [支持](arch_overview/grpc.md#arch-overview-grpc)被 gRPC 请求和响应的作为路由和负载均衡底层的所有 HTTP/2 功能。这两个系统是非常互补的。


**MongoDB L7 支持：**[MongoDB](https://www.mongodb.com/) 是一种用于现代 Web 应用程序的流行数据库。Envoy [支持](arch_overview/mongo.md#arch-overview-mongo)对 MongoDB 连接进行 L7 嗅探、统计和日志记录。


**DynamoDB L7 支持**：[DynamoDB](https://aws.amazon.com/dynamodb/) 是亚马逊的托管键/值 NOSQL 数据存储。Envoy [支持](arch_overview/dynamo.md#arch-overview-dynamo)对 DynamoDB 连接进行 L7 嗅探和统计。


**服务发现和动态配置：** Envoy 可以选择使用[动态配置 API](arch_overview/dynamic_configuration.md#arch-overview-dynamic-config) 的分层集合实现集中管理。这些层为Envoy 提供了以下内容的动态更新：后端集群内的主机、后端集群本身、HTTP 路由、监听套接字和加密材料。对于更简单的部署，可以通过[DNS 解析](arch_overview/service_discovery.md#arch-overview-service-discovery)（甚至[完全跳过](arch_overview/service_discovery.md#arch-overview-service-discovery-types-sds)）发现后端主机，静态配置文件将替代更深的层。


**健康检查：**[推荐](arch_overview/service_discovery.md#arch-overview-service-discovery-eventually-consistent)使用将服务发现视为最终一致的过程的方式来建立 Envoy 网格。Envoy 包含了一个[健康检查](arch_overview/health_checking.md#arch-overview-health-checking)子系统，可以选择对上游服务集群执行主动健康检查。然后，Envoy 联合使用服务发现和健康检查信息来确定健康的负载均衡目标。Envoy 还通过[异常检测](arch_overview/outlier.md#arch-overview-outlier-detection)子系统支持被动健康检查。


**高级负载均衡：**[负载均衡](arch_overview/load_balancing.md#arch-overview-load-balancing)是分布式系统中不同组件之间的一个复杂问题。由于 Envoy 是一个独立代理而不是库，因此可以独立实现高级负载均衡以供任何应用程序访问。目前，Envoy 支持[自动重试](arch_overview/http_routing.md#arch-overview-http-routing-retry) 、[熔断](arch_overview/circuit_breaking.md#arch-overview-circuit-break)、通过外部速率限制服务的[全局速率限制](arch_overview/global_rate_limiting.md#arch-overview-rate-limit)、[请求映射](https://www.envoyproxy.io/docs/envoy/latest/api-v1/route_config/route#config-http-conn-man-route-table-route-shadow)和[异常点检测](arch_overview/outlier.md#arch-overview-outlier-detection)。未来还计划支持请求竞争。


**前端/边缘代理支持：**尽管 Envoy 主要设计用来作为一个服务间的通信系统，但在系统边缘使用相同的软件也是大有好处的（可观察性、管理、相同的服务发现和负载均衡算法等）。Envoy 包含足够多的功能，使其可作为大多数现代 Web 应用程序的边缘代理。这包括 [TLS](arch_overview/ssl.md#arch-overview-ssl) 终止、HTTP/1.1 和 HTTP/2 [支持](arch_overview/http_connection_management.md#arch-overview-http-protocols)，以及 HTTP L7 [路由](arch_overview/http_routing.md#arch-overview-http-routing)。


**最佳的可观察性：** 如上所述，Envoy 的主要目标是使网络透明。但是，问题在网络层面和应用层面都可能会出现。Envoy 包含对所有子系统强大的[统计](arch_overview/statistics.md#arch-overview-statistics)功能支持。目前支持 [statsd](https://github.com/etsy/statsd)（和兼容的提供程序）作为统计信息接收器，但是插入不同的接收器并不困难。统计信息也可以通过[管理](../operations/admin.md#operations-admin-interface)端口查看。Envoy 还通过第三方提供商支持分布式[追踪](arch_overview/tracing.md#arch-overview-tracing)。


## 设计目标


关于 Envoy 本身设计目标的简短说明：尽管 Envoy 绝对不慢（我们用了大量的时间来优化某些快速路径），我们的代码是按照模块化和易于测试的方式来编写的，而不是最大限度地实现最佳性能。我们认为这会更有效的利用时间，因为 Envoy 通常会和比它自身慢数倍、内存占用高数倍的语言和运行时部署在一起。
