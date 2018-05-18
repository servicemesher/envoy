# 异常点检测

Outlier detection and ejection is the process of dynamically determining whether some number of hosts in an upstream cluster are performing unlike the others and removing them from the healthy[load balancing](load_balancing.md#arch-overview-load-balancing) set. Performance might be along different axes such as consecutive failures, temporal success rate, temporal latency, etc. Outlier detection is a form of *passive* health checking. Envoy also supports [active health checking](health_checking.md#arch-overview-health-checking). *Passive* and *active* health checking can be enabled together or independently, and form the basis for an overall upstream health checking solution.

## 逐出算法

Depending on the type of outlier detection, ejection either runs inline (for example in the case of consecutive 5xx) or at a specified interval (for example in the case of periodic success rate). The ejection algorithm works as follows:

1. A host is determined to be an outlier.
2. If no hosts have been ejected, Envoy will eject the host immediately. Otherwise, it checks to make sure the number of ejected hosts is below the allowed threshold (specified via the[outlier_detection.max_ejection_percent](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection) setting). If the number of ejected hosts is above the threshold, the host is not ejected.
3. The host is ejected for some number of milliseconds. Ejection means that the host is marked unhealthy and will not be used during load balancing unless the load balancer is in a [panic](load_balancing.md#arch-overview-load-balancing-panic-threshold)scenario. The number of milliseconds is equal to the [outlier_detection.base_ejection_time_ms](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection)value multiplied by the number of times the host has been ejected. This causes hosts to get ejected for longer and longer periods if they continue to fail.
4. An ejected host will automatically be brought back into service after the ejection time has been satisfied. Generally, outlier detection is used alongside [active health checking](health_checking.md#arch-overview-health-checking) for a comprehensive health checking solution.

## 检测类型

Envoy supports the following outlier detection types:

### Consecutive 5xx

If an upstream host returns some number of consecutive 5xx, it will be ejected. Note that in this case a 5xx means an actual 5xx respond code, or an event that would cause the HTTP router to return one on the upstream’s behalf (reset, connection failure, etc.). The number of consecutive 5xx required for ejection is controlled by the [outlier_detection.consecutive_5xx](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-5xx) value.

### Consecutive Gateway Failure

If an upstream host returns some number of consecutive “gateway errors” (502, 503 or 504 status code), it will be ejected. Note that this includes events that would cause the HTTP router to return one of these status codes on the upstream’s behalf (reset, connection failure, etc.). The number of consecutive gateway failures required for ejection is controlled by the [outlier_detection.consecutive_gateway_failure](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-gateway-failure) value.

### Success Rate

Success Rate based outlier ejection aggregates success rate data from every host in a cluster. Then at given intervals ejects hosts based on statistical outlier detection. Success Rate outlier ejection will not be calculated for a host if its request volume over the aggregation interval is less than the[outlier_detection.success_rate_request_volume](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) value. Moreover, detection will not be performed for a cluster if the number of hosts with the minimum required request volume in an interval is less than the [outlier_detection.success_rate_minimum_hosts](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-minimum-hosts) value.

## 逐出事件记录

A log of outlier ejection events can optionally be produced by Envoy. This is extremely useful during daily operations since global stats do not provide enough information on which hosts are being ejected and for what reasons. The log uses a JSON format with one object per line:

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

  The time that the event took place.

- secs_since_last_action

  The time in seconds since the last action (either an ejection or unejection) took place. This value will be `-1` for the first ejection given there is no action before the first ejection.

- cluster

  The [cluster](../../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster) that owns the ejected host.

- upstream_url

  The URL of the ejected host. E.g., `tcp://1.2.3.4:80`.

- action

  The action that took place. Either `eject` if a host was ejected or `uneject` if it was brought back into service.

- type

  If `action` is `eject`, specifies the type of ejection that took place. Currently type can be one of `5xx`, `GatewayFailure` or `SuccessRate`.

- num_ejections

  If `action` is `eject`, specifies the number of times the host has been ejected (local to that Envoy and gets reset if the host gets removed from the upstream cluster for any reason and then re-added).

- enforced

  If `action` is `eject`, specifies if the ejection was enforced. `true` means the host was ejected.`false` means the event was logged but the host was not actually ejected.

- host_success_rate

  If `action` is `eject`, and `type` is `SuccessRate`, specifies the host’s success rate at the time of the ejection event on a `0-100` range.

- cluster_success_rate_average

  If `action` is `eject`, and `type` is `SuccessRate`, specifies the average success rate of the hosts in the cluster at the time of the ejection event on a `0-100` range.

- cluster_success_rate_ejection_threshold

  If `action` is `eject`, and `type` is `SuccessRate`, specifies success rate ejection threshold at the time of the ejection event.

## 参考配置

- 集群管理器[全局配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/outlier.html#config-cluster-manager-outlier-detection)
- 单集群[配置](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection.html#config-cluster-manager-cluster-outlier-detection)
- 运行时[设置](https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cluster_runtime.html#config-cluster-manager-cluster-runtime-outlier-detection)
- 统计[参考](../../configuration/cluster_manager/cluster_stats.md#config-cluster-manager-cluster-stats-outlier-detection)