# v1 API 概览

> **Attention**
>
> The v1 configuration/API is now considered legacy and the [deprecation schedule](https://groups.google.com/forum/#!topic/envoy-announce/Lb1QZcSclGQ) has been announced. Please upgrade and use the [v2 configuration/API](v2_overview.md#config-overview-v2).

The Envoy configuration format is written in JSON and is validated against a JSON schema. The schema can be found in [source/common/json/config_schemas.cc](https://github.com/envoyproxy/envoy/blob/master/source/common/json/config_schemas.cc). The main configuration for the server is contained within the listeners and cluster manager sections. The other top level elements specify miscellaneous configuration.

YAML support is also provided as a syntactic convenience for hand-written configurations. Envoy will internally convert YAML to JSON if a file path ends with .yaml. In the rest of the configuration documentation, we refer exclusively to JSON. Envoy expects unambiguous YAML scalars, so if a cluster name (which should be a string) is called *true*, it should be written in the configuration YAML as *“true”*. The same applies to integer and floating point values (e.g. *1* vs. *1.0* vs. *“1.0”*).

```json
{
  "listeners": [],
  "lds": "{...}",
  "admin": "{...}",
  "cluster_manager": "{...}",
  "flags_path": "...",
  "statsd_udp_ip_address": "...",
  "statsd_tcp_cluster_name": "...",
  "stats_flush_interval_ms": "...",
  "watchdog_miss_timeout_ms": "...",
  "watchdog_megamiss_timeout_ms": "...",
  "watchdog_kill_timeout_ms": "...",
  "watchdog_multikill_timeout_ms": "...",
  "tracing": "{...}",
  "rate_limit_service": "{...}",
  "runtime": "{...}",
}
```

- [listeners](../listeners/listeners.md#config-listeners)

  *(required, array)* An array of [listeners](../../intro/arch_overview/listeners.md#arch-overview-listeners) that will be instantiated by the server. A single Envoy process can contain any number of listeners.

- [lds](../listeners/lds.md#config-listeners-lds)

  *(optional, object)* Configuration for the Listener Discovery Service (LDS). If not specified only static listeners are loaded.

- [admin](../../api-v1/admin.md#config-admin-v1)

  *(required, object)* Configuration for the [local administration HTTP server](../../operations/admin.md#operations-admin-interface).

- [cluster_manager](../cluster_manager/cluster_manager.md#config-cluster-manager)

  *(required, object)* Configuration for the [cluster manager](../../intro/arch_overview/cluster_manager.md#arch-overview-cluster-manager) which owns all upstream clusters within the server.

- flags_path

  *(optional, string)* The file system path to search for [startup flag files](../../operations/fs_flags.md#operations-file-system-flags).

- statsd_udp_ip_address

  *(optional, string)* The UDP address of a running statsd compliant listener. If specified, [statistics](../../intro/arch_overview/statistics.md#arch-overview-statistics)will be flushed to this address. IPv4 addresses should have format host:port (ex: 127.0.0.1:855). IPv6 addresses should have URL format [host]:port (ex: [::1]:855).

- statsd_tcp_cluster_name

  *(optional, string)* The name of a cluster manager cluster that is running a TCP statsd compliant listener. If specified, Envoy will connect to this cluster to flush [statistics](../../intro/arch_overview/statistics.md#arch-overview-statistics).

- stats_flush_interval_ms

  *(optional, integer)* The time in milliseconds between flushes to configured stats sinks. For performance reasons Envoy latches counters and only flushes counters and gauges at a periodic interval. If not specified the default is 5000ms (5 seconds).

- watchdog_miss_timeout_ms

  *(optional, integer)* The time in milliseconds after which Envoy counts a nonresponsive thread in the “server.watchdog_miss” statistic. If not specified the default is 200ms.

- watchdog_megamiss_timeout_ms

  *(optional, integer)* The time in milliseconds after which Envoy counts a nonresponsive thread in the “server.watchdog_mega_miss” statistic. If not specified the default is 1000ms.

- watchdog_kill_timeout_ms

  *(optional, integer)* If a watched thread has been nonresponsive for this many milliseconds assume a programming error and kill the entire Envoy process. Set to 0 to disable kill behavior. If not specified the default is 0 (disabled).

- watchdog_multikill_timeout_ms

  *(optional, integer)* If at least two watched threads have been nonresponsive for at least this many milliseconds assume a true deadlock and kill the entire Envoy process. Set to 0 to disable this behavior. If not specified the default is 0 (disabled).

- [tracing](../../api-v1/tracing.md#config-tracing-v1)

  *(optional, object)* Configuration for an external [tracing](../../intro/arch_overview/tracing.md#arch-overview-tracing) provider. If not specified, no tracing will be performed.

- [rate_limit_service](../rate_limit.md#config-rate-limit-service)

  *(optional, object)* Configuration for an external [rate limit service](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit) provider. If not specified, any calls to the rate limit service will immediately return success.

- [runtime](../../api-v1/runtime.md#config-runtime-v1)

  *(optional, object)* Configuration for the [runtime configuration](../../intro/arch_overview/runtime.md#arch-overview-runtime) provider. If not specified, a “null” provider will be used which will result in all defaults being used.