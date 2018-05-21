# 异常点检测

异常点检测和驱逐是用来动态确定上游集群中是否有主机的表现不同于其他主机的过程，并将它们从健康[负载均衡](load_balancing.md#arch-overview-load-balancing)集中移除。性能可能沿着不同的轴，例如连续失败、一时的成功率下降、一时的延迟等。异常检测是被动健康检查的一种形式。Envoy 还支持[主动健康检查](health_checking.md#arch-overview-health-checking)。被动和主动健康检查既可以一起也可以独立使用，它们共同形成整体上游健康检查解决方案的基础。

## 驱逐算法

这取决于异常点检测的类型，驱逐或者以内联（例如在连续的 5xx 的情况下）或以指定的间隔（例如在定期成功率的情况下）运行。驱逐算法的工作原理如下：

1. 主机被确定为异常点。
2. 如果没有主机被驱逐，Envoy 会立即驱逐主机。否则，它会检查以确保驱逐主机的数量低于允许的阈值（通过 [outlier_detection.max_ejection_percent](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection) 设置指定）。如果驱逐的主机数量超过阈值，则主机不会被驱逐。
3. 主机被驱逐需要几毫秒。主机被驱逐意味着主机被标记为不健康，并且在负载均衡期间不会被使用，除非负载均衡器处于恐慌情形中。毫秒数等于 [outlier_detection.base_ejection_time_msvalue](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection) 乘以主机被驱逐的次数。这会导致主机主机被弹出需要越来越长的时间（如果它们继续失败）。
4. 被驱逐的主机将在弹出时间满足后自动重新投入使用。通常，异常检测与主动健康检查一起用于全面的健康检查解决方案。

## 检测类型

Envoy 支持以下的异常点检测类型：

### 连续的 5xx

如果上游主机返回一些连续的 5xx，它将被驱逐。请注意，在这种情况下，5xx 意味着实际的 5xx 响应代码，或者会导致 HTTP 路由器代表上游返回一个事件（重置、连接失败等）的事件。主机被弹出时所需的连续 5xx 数量由 [outlier_detection.consecutive_5xx](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection-consecutive-5xx) 值控制。

### 连续的网关故障

如果上游主机返回一些连续的“网关错误”（502、503 或 504 状态码），它将被驱逐。请注意，这包括会导致 HTTP 路由器代表上游返回其中一个状态码的事件（重置、连接失败等）。主机被驱逐所需的连续网关故障的数量由 [outlier_detection.consecutive_gateway_failure](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection-consecutive-gateway-failure) 值控制。

### 成功率

基于成功率的异常值驱逐聚合来自集群中每个主机的成功率数据。然后以给定的间隔基于统计异常值检测来驱逐主机。如果主机在聚合时间间隔内的请求量小于 [outlier_detection.success_rate_request_volume](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) 值，则无法为主机计算成功率异常点驱逐。此外，如果某个时间间隔内所需的最小请求量的主机数小于 [outlier_detection.success_rate_minimum_hosts](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection-success-rate-minimum-hosts) 值，则不会对集群执行检测。

## 驱逐事件记录

Envoy 可以选择生成异常点驱逐事件日志。这在日常操作中非常有用，因为全局统计信息不能提供关于哪些主机正在被驱逐以及被驱逐的原因。日志使用 JSON 格式，每行一个对象：

```json
{
  "time": "...",
  "secs_since_last_action": "...",
  "cluster": "...",
  "upstream_url": "...",
  "action": "...",
  "type": "...",
  "num_ejections": "...",
  "enforced": "...",
  "host_success_rate": "...",
  "cluster_success_rate_average": "...",
  "cluster_success_rate_ejection_threshold": "..."
}
```

- time

  事件发生的时间。

- secs_since_last_action

  自上次动作（被驱逐或未被驱逐）发生时到现的时间（以秒为单位）。由于在第一次驱逐之前没有动作，所以该值将为 ` -1`。

- cluster

  拥有被驱逐主机的 [集群](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster.html#config-cluster-manager-cluster)。

- upstream_url

  被驱逐主机的 URL，例如 `tcp://1.2.3.4:80`。

- action

  发生的动作。如果主机被驱逐，则为 `eject`；如果主机恢复服务，则为 `uneject`。

- type

  如果 `action` 为 `eject`，`type` 指定发生的驱逐类型。当前类型可以是 5xx、`GatewayFailure` 或 `SuccessRate`。

- num_ejections

  如果 `action` 为 `eject`，则指定主机被驱逐的次数（对于 Envoy 而言是本地的，并且如果主机由于任何原因从上游集群移除然后被重新添加则重置该值）。

- enforced

  如果 `action` 为 `eject`，则指定驱逐是否被强制执行。`true` 表示主机被驱逐。`false` 表示事件已记录，但主机未被实际驱逐。

- host_success_rate

  如果 `action` 为 `eject`，指定发生驱逐事件时主机在 `0-100` 范围内的成功率。

- cluster_success_rate_average

  如果 `action` 为 `eject`，指定发生驱逐事件时集群中主机在 `0-100` 范围内的平均成功率。

- cluster_success_rate_ejection_threshold

  如果 `action` 为 `eject` 并且 `type` 为 `SuccessRate`，指定发生驱逐事件时的成功率驱逐阈值。

## 参考配置

- 集群管理器[全局配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/outlier.html#config-cluster-manager-outlier-detection)
- 单集群[配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection)
- 运行时[设置](https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cluster_runtime.html#config-cluster-manager-cluster-runtime-outlier-detection)
- 统计[参考](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-outlier-detection)