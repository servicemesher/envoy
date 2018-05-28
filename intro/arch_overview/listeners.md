# 监听器

Envoy 配置支持在单个进程中启用任意数量的监听器。通常建议每台机器运行一个 Envoy，而不必介意配置的监听器数量。这样运维更简单，而且只有单个统计来源。目前 Envoy 只支持 TCP 监听器。

每个监听器都独立配置有一些网络级别（L3/L4）的[过滤器](network_filters.md#arch-overview-network-filters)。当监听器接接收到新连接时，配置好的连接本地过滤器将被实例化，并开始处理后续事件。通用监听器架构用于执行绝大多数不同的代理任务（例如，[限速](global_rate_limiting.md#arch-overview-rate-limit)、[TLS客户端认证](ssl.md#arch-overview-ssl-auth-filter)、[HTTP 连接管理](http_connection_management.md#arch-overview-http-conn-man)、 MongoDB [sniffing](mongo.md#arch-overview-mongo)、 原始 [TCP 代理](tcp_proxy.md#arch-overview-tcp-proxy)等）。

监听器也可以选择性的配置某些[监听器过滤器](listener_filters.md#arch-overview-listener-filters)。这些过滤器的处理在网络过滤器之前进行，并有机会操纵连接元数据，通常会影响后续过滤器或集群处理连接的方式。

监听器也可以通过[监听器发现服务 (LDS)](../../configuration/listeners/lds.md#config-listeners-lds)动态获取。

监听器[配置](../../configuration/listeners/listeners.md#config-listeners)。