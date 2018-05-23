# v1 API 概览

> **注意**
>
> v1 配置 API 现在被认为是过时了而且宣布了[废弃时间表](https://groups.google.com/forum/#!topic/envoy-announce/Lb1QZcSclGQ)。请升级并使用 [v2  配置 API](v2_overview.md#config-overview-v2)。

Envoy 配置格式是 JSON 的并用一个 JSON schema 验证。Schema 可以在 [source/common/json/config_schemas.cc](https://github.com/envoyproxy/envoy/blob/master/source/common/json/config_schemas.cc) 找到。服务器的主配置被包含在监听器和集群管理器部分。其他顶级元素指定各种配置。

为语法方便，也为手写的配置提供了 YAML 支持。如果一个文件路径的结尾是 .yaml，Envoy 将在内部将 YAML 转换为 JSON。在配置文档的剩余部分，我们仅指 JSON。Envoy 期望一个清晰的 YAML 标量，因此，如果一个集群名字（应该是一个字符串）被称为 *true*，它应该在配置 YAML 写为 *“true”*。同样适用于整数和浮点值(即 *1* vs. *1.0* vs. *“1.0”*)。

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

  *(required, array)* [监听器](../../intro/arch_overview/listeners.md#arch-overview-listeners) 的数组会被服务器实例化。一个单独的 Envoy 进程可以包含任意数量的监听器。

- [lds](../listeners/lds.md#config-listeners-lds)

  *(optional, object)* 对监听器发现服务(LDS)的配置。如果没有指定，仅静态监听器被加载。

- [admin](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v1_overview/api-v1/admin#config-admin-v1)

  *(required, object)* [本地管理 HTTP 服务器](../../operations/admin.md#operations-admin-interface)配置。

- [cluster_manager](../cluster_manager/cluster_manager.md#config-cluster-manager)

  *(required, object)* [集群管理器](../../intro/arch_overview/cluster_manager.md#arch-overview-cluster-manager) 配置，它拥有服务器内所有上游集群。

- flags_path

  *(optional, string)* 搜索[启动标志文件](../../operations/fs_flags.md#operations-file-system-flags)的文件系统路径。

- statsd_udp_ip_address

  *(optional, string)* 一个运行中的 遵循 statsd 的 监听器的 UDP 地址。如果被指定，[统计数据](../../intro/arch_overview/statistics.md#arch-overview-statistics)将被写入到这个地址。IPv4 地址应该具有主机:端口 (例如：127.0.0.1:855)的格式。IPv6 地址应该具有 URL 格式 [主机]:端口 (例如： [::1]:855)。

- statsd_tcp_cluster_name

  *(optional, string)* 一个运行遵循 TCP statsd 的监听器的集群管理器集群的名字。如果被指定，Envoy 将连接到这个集群以写入[统计数据](../../intro/arch_overview/statistics.md#arch-overview-statistics)。

- stats_flush_interval_ms

  *(optional, integer)* 两次写入到配置好的统计池之间以毫秒计的时间。出于性能原因，Envoy 锁定了计数器，仅周期性写入计数器和仪表的数据。如果未指定，默认为 5000ms (5 秒)。

- watchdog_miss_timeout_ms

  *(optional, integer)* Envoy 在 “server.watchdog_miss” 统计中计数一个无响应的线程后以毫秒计的时间。如果未指定，默认为 200ms。

- watchdog_megamiss_timeout_ms

  *(optional, integer)* Envoy 在 “server.watchdog_mega_miss” 统计中计数一个无响应的线程后以毫秒计的时间。如果未指定，默认为 1000ms。

- watchdog_kill_timeout_ms

  *(optional, integer)* 如果一个被观察的线程在这么多微秒的时间内没有响应，则假设一个编程错误并杀掉整个 Envoy 进程。设为 0 以关掉整个行为。如果未指定，默认为 0 (关掉的)。

- watchdog_multikill_timeout_ms

  *(optional, integer)* 如果至少两个被观察的线程在至少这么多微秒的时间内没有响应，则假设一个真正的死锁并杀掉整个 Envoy 进程。设为 0 以关掉整个行为。如果未指定，默认为 0 (关掉的)。

- [tracing](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v1_overview/api-v1/tracing#config-tracing-v1)

  *(optional, object)* 外部[追踪](../../intro/arch_overview/tracing.md#arch-overview-tracing)提供者的配置。 如果未指定，将不执行追踪。

- [rate_limit_service](../rate_limit.md#config-rate-limit-service)

  *(optional, object)* 外部[速率限制服务](../../intro/arch_overview/global_rate_limiting.md#arch-overview-rate-limit)提供者的配置。如果未指定，任何对速率限制服务的调用将立即返回成功。

- [runtime](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v1_overview/api-v1/runtime#config-runtime-v1)

  *(optional, object)* [运行时配置](../../intro/arch_overview/runtime.md#arch-overview-runtime) 提供者的配置。如果没有被指定，一个 “null” 提供者将被使用，这将导致使用所有的默认值。
配置
