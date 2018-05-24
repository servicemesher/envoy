# 运行时

上游集群支持以下运行时设置：

## 主动健康检查

- health_check.min_interval

  健康检查的最小值 [interval](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_hc#config-cluster-manager-cluster-hc-interval)。默认值是0。健康检查 interval 的值将位于 *min_interval* 和 *max_interval*之间。

- health_check.max_interval

  健康检查的最大值 [interval](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_hc#config-cluster-manager-cluster-hc-interval)。默认值是 MAX_INT。健康检查interval的值将位于 *min_interval* 和  *max_interval* 之间。

- health_check.verify_cluster

   当[健康检查过滤器](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_hc#config-cluster-manager-cluster-hc-service-name)将远程服务集群写入响应时，将对[预期的上游服务](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/health_checking#arch-overview-health-checking-filter)验证什么百分比的健康检查请求。

## 异常点检测

查看异常点检测[架构概览](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/outlier#arch-overview-outlier-detection) 取得更多异常点检测的信息。异常点检测支持的运行时参数和 [静态配置参数](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection)一样，分别为：

- outlier_detection.consecutive_5xx

  [consecutive_5XX](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-consecutive-5xx) 在异常点检测中的配置

- outlier_detection.consecutive_gateway_failure

  [consecutive_gateway_failure](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-consecutive-gateway-failure) 在异常点检测中的配置

- outlier_detection.interval_ms

  [interval_ms](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-interval-ms) 在异常点检测中的配置

- outlier_detection.base_ejection_time_ms

  [base_ejection_time_ms](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-base-ejection-time-ms) 在异常点检测中的配置

- outlier_detection.max_ejection_percent

  [max_ejection_percent](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-max-ejection-percent) 在异常点检测中的配置

- outlier_detection.enforcing_consecutive_5xx

  [enforcing_consecutive_5xx](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-5xx) 在异常点检测中的配置

- outlier_detection.enforcing_consecutive_gateway_failure

  [enforcing_consecutive_gateway_failure](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-gateway-failure) 在异常点检测中的配置

- outlier_detection.enforcing_success_rate

  [enforcing_success_rate](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-enforcing-success-rate) 在异常点检测中的配置

- outlier_detection.success_rate_minimum_hosts

  [success_rate_minimum_hosts](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-success-rate-minimum-hosts) 在异常点检测中的配置

- outlier_detection.success_rate_request_volume

  [success_rate_request_volume](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) 在异常点检测中的配置

- outlier_detection.success_rate_stdev_factor

  [success_rate_stdev_factor](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_outlier_detection#config-cluster-manager-cluster-outlier-detection-success-rate-stdev-factor) 在异常点检测中的配置

## 核心

- upstream.healthy_panic_threshold

  设置[恐慌阈](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/load_balancing#arch-overview-load-balancing-panic-threshold)百分比. 默认达到 50%.

- upstream.use_http2

   如果配置的话，集群是否使用 *http2* [特征](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster#config-cluster-manager-cluster-features) 。设置为0以禁用HTTP / 2，即使配置了该功能.默认值是关闭。

- upstream.weight_enabled

  用来打开或者关闭权重负载均衡的二级制开关。如果设置成非0数值，按权重负载均衡的功能是打开的。默认值是打开。

## Zone 感知负载均衡

- upstream.zone_routing.enabled

  多大百分比的请求将会被路由到相同上游 zone。默认是 100% 请求。

- upstream.zone_routing.min_cluster_size

  某个上游集群能被 zone 感知尝试的最小值。 默认值是 6。如果上游集群数值比 *min_cluster_size* 小，zone 路由感知将不被执行。

## 断路

- circuit_breakers.<cluster_name>.<priority>.max_connections

  [断路器设置最大连接数](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers#config-cluster-manager-cluster-circuit-breakers-max-connections)

- circuit_breakers.<cluster_name>.<priority>.max_pending_requests

  [断路器设置最大等待数](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers#config-cluster-manager-cluster-circuit-breakers-max-pending-requests)

- circuit_breakers.<cluster_name>.<priority>.max_requests

  [断路器设置最大请求数](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers#config-cluster-manager-cluster-circuit-breakers-max-requests)

- circuit_breakers.<cluster_name>.<priority>.max_retries

  [断路器设置最大重试次数](https://www.envoyproxy.io/docs/envoy/latest/api-v1/cluster_manager/cluster_circuit_breakers#config-cluster-manager-cluster-circuit-breakers-max-retries)