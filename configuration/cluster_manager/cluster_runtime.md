# 运行时

上游集群支持以下运行时设置：

## 主动健康检查

- health_check.min_interval

  健康检查的最小值 [interval](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-interval)。默认值是0。健康检查interval的值将位于*min_interval* 和 *max_interval*之间。

- health_check.max_interval

  健康检查的最大值 [interval](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-interval). 默认值是MAX_INT。健康检查interval的值将位于*min_interval* 和 *max_interval*之间。

- health_check.verify_cluster

  What % of health check requests will be verified against the [expected upstream service](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-service-name) as the [health check filter](../../intro/arch_overview/health_checking.md#arch-overview-health-checking-filter) will write the remote service cluster into the response.

## 异常点检测

查看异常点检测[架构概览](../../intro/arch_overview/outlier.md#arch-overview-outlier-detection) 取得更多异常点检测的信息。异常点检测支持的运行时参数和 [静态配置参数](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection)一样, namely:

- outlier_detection.consecutive_5xx

  [consecutive_5XX](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-5xx) 在异常点检测中的配置

- outlier_detection.consecutive_gateway_failure

  [consecutive_gateway_failure](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-gateway-failure) 在异常点检测中的配置

- outlier_detection.interval_ms

  [interval_ms](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-interval-ms) 在异常点检测中的配置

- outlier_detection.base_ejection_time_ms

  [base_ejection_time_ms](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-base-ejection-time-ms) 在异常点检测中的配置

- outlier_detection.max_ejection_percent

  [max_ejection_percent](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-max-ejection-percent) 在异常点检测中的配置

- outlier_detection.enforcing_consecutive_5xx

  [enforcing_consecutive_5xx](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-5xx) 在异常点检测中的配置

- outlier_detection.enforcing_consecutive_gateway_failure

  [enforcing_consecutive_gateway_failure](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-gateway-failure) 在异常点检测中的配置

- outlier_detection.enforcing_success_rate

  [enforcing_success_rate](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-success-rate) 在异常点检测中的配置

- outlier_detection.success_rate_minimum_hosts

  [success_rate_minimum_hosts](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-minimum-hosts) 在异常点检测中的配置

- outlier_detection.success_rate_request_volume

  [success_rate_request_volume](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) 在异常点检测中的配置

- outlier_detection.success_rate_stdev_factor

  [success_rate_stdev_factor](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-stdev-factor) 在异常点检测中的配置

## 核心

- upstream.healthy_panic_threshold

  设置 [panic threshold](../../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-panic-threshold) 百分比. 默认达到50%.

- upstream.use_http2

  Whether the cluster utilizes the *http2* [feature](../../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-features) if configured. Set to 0 to disable HTTP/2 even if the feature is configured. 默认值是关闭。

- upstream.weight_enabled

  用来打开或者关闭权重负载均衡的二级制开关。如果设置成非0数值，按权重负载均衡的功能是打开的. 默认值是打开.

## 区域感知负载均衡

- upstream.zone_routing.enabled

  多大百分比的请求将会被路由到相同上游区域。默认是100%请求

- upstream.zone_routing.min_cluster_size

  某个上游集群能被区域感知尝试的最小值。 默认值是6。如果上游集群数值比 *min_cluster_size*小，区域路由感知将不被执行.

## 短路

- circuit_breakers.<cluster_name>.<priority>.max_connections

  [断路器设置最大连接数](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-connections)

- circuit_breakers.<cluster_name>.<priority>.max_pending_requests

  [断路器设置最大等待数](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-pending-requests)

- circuit_breakers.<cluster_name>.<priority>.max_requests

  [断路器设置最大请求数](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-requests)

- circuit_breakers.<cluster_name>.<priority>.max_retries

  [断路器设置最大重试次数](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-retries)