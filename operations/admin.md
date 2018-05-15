# 管理接口

Envoy exposes a local administration interface that can be used to query and modify different aspects of the server:

- [v1 API reference](../api-v1/admin.md#config-admin-v1)
- [v2 API reference](../api-v2/config/bootstrap/v2/bootstrap.proto.md#envoy-api-msg-config-bootstrap-v2-admin)

Attention

The administration interface in its current form both allows destructive operations to be performed (e.g., shutting down the server) as well as potentially exposes private information (e.g., stats, cluster names, cert info, etc.). It is **critical** that access to the administration interface is only allowed via a secure network. It is also **critical** that hosts that access the administration interface are **only** attached to the secure network (i.e., to avoid CSRF attacks). This involves setting up an appropriate firewall or optimally only allowing access to the administration listener via localhost. This can be accomplished with a v2 configuration like the following:

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }
```

In the future additional security options will be added to the administration interface. This work is tracked in [this](https://github.com/envoyproxy/envoy/issues/2763) issue.

All mutations should be sent as HTTP POST operations. For a limited time, they will continue to work with HTTP GET, with a warning logged.

- `GET /`

  Render an HTML home page with a table of links to all available options.

- `GET /help`

  Print a textual table of all available options.

- `GET /certs`

  List out all loaded TLS certificates, including file name, serial number, and days until expiration.

- `GET /clusters`

  List out all configured [cluster manager](../intro/arch_overview/cluster_manager.md#arch-overview-cluster-manager) clusters. This information includes all discovered upstream hosts in each cluster along with per host statistics. This is useful for debugging service discovery issues.Cluster manager information`version_info` string – the version info string of the last loaded [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) update. If envoy does not have [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) setup, the output will read `version_info::static`.Cluster wide information[circuit breakers](../configuration/cluster_manager/cluster_circuit_breakers.md#config-cluster-manager-cluster-circuit-breakers) settings for all priority settings.Information about [outlier detection](../intro/arch_overview/outlier.md#arch-overview-outlier-detection) if a detector is installed. Currently [success rate average](../intro/arch_overview/outlier.md#arch-overview-outlier-detection-ejection-event-logging-cluster-success-rate-average), and [ejection threshold](../intro/arch_overview/outlier.md#arch-overview-outlier-detection-ejection-event-logging-cluster-success-rate-ejection-threshold) are presented. Both of these values could be `-1` if there was not enough data to calculate them in the last [interval](../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-interval-ms).`added_via_api` flag – `false` if the cluster was added via static configuration, `true` if it was added via the [CDS](../configuration/cluster_manager/cds.md#config-cluster-manager-cds) api.Per host statisticsNameTypeDescriptioncx_totalCounterTotal connectionscx_activeGaugeTotal active connectionscx_connect_failCounterTotal connection failuresrq_totalCounterTotal requestsrq_timeoutCounterTotal timed out requestsrq_successCounterTotal requests with non-5xx responsesrq_errorCounterTotal requests with 5xx responsesrq_activeGaugeTotal active requestshealthyStringThe health status of the host. See belowweightIntegerLoad balancing weight (1-100)zoneStringService zonecanaryBooleanWhether the host is a canarysuccess_rateDoubleRequest success rate (0-100). -1 if there was not enough [request volume](../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-success-rate-request-volume) in the [interval](../api-v1/cluster_manager/cluster_outlier_detection.md#config-cluster-manager-cluster-outlier-detection-interval-ms) to calculate itHost health statusA host is either healthy or unhealthy because of one or more different failing health states. If the host is healthy the `healthy` output will be equal to *healthy*.If the host is not healthy, the `healthy` output will be composed of one or more of the following strings:*/failed_active_hc*: The host has failed an [active health check](../configuration/cluster_manager/cluster_hc.md#config-cluster-manager-cluster-hc).*/failed_eds_health*: The host was marked unhealthy by EDS.*/failed_outlier_check*: The host has failed an outlier detection check.

- `GET /config_dump`

  Dump currently loaded configuration from various Envoy components as JSON-serialized proto messages. See the [response definition](../api-v2/admin/v2alpha/config_dump.proto.md#envoy-api-msg-admin-v2alpha-configdump) for more information.

- `POST /cpuprofiler`

  Enable or disable the CPU profiler. Requires compiling with gperftools.

- `POST /healthcheck/fail`

  Fail inbound health checks. This requires the use of the HTTP [health check filter](../configuration/http_filters/health_check_filter.md#config-http-filters-health-check). This is useful for draining a server prior to shutting it down or doing a full restart. Invoking this command will universally fail health check requests regardless of how the filter is configured (pass through, etc.).

- `POST /healthcheck/ok`

  Negate the effect of [`POST /healthcheck/fail`](#post--healthcheck-fail). This requires the use of the HTTP [health check filter](../configuration/http_filters/health_check_filter.md#config-http-filters-health-check).

- `GET /hot_restart_version`

  See [`--hot-restart-version`](cli.md#cmdoption-hot-restart-version).

- `POST /logging`

  Enable/disable different logging levels on different subcomponents. Generally only used during development.

- `POST /quitquitquit`

  Cleanly exit the server.

- `POST /reset_counters`

  Reset all counters to zero. This is useful along with [`GET /stats`](#get--stats) during debugging. Note that this does not drop any data sent to statsd. It just effects local output of the [`GET /stats`](#get--stats) command.

- `GET /server_info`

  Outputs information about the running server. Sample output looks like:

```
envoy 267724/RELEASE live 1571 1571 0
```

The fields are:

- Process name
- Compiled SHA and build type
- Health check state (live or draining)
- Current hot restart epoch uptime in seconds
- Total uptime in seconds (across all hot restarts)
- Current hot restart epoch

- `GET /stats`

  Outputs all statistics on demand. This command is very useful for local debugging. Histograms will output the computed quantiles i.e P0,P25,P50,P75,P90,P99,P99.9 and P100. The output for each quantile will be in the form of (interval,cumulative) where interval value represents the summary since last flush interval and cumulative value represents the summary since the start of envoy instance. See [here](stats_overview.md#operations-stats) for more information.`GET /stats?format=json`Outputs /stats in JSON format. This can be used for programmatic access of stats. Counters and Gauges will be in the form of a set of (name,value) pairs. Histograms will be under the element “histograms”, that contains “supported_quantiles” which lists the quantiles supported and an array of computed_quantiles that has the computed quantile for each histogram. Only histograms with recorded values will be exported.If a histogram is not updated during an interval, the ouput will have null for all the quantiles.Example histogram output:

```json
{
  "histograms": {
    "supported_quantiles": [
      0, 25, 50, 75, 90, 95, 99, 99.9, 100
    ],
    "computed_quantiles": [
      {
        "name": "cluster.external_auth_cluster.upstream_cx_length_ms",
        "values": [
          {"interval": 0, "cumulative": 0},
          {"interval": 0, "cumulative": 0},
          {"interval": 1.0435787, "cumulative": 1.0435787},
          {"interval": 1.0941565, "cumulative": 1.0941565},
          {"interval": 2.0860023, "cumulative": 2.0860023},
          {"interval": 3.0665233, "cumulative": 3.0665233},
          {"interval": 6.046609, "cumulative": 6.046609},
          {"interval": 229.57333,"cumulative": 229.57333},
          {"interval": 260,"cumulative": 260}
        ]
      },
      {
        "name": "http.admin.downstream_rq_time",
        "values": [
          {"interval": null, "cumulative": 0},
          {"interval": null, "cumulative": 0},
          {"interval": null, "cumulative": 1.0435787},
          {"interval": null, "cumulative": 1.0941565},
          {"interval": null, "cumulative": 2.0860023},
          {"interval": null, "cumulative": 3.0665233},
          {"interval": null, "cumulative": 6.046609},
          {"interval": null, "cumulative": 229.57333},
          {"interval": null, "cumulative": 260}
        ]
      }
    ]
  }
}
```

- `GET /stats?format=prometheus`

  or alternatively,`GET /stats/prometheus`Outputs /stats in [Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/) v0.0.4 format. This can be used to integrate with a Prometheus server. Currently, only counters and gauges are output. Histograms will be output in a future update.

- `GET /runtime`

  Outputs all runtime values on demand in JSON format. See [here](../intro/arch_overview/runtime.md#arch-overview-runtime) for more information on how these values are configured and utilized. The output include the list of the active runtime override layers and the stack of layer values for each key. Empty strings indicate no value, and the final active value from the stack also is included in a separate key. Example output:

```json
{
  "layers": [
    "disk",
    "override",
    "admin",
  ],
  "entries": {
    "my_key": {
      "layer_values": [
        "my_disk_value",
        "",
        ""
      ],
      "final_value": "my_disk_value"
    },
    "my_second_key": {
      "layer_values": [
        "my_second_disk_value",
        "my_disk_override_value",
        "my_admin_override_value"
      ],
      "final_value": "my_admin_override_value"
    }
  }
}
```

- `POST /runtime_modify?key1=value1&key2=value2&keyN=valueN`

  Adds or modifies runtime values as passed in query parameters. To delete a previously added key, use an empty string as the value. Note that deletion only applies to overrides added via this endpoint; values loaded from disk can be modified via override but not deleted.

> **Attention**
>
> Use the /runtime_modify endpoint with care. Changes are effectively immediately. It is **critical**that the admin interface is [properly secured](#operations-admin-interface-security).