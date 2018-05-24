# 全局速率限制

在分布式系统中，多数情况下，分布式[熔断](../circuit_breaking#arch-overview-circuit-break)对于吞吐的控制都是非常有效的；但是在有些例外情况下，是需要进行全局的速率控制的，例如大量主机（的请求）被转发给少量主机，并且平均请求延迟很低（比如访问数据库服务器的例子）。如果目标主机因故负载能力降低，下游主机就会影响到上游的集群。这种场景下，要不影响正常通信，又要防止主机故障引起的雪崩，想要为每个下游主机分别配置合适的断路器是很难的。这种案例就是使用全局速率控制的典型场景。

Envoy 直接集成了一个全局 gRPC 速率限制服务。所有实现了预定义 RPC/IDL 协议的服务都可以使用这一服务，Lyft 还提供了一个 [Go 编写的实现参考](https://github.com/lyft/ratelimit)，其中使用 Redis 作为后端。Envoy 的速率限制集成具有如下功能：

- *网络层的速率限制过滤器*：安装了这一过滤器的 Envoy，当监听器上新建连接的时候，就会调用速率限制服务。配置中会指定一个特定的域和描述符来设置速率限制。这种方式对监听器上的每秒连接次数进行了限制，从而达到了在监听器中进行速率限制的效果。[配置参考](../../configuration/network_filters/rate_limit_filter.md#config-network-filters-rate-limit)
- *HTTP 级的速率限制过滤器*：在安装了这一过滤器并且路由表要求调用全局速率限制服务的时候，Envoy 会在监听器的每次新请求的时候调用速率限制服务。所有到目标上游服务器、所有来自于源集群到目标集群的请求都可以进行速率限制。[配置参考](../../configuration/http_filters/rate_limit_filter.md#config-http-filters-rate-limit)

[速率限制服务配置](../../configuration/rate_limit.md#config-rate-limit-service)。