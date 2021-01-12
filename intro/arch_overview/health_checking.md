# 健康检查

主动健康检查可以在每个上游集群的基础上进行[配置](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc)。如[服务发现](service_discovery.md#arch-overview-service-discovery)部分所述，主动运行状况检查和 EDS 服务发现类型会同时进行。但是，即使使用其他服务发现类型，也有其他需要进行主动健康检查的情况。Envoy 支持三种不同类型的健康检查及各种设置（检查时间间隔、主机不健康标记为故障、主机健康时标记为成功等）：

- **HTTP**：在 HTTP 健康检查期间，Envoy 将向上游主机发送 HTTP 请求。如果主机是健康的，会有 200 响应。如果上游主机想立即通知下游主机不再转发流量，则返回 503。
- **L3/L4**：在 L3/L4 健康检查期间，Envoy 会向上游主机发送一个可配置的字节缓冲区。如果主机被认为是健康的，字节缓冲区在响应中会被显示出来。Envoy 还支持仅连接 L3/L4 健康检查。
- **Redis**：Envoy 将发送 Redis PING 命令并期望 PONG 响应。如果上游 Redis 服务器使用 PONG 以外的任何其他响应命令，则会导致健康检查失败。或者，Envoy 可以在用户指定的密钥上执行 EXISTS。如果密钥不存在，则认为它是合格的健康检查。这允许用户通过将指定的密钥设置为任意值来标记 Redis 实例以进行维护直至流量耗尽。请参阅 [redis_key](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/health_checker/redis/v2/redis.proto#envoy-v3-api-msg-config-health-checker-redis-v2-redis)。

通过为集群传输指定的套接字进行健康检查。这意味着如果集群使用了启用 TLS 的传输套接字，则健康检查也将通过 TLS 进行。可以指定用于健康检查连接的 [TLS 选项](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/health_check.proto#envoy-v3-api-msg-config-core-v3-healthcheck-tlsoptions)，如果相应的上游将基于 ALPN 的 [FilterChainMatch](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchainmatch) 使用带有不同协议的健康检查与数据连接协议，则此选项很有用。

## 每个集群成员的健康检查配置

如果在上游集群配置了主动健康检查，通过在 [ClusterLoadAssignment](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/endpoint/v3/endpoint.proto#envoy-v3-api-msg-config-endpoint-v3-clusterloadassignment) 中的每个定义的 [LocalityLbEndpoints](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/endpoint/v3/endpoint_components.proto#envoy-v3-api-msg-config-endpoint-v3-localitylbendpoints) 的 [LbEndpoint](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/endpoint/v3/endpoint_components.proto#envoy-v3-api-msg-config-endpoint-v3-lbendpoint) 的 [Endpoint](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/endpoint/v3/endpoint_components.proto#envoy-v3-api-msg-config-endpoint-v3-endpoint) 设置 [HealthCheckConfig](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/endpoint/v3/endpoint_components.proto#envoy-v3-api-msg-config-endpoint-v3-endpoint-healthcheckconfig)，可以为每个已注册的成员指定额外的配置项。

通过设置健康检查配置以设置集群成员的备用健康检查端口的例子：

```yaml
load_assignment:
  endpoints:
  - lb_endpoints:
    - endpoint:
        health_check_config:
          port_value: 8080
        address:
          socket_address:
            address: localhost
            port_value: 80
```

## 健康检查事件日志

Envoy 可以选择在 [HealthCheck 配置](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/health_check.proto#envoy-v3-api-field-config-core-v3-healthcheck-event-log-path) 中指定日志文件路径，从而选择性地生成每个健康检查的输出和添加事件的日志。该日志的结构是 [HealthCheckEvent 消息](https://www.envoyproxy.io/docs/envoy/latest/api-v3/data/core/v3/health_check_event.proto#envoy-v3-api-msg-data-core-v3-healthcheckevent) 的 JSON 转储。

通过将 [always_log_health_check_failures 标记](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/health_check.proto#envoy-v3-api-field-config-core-v3-healthcheck-always-log-health-check-failures) 设置为 true，可以将 Envoy 配置为记录所有健康检查失败事件。

## 被动健康检查

Envoy 还支持通过[异常值检测](outlier.md#arch-overview-outlier-detection)进行被动健康检查。

## 连接池交互

请参阅[此处](connection_pooling.md#arch-overview-conn-pool-health-checking)了解更多信息

## HTTP 健康检查过滤器

当部署 Envoy 网格并在集群之间进行主动健康检查时，会生成大量健康检查流量。Envoy 包含一个HTTP 健康检查过滤器，可以安装在配置的 HTTP 侦听器中。这个过滤器有几种不同的操作模式：

- **不通过**：在此模式下，运行状况检查请求永远不会传递给本地服务。Envoy 会根据服务器当前的耗尽状态以200或503响应。
- **从上游集群运行健康状况计算得出的不通过**：在此模式下，运行健康检查筛选器将返回200或503，具体的值取决于在一个或多个上游集群中是否存在至少一个[指定百分比的服务器健康](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/health_check/v3/health_check.proto#envoy-v3-api-field-extensions-filters-http-health-check-v3-healthcheck-cluster-min-healthy-percentages)。（但是，如果 Envoy 服务器处于排空状态，则无论上游集群运行状况如何，它都将以503响应。）
- **通过**：在此模式下，Envoy 会将每个健康检查请求传递给本地服务。根据该服务的健康状态返回200或503。
- **通过缓存传递**：在此模式下，Envoy 会将健康检查请求传递给本地服务，但会将结果缓存一段时间。如果在缓存有效期，在随后的健康检查请求会直接获取缓存的值。缓存到期后，下一个运行健康检查请求将传递给本地服务。操作大型网格时，这是推荐的操作模式。Envoy 会保存进行健康检查的连接，因此健康检查请求对 Envoy 本身的成本很低。因此，这种操作模式产生了每个上游主机的健康状态的最终一致的视图，而没有用大量的健康检查请求压倒本地服务

进一步阅读：

- [健康检查过滤器配置](../../configuration/http_filters/health_check_filter.md#config-http-filters-health-check)。
- [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) 管理端点。
- [/healthcheck/ok](../../operations/admin.md#operations-admin-interface-healthcheck-ok) 管理端点。

## 主动健康检查快速失败

当同时使用主动健康检查和被动健康检查（[异常检测](outlier.md#arch-overview-outlier-detection)）时，通常使用较长的健康检查间隔来避免大量主动健康检查流量。在这种情况下，使用 [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) 管理端点时，它对能够快速排除上游主机仍然很有用。为此，[路由过滤器](../../configuration/http_filters/router_filter.md#config-http-filters-router)将响应 [x-envoy-immediate-health-check-fail](../../configuration/http_filters/router_filter.md#config-http-filters-router-x-envoy-immediate-health-check-fail) 头。如果此头由上游主机设置，Envoy 会立即将主机标记为主动健康检查失败。请注意，这种情况只有在主机集群[配置](../../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc)了主动健康检查时才会发生。如果 Envoy 已通过 [/healthcheck/fail](../../operations/admin.md#operations-admin-interface-healthcheck-fail) 管理端点被标记为失败时，[健康检查过滤器](../../configuration/http_filters/health_check_filter#config-http-filters-health-check)会自动设置此标头。

## 健康检查身份

只验证上游主机是否响应特定的运行状况检查 URL 并不一定意味着上游主机有效。例如，当在自动缩放的云环境或容器环境中使用最终一致的服务发现时，被检查主机可能会消失，但是其他主机会以相同的 IP 地址返回。解决此问题的一个办法是针对每种服务类型都有不同的 HTTP 健康检查 URL。该方法的缺点是整体配置会变得更加复杂，因为每个健康检查 URL 都是完全自定义的。

Envoy HTTP 健康检查器支持 [service_name_matcher](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/health_check.proto#envoy-v3-api-field-config-core-v3-healthcheck-httphealthcheck-service-name-matcher) 选项。如果设置了此选项，健康检查程序还会使用 *x-envoy-upstream-healthchecked-cluster* 响应标头的值与 *service_name_matcher* 进行比较。如果值不匹配，健康检查不通过。上游健康检查过滤器会将 *x-envoy-upstream-healthchecked-cluster* 附加到响应头。这个值由 [--service-cluster](../../operations/cli.md#cmdoption-service-cluster) 命令行选项决定。

## 健康降级

当使用 HTTP 健康检查器时，上游主机可以返回 `x-envoy-degraded` 以通知健康检查器主机已经降级。请参阅[此处](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/degraded#arch-overview-load-balancing-degraded)了解这是如何影响到负载均衡的。
