# 运行时

Upstream clusters support the following runtime settings:

## Active health checking

- health_check.min_interval

  Min value for the health checking [interval](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-interval). Default value is 0. The health checking interval will be between *min_interval* and *max_interval*.

- health_check.max_interval

  Max value for the health checking [interval](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-interval). Default value is MAX_INT. The health checking interval will be between *min_interval* and *max_interval*.

- health_check.verify_cluster

  What % of health check requests will be verified against the [expected upstream service](../../api-v1/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc-service-name) as the [health check filter](../../intro/arch_overview/health_checking.md#arch-overview-health-checking-filter) will write the remote service cluster into the response.

## Outlier detection

See the outlier detection [architecture overview](../../intro/arch_overview/outlier.md#arch-overview-outlier-detection) for more information on outlier detection. The runtime parameters supported by outlier detection are the same as the [static configuration parameters](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection), namely:

- outlier_detection.consecutive_5xx

  [consecutive_5XX](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-5xx) setting in outlier detection

- outlier_detection.consecutive_gateway_failure

  [consecutive_gateway_failure](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-consecutive-gateway-failure) setting in outlier detection

- outlier_detection.interval_ms

  [interval_ms](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-interval-ms) setting in outlier detection

- outlier_detection.base_ejection_time_ms

  [base_ejection_time_ms](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-base-ejection-time-ms) setting in outlier detection

- outlier_detection.max_ejection_percent

  [max_ejection_percent](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-max-ejection-percent) setting in outlier detection

- outlier_detection.enforcing_consecutive_5xx

  [enforcing_consecutive_5xx](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-5xx) setting in outlier detection

- outlier_detection.enforcing_consecutive_gateway_failure

  [enforcing_consecutive_gateway_failure](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-consecutive-gateway-failure) setting in outlier detection

- outlier_detection.enforcing_success_rate

  [enforcing_success_rate](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-enforcing-success-rate) setting in outlier detection

- outlier_detection.success_rate_minimum_hosts

  [success_rate_minimum_hosts](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-minimum-hosts) setting in outlier detection

- outlier_detection.success_rate_request_volume

  [success_rate_request_volume](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) setting in outlier detection

- outlier_detection.success_rate_stdev_factor

  [success_rate_stdev_factor](../../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-stdev-factor) setting in outlier detection

## Core

- upstream.healthy_panic_threshold

  Sets the [panic threshold](../../intro/arch_overview/load_balancing.md#arch-overview-load-balancing-panic-threshold) percentage. Defaults to 50%.

- upstream.use_http2

  Whether the cluster utilizes the *http2* [feature](../../api-v1/cluster_manager/cluster.md#config-cluster-manager-cluster-features) if configured. Set to 0 to disable HTTP/2 even if the feature is configured. Defaults to enabled.

- upstream.weight_enabled

  Binary switch to turn on or off weighted load balancing. If set to non 0, weighted load balancing is enabled. Defaults to enabled.

## Zone aware load balancing

- upstream.zone_routing.enabled

  % of requests that will be routed to the same upstream zone. Defaults to 100% of requests.

- upstream.zone_routing.min_cluster_size

  Minimal size of the upstream cluster for which zone aware routing can be attempted. Default value is 6. If the upstream cluster size is smaller than *min_cluster_size* zone aware routing will not be performed.

## Circuit breaking

- circuit_breakers.<cluster_name>.<priority>.max_connections

  [Max connections circuit breaker setting](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-connections)

- circuit_breakers.<cluster_name>.<priority>.max_pending_requests

  [Max pending requests circuit breaker setting](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-pending-requests)

- circuit_breakers.<cluster_name>.<priority>.max_requests

  [Max requests circuit breaker setting](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-requests)

- circuit_breakers.<cluster_name>.<priority>.max_retries

  [Max retries circuit breaker setting](../../api-v1/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers-max-retries)